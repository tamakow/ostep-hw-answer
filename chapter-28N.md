# note of chap 28 (lock)

并发编程最基本的问题：希望原子性的执行一系列指令，但由于单处理器的中断/多个线程在多处理器上并发执行，原子性无法保证。

如何解决： 在临界区周围加锁，保证临界区代码能够像单条原子指令一样执行

## 28.1 锁的基本思想

### 锁的定义
锁就是一个变量，故需要声明一个某种类型的锁变量（lock variable）才能使用，且这个锁变量时刻保存着锁的状态

锁只有两种状态：
* 可用的(available / unlocked / free)，代表没有线程持有该锁
* 被占用的(acquired / locked / held)，代表有一个线程持有锁，正处于临界区

锁变量中可以保存其他的信息，比如持有锁的线程，或请求获取锁的线程队列，但这些信息应该对锁的使用者不可见

### 上锁 解锁
* lock()尝试获取锁，如果没有其他线程持有锁（即锁是可用的），该线程会获得锁，进入临界区，这个线程被称为锁的持有者（owner）。如果其他线程此时调用lock()企图获得同一把锁，那么直到持有者释放这把锁之前此次调用都不会返回。这样就可以阻止其他线程进入临界区。
* 当持有者调用unlock()时，锁就再次可用。如果没有其他线程在等锁，那么锁就被标记为可用(free)。否则，等待的线程中会（最终）有一个注意到锁的状态发生变化，获得它并进入临界区。


锁为程序员提供了最小程度的调度控制。我们把线程视为程序员创建的实体,但是被
操作系统调度,具体方式由操作系统选择。锁让程序员获得一些控制权。通过给临界区加
锁,可以保证临界区内只有一个线程活跃。锁将原本由操作系统调度的混乱状态变得更为
可控。

### Pthread 锁

POSIX标准库提供的锁叫做mutex（互斥锁），提供线程之间的互斥。如果一个线程处于临界区，它（线程）就能阻止其他线程进入临界区，直到它（线程）完成了临界区的任务并退出。

POSIX允许上锁时传入一个锁变量，这样就可以使用不同的锁来保护不同的变量。这样做的好处是可以增加并发性：不需要一把大锁锁住所有的临界区（coarse-grained strategy)，多把锁可以让更多的线程同时处于上锁的区间(fine-grained approach)。

## 28.4 评价锁
1. 锁能否完成它的基本任务，即提供互斥
2. 公平性：是否有竞争锁的线程会饿死，一直无法获得锁
3. 性能：使用锁之后增加的时间开销
   * 没有竞争的情况下：只有一个线程抢锁、释放锁的开支如何？
   * 一个CPU多个线程竞争，性能如何
   * 多个CPU，多个线程竞争时的性能

### 28.5 控制中断
定义： 上锁——关中断，解锁——开中断

在**单处理器系统**中，通过关闭中断，我们可以处于临界区中的代码不被干扰，因此它的执行是原子的。

优点： 简单，正确性显然

**缺点**：
   1. 开、关中断要求允许程进行特权操作，OS必须要信任这个机制不被滥用。（但是很可能出麻烦，比如独占CPU，死循环后OS无法得到控制，只能重启）
   2. 不支持多处理器
   3. 关闭中断导致中断丢失，可能会导致严重的系统问题（磁盘完成了读取请求，但CPU错失了这一信息）
   4. 效率低

因此，关闭中断仅在有限的环境下，例如原始的互斥条件下使用。在OS中这样的用法是可行的，因为OS肯定会信任自己并允许自己进行特权操作。

#### 软件实现锁
Dekker算法
Peterson算法（针对两个线程）
```c
int flag[2];
int turn;

void init() {
   flag[0] = flag[1] = 0;
   turn = 0;
}

void lock() {
   flag[self] = 1; // self : thread ID of caller
   turn = 1 - self; // make it other thread's turn 
   while ((flag[1-self] == 1) && (turn == 1 - self)) ;// spin-wait
}

void unlock() {
   flag[self] = 0;
}
```

但是只需要很少的硬件支持，实现锁就会容易很多，而且上面的方法对于现代硬件（松散内存一致性模型）没有什么用处


### 硬件支持以及对应的自旋锁 

首先用一个变量实现不依赖指令的锁，然而很显然的出现了问题，通过某种中断

#### test-and-set
```c
int TestAndSet (int *old_ptr, int new) {
   int old = *old_ptr; //fetch old value at old_ptr
   *old_ptr = new; //store 'new' into old_ptr
   return old;
}
```
返回old_ptr的旧值，同时更新为new的新值。这个操作序列是原子地执行的。拥有这个操作已经足够构建自旋锁
```c
typedef struct lock_t {int flag;} lock_t;

void init(lock_t *lock) {
   // 0 indicates that lock is available, 1 that it is held
   lock->flag = 0;
}

void lock(lock_t *lock) {
   while (TestAndSet(&lock->flag, 1) == 1) ; //spin-wait
}

void unlock(lock_t *lock) {
   lock->flag = 0;
}
```

为了让自旋锁在单CPU系统上正确工作，需要一个抢占式的调度处理器(preemptive scheduler)来根据占用时间打断、切换线程。

Test-And-Set 自旋锁的评估：
* 正确性：bingo，只允许一个线程进入临界区
* 公平性：无公平可言
* 性能：单CPU中自旋开销很大;多CPU中（如果线程数大致等于CPU数）可以认为临界区很短，因此锁可以很快恢复可用且被其他线程获得，自旋不会有很大的开销

x86中的Test-And-Set指令是**xchg**

#### Compare-And-Swap
```c
int CompareAndSwap(int *ptr, int expected, int new) {
   int actual = *ptr;
   if (actual == expected)
      *ptr = new;
   return actual;
}
```

以上函数会返回内存中的实际值，由此调用者即可知道其执行是否成功。使用此方法构造的自旋锁如下(与Test-And-Set只需要修改lock函数即可)：

```c
void lock(lock_t *lock) {
   while (CompareAndSwap(&lock->flag, 0, 1) == 1); //spin wait
}
```

此款锁的行为与上一款锁完全一致，但对于无锁同步(lock-free synchronization)有神秘好处（书里此处没说）。

x86中对应的指令为**compare-and-exchange**

#### Load-Linked and Store-conditional
MIPS架构中提供了实现临界区的一堆指令(ARM,Alpha,PowerPC......也有)

```c
int LoadLinked(int *ptr) {
  return *ptr;
}

int StoreConditional(int *ptr, int value) {
  if (/* no one has updated *ptr since the LoadLinked to this address */) {
    *ptr = value;
    return 1; // success
  } else {
    return 0; // failed to update
  }
}
```
由之实现的自旋锁为：
```c
void lock(lock_t *lock) {
  while (1) {
    while (LoadLinked(&lock->flag) == 1)
      ; // spin until it's zero
    if (StoreConditional(&lock->flag, 1) == 1)
      return; // if set to 1: success将线程的堆栈放置在同一个地址空间中，在实现上既比较容易，同时对 C 程序的执行也是非常必要的，例如我们可以写出以下代码：


  }
}
```

#### Fetch-And-Add
指令功能：原子地返回特定地址的旧值，并让该值自增1
```c
int FetchAndAdd(int *ptr) {
  int old = *ptr;
  *ptr = old + 1;
  return *ptr;
}
```
可以用这个指令实现一个门票锁（ticket lock）
```c
typedef struct __lock_t {
  int ticket;
  int turn;
} lock_t;

void lock(lock_t *lock) {
  int myTurn = FetchAndAdd(&lock->ticket);
  while (lock->turn != myTurn)
    ; // spin-wait
}

void unlock(lock_t *lock) {
  FetchAndAdd(&lock->turn); //lock->turn = lock->turn + 1;
}

```

这款锁和之前的所有锁之间的差别是：它保证了每个线程都有机会获得锁。一旦一个线程被分配了一个ticket value，它就被安排在未来的某个时刻能够获得锁。即，**公平**性得到了保证


### 怎样避免自旋，浪费CPU时间
需要OS的支持

#### yield
```c
void lock() {
   while(TestAndSet(&flag, 1 1) yield(); //give up the CPU
}
```
每当自旋过程中没有获得锁时，线程执行yield()来主动放弃CPU，让下一个线程执行。

这样可以解决性能问题，但无法解决公平性（有线程无法获得锁，starvation problem）的问题。


#### 使用队列：休眠替代自旋
Solaris采用的解决方案是使用两个调用：
* park():让调用者进入睡眠状态
* unpark(threadID):唤醒指定的线程

用这两个调用来实现锁，让调用者在获取不到锁时睡眠，在锁可用的时候被唤醒

```c
typedef struct __lock_t {
  int flag;
  int guard;
  queue_t *q;
} lock_t;

void lock(lock_t *m) {
  while (TestAndSet(&m->guard, 1) == 1)
    ; // acquire guard lock by spinning
  if (m->flag == 0) {
    m->flag = 1; // lock is acqured
    m->guard = 0;
  } else {
    queue_add(m->q, gettid());
    m->guard = 0;
    park(); // BOOOOOOOOOOOOM!
  }
}

void unlock(lock_t *m) {
  while (TestAndSet(&m->guard, 1) == 1)
    ; // acquire guard lock by spinning
  if (queue_empty(m->q))
    m->flag = 0;
  else
    unpark(queue_remove(m->q)); // hold lock
  m->guard = 0;
}
```

当一个进程被唤醒时，将会从park()返回；否则，他就不会持有guard锁，更不能将锁的flag设为1。因此，我们只需要将锁直接传递给下一个需要它的进程，而不能把flag清除。

但以上实现可能会出现竞争状况：一个线程即将进入睡眠，直到锁的状态被解除，但此时另一个线程刚刚好将其“唤醒”（从队列中删除），那么前者就永远无法被唤醒。这个问题也被叫做唤醒/等待竞争(wakeup/waiting race)。

Solaris的解决方案是增加第三个系统调用：setpark()，告知OS该线程即将进入睡眠状态。如果此时发生中断，另一个线程唤醒当前线程，那么下一步调用的park()就会直接返回，不会进入睡眠状态。


#### 不同的OS有不同的实现
Linux实现了futex:
* futex_wait(address, expected)：如果address处的值等于expected，就会让调线程睡眠，否则，调用立刻返回
* futex_wake(address)唤醒等待队列中的一个线程

<font color='red'>在os的课堂上介绍了futex（并发控制：互斥（2））</font>