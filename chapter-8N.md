# note of chapter 8 (cpu-sched-mlfq)

1. two-fold problem:
   1. optimize turnaround time(however we do not know how long a job will long for)
   2. minimize response time to make a system feel responsive to interactive users(but RR is terrible for turnaround time)  


## 8.1 MLFQ: Basic Rules
1. two basic rules:
   * **Rule 1:** if Priority(A) > Pirority(B), A runs(B doesn't)
   * **Rule 2:** if Priority(A) = Priority(B), A & B run in RR

2. how the scheduler sets priorities: **learn** about processes as they run, and thus use the **history** of the job to predict its **future** behavior
   1. a job repeatedly relinquishes the CPU while waiting for input from the keyboard (keep priority **high**)
   2. a job uses the CPU intensively for long period of time (**reduce** its priority)


## 8.2 Attempt #1: How To Change Priority
1. addtional rules(first attempt at a priority-adjustment algorithm):
   * **Rule 3:** When a job enters the system, it is placed at the highest priority(the topmost queue)
   *  **Rule 4a:** If a job uses up an entire time slice while running, its priority is reduced(moves down one queue)
   *  **Rule 4b:** If a job gives up the CPU before the time slice is up, it stays at the same priority level  

2. problems:
   1. **starvation**: if there are "too many" interactive jobs in the system, they will combine all CPU time, and thus long-running jobs will never receive any CPU time
   2. a smart user could rewrite their program to **game the scheduler**
   3. may *change its behavior* over time


## 8.3 Attempt #2: The Priority Boost
1. rules to avoid starvation:
   * **Rule 5:** After some time period $S$, move all the jobs in the system to the topmost queue

## 8.4 Attempt #3: Better Accounting
1. rewrite Rule 4a and ab to prevent gaming of scheduler:
   * **Rule 4:** Once a job uses up its time allotment at a given level(regardless of how many times it has given up the CPU), its priority is reduced
