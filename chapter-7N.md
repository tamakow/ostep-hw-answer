# note of chapter 7 (cpu-sched)

## 7.1 woekload assumptions
1. some assumptions about the processes(sometimes called **jobs**):
   1. each job runs for the same amount of time
   2. all jobs arrive at the same time
   3. once started, each job runs to completion
   4. all jobs only use the CPU(i.e., they perform no I/O)
   5. the run-time of each job is known 


## 7.2 scheduling metrics
1. metric: something that we use to measure something
2. turnaround time(a **performance** metric):$T_{turnaround} = T_{completion} - T_{arrival}$.
   * if we use the assumption in 7.1(all jobs arrive at the same time), $T_{arrival} = 0 \Rightarrow T_{turnaround} = T_{completion}$  


## 7.3 FIRST IN FIRST OUT(FIFO)
1. It performs well in the 7.1 assumptions, but if we **relax** the assumption that all the jobs run the same time, FIFO perform poorly(convoy effect)


## 7.4 SHORTEST JOB FIRST(SJF)
1. it runs the shortest job first, then the next shortest, and so on.
2. if all jobs arrives at the same time, SJF is an optimal scheduling algorithm
3. now **relax** the assumption that jobs arrive at the same time, SJF is not optimal

## 7.5 SHORTEST TIME-TO-COMPLETION FIRST (STCF)
1. relax the assumption 3 that jobs must run to completion
2. any time a new job enters the ststem, the STCF scheduler determines which of the remaining jobs has the least time left, and schedules that one
3. STCF is a great policy when we know job lengths, and jobs only use CPU, and only metric is turnaround time


## 7.6 A New Metric: Response Time
1. response time is the time from when the job arrives in a system to the first time it is scheduled
2. $T_{response} = T_{firstrun} - T_{arrival}$

## 7.7 Round Robin
1. basic idea: instead of running jobs to completion, RR runs a job for a time slice and then switches to the next job in the run queue
2. however, RR performs badly considering turnaround time(it shows the trade-off between fairness and performance)


## 7.8 Incorporating I/O
1. relax assumption 4: no I/O
2. overlap


## 7.9 no more oracle
1. the worst assumption: os know the jobs length
2. how to overcome the problem: building a scheduler that uses the recent past to predict the future(**multi-level feedback queue**)

