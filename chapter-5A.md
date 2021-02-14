# answer of chapter 5

### simulation
1. just simulate(notice README-fork.md)
2. while fork_percentage increases from 0.1 to 0.9, there are more and more children in the final process trees
3. of course I can
4. I thought d and e would be b's children but actually they became a's children. Using -R they became b's children
5. actions are easy to know but the order is often not sure.


### code


1. they both change the value in their own process,but donnot affect the value of the other process.(For example, if we let the parent process run first, the result is 200(parent process) and 100(child process))
source code:  
```c
#include <stdio.h>
#include <stdlib.h> // exit
#include <unistd.h> // fork
#include <assert.h> // assert
#include <wait.h>

int main(){
    int m = 100;
    int rc = fork();
    if(rc < 0){
        fprintf(stderr,"fork failed!\n");
        assert(0);
    }else if(rc == 0){
        printf("m in the child process is %d\n",m);
    }else{
        // wait(NULL);
        m = 200;
        printf("m in the parent process is %d\n",m);
    }
    return 0;
}
```

2. They can both access the file descriptor. Uncertainly but mostly, the parent writes before the child if they are writing to the file concurrently.
3. use kill() to send SIGCONT signal to parent process
4. STFW
5. if success, return value is the PID of the child process;else return -1
6. won't show in the terminal