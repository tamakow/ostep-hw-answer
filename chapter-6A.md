# homework answer for chapter 6

### the precision and accuracy of timer
run this code:  
```c
#include <stdio.h>
#include <stdlib.h> // exit
#include <unistd.h> // fork
#include <sys/time.h>

int main(){
    int n = 0;
    while(n++<10){
        struct timeval t;
        gettimeofday(&t,NULL);
        __time_t s = t.tv_sec;
        __suseconds_t us = t.tv_usec;
        long long rs = (long long)s * 1000000 + us;
        printf("%lld\n", rs);
        sleep(1);
    }
    return 0;
}
```
the result is actually as below:
```sh
1612860743179331
1612860744179714
1612860745180126
1612860746180516
1612860747180888
1612860748181112
1612860749181284
1612860750181443
1612860751181706
1612860752181883
```
we can know from the result that the timer is not absolutely precise and accurate

### measuring the cost of a context switch
run this code(https://github.com/xxyzz/ostep-hw/blob/master/6/1.c):
```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/time.h>
#include <sched.h>

int
main(int argc, char *argv[]) {
    // measure system call
    int fd = open("./1.txt", O_CREAT | O_WRONLY | O_TRUNC, S_IRWXU), nloops = 1000000;

    struct timeval start, end;
    gettimeofday(&start, NULL);
    for (size_t i = 0; i < nloops; i++) {
        read(fd, NULL, 0);
    }
    gettimeofday(&end, NULL);
    printf("system call: %f microseconds\n\n", (float) (end.tv_sec * 1000000 + end.tv_usec - start.tv_sec * 1000000 - start.tv_usec) / nloops);
    close(fd);

    // measure context switch
    cpu_set_t set;
    CPU_ZERO(&set);
    CPU_SET(0, &set);

    int first_pipefd[2], second_pipefd[2];
    if (pipe(first_pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }
    if (pipe(second_pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid_t cpid = fork();

    if (cpid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (cpid == 0) {    // child
        if (sched_setaffinity(getpid(), sizeof(cpu_set_t), &set) == -1) {
            exit(EXIT_FAILURE);
        }

        for (size_t i = 0; i < nloops; i++) {
            read(first_pipefd[0], NULL, 0);
            write(second_pipefd[1], NULL, 0);
        }
    } else {           // parent
        if (sched_setaffinity(getpid(), sizeof(cpu_set_t), &set) == -1) {
            exit(EXIT_FAILURE);
        }

        gettimeofday(&start, NULL);
        for (size_t i = 0; i < nloops; i++) {
            write(first_pipefd[1], NULL, 0);
            read(second_pipefd[0], NULL, 0);
        }
        gettimeofday(&end, NULL);
        printf("context switch: %f microseconds\n", (float) (end.tv_sec * 1000000 + end.tv_usec - start.tv_sec * 1000000 - start.tv_usec) / nloops);
    }
    return 0;
}
```

the answer is as below:
```sh
system call: 0.316276 microseconds

context switch: 1.442069 microseconds
```