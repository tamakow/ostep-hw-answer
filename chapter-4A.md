# ANSWER of chapter 4

1.  100% , because there is no IO process
2.  total time is 11 clock ticks with 4(prcess 0) and 7(process 1)
3.  total time changes to 7 clock ticks from 11. switching the order matters. Because process 1 can run while process 0 is waiting for IO completes
4.  total time is 11 clock ticks because process 1 only runs when process 0 is completely finished(the affection of -S SWITCH_ON_END flag)
5.  total time is 7 clock ticks, and the reason is the same as answer 3 (the affection of -S SWITCH_ON_IO flag)
6.  total time is 26 clock ticks. Process 0 runs the first IO then maintains ready state until the other processes finish. System resources are not being effectively utilized
7.  total time is 21 clock ticks, which is better than using -I IO_RUN_LATER. Because other processes can run while process 0 is waiting IO.
8. three processes are generally the same, so take the first one as an example (-s 1 -l 3:50,3:50). -I flag doesn't matter here because process 0 is 1 cpu and 2 io,process 1 is 3 cpu, which means that process 1 can be done before the first IO finishs. -S flag matters and if -S SWITCH_ON_END is chosen total time will increase 3 more clock ticks(process 1)