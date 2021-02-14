# homework answer for chapter 7
1. 
   ```sh
   ./scheduler.py -p FIFO -l 200,200,200 -c
   Final statistics:
    Job   0 -- Response: 0.00  Turnaround 200.00  Wait 0.00
    Job   1 -- Response: 200.00  Turnaround 400.00  Wait 200.00
    Job   2 -- Response: 400.00  Turnaround 600.00  Wait 400.00

    Average -- Response: 200.00  Turnaround 400.00  Wait 200.00
   ```
   ```sh
   ./scheduler.py -p SFJ -l 200,200,200 -c
   Final statistics:
    Job   0 -- Response: 0.00  Turnaround 200.00  Wait 0.00
    Job   1 -- Response: 200.00  Turnaround 400.00  Wait 200.00
    Job   2 -- Response: 400.00  Turnaround 600.00  Wait 400.00

    Average -- Response: 200.00  Turnaround 400.00  Wait 200.00
   ```
2. ```sh
   ./scheduler.py -p FIFO -l 200,200,200 -c
   Final statistics:
    Job   0 -- Response: 0.00  Turnaround 100.00  Wait 0.00
    Job   1 -- Response: 100.00  Turnaround 300.00  Wait 100.00
    Job   2 -- Response: 300.00  Turnaround 600.00  Wait 300.00

    Average -- Response: 133.33  Turnaround 333.33  Wait 133.33
   ```
   ```sh
   ./scheduler.py -p SFJ -l 200,200,200 -c
   Final statistics:
    Job   0 -- Response: 0.00  Turnaround 100.00  Wait 0.00
    Job   1 -- Response: 100.00  Turnaround 300.00  Wait 100.00
    Job   2 -- Response: 300.00  Turnaround 600.00  Wait 300.00

    Average -- Response: 133.33  Turnaround 333.33  Wait 133.33
   ```
3. ```sh
    ./scheduler.py -p RR -q 1 -l 100,200,300 -c
   Final statistics:
    Job   0 -- Response: 0.00  Turnaround 298.00  Wait 198.00
    Job   1 -- Response: 1.00  Turnaround 499.00  Wait 299.00
    Job   2 -- Response: 2.00  Turnaround 600.00  Wait 300.00

    Average -- Response: 1.00  Turnaround 465.67  Wait 265.67
   ```
4. Jobs arranged in increasing length
5. workloads are all the same and are equal to quantum length
6. response time will increase of course
7. response time will increase. And the $n^{th}$ job's response time is $(n-1)q$, average response time is $(n-1)q/2$
