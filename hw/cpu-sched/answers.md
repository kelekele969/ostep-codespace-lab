# OSTEP Homework: CPU Scheduling (scheduler.py)

This is my first time studying CPU scheduling policies, so I wrote down my predictions honestly before running each command. All results below were produced with `python3 scheduler.py ... -c` on the real OSTEP scheduler simulator.

## Question 1

Task: Run 3 jobs of length 200 with FIFO and SJF, and compute the response and turnaround times.

Commands: `-l 200,200,200 -p FIFO -c` (q1-fifo.txt) and `-l 200,200,200 -p SJF -c` (q1-sjf.txt)

**Prediction:** Since all three jobs have the exact same length, I expected FIFO and SJF to give identical results. SJF wants to pick the shortest job, but when everyone is the same length there is nothing to pick — it just follows the arrival order like FIFO. Each job waits for the previous two to finish, so my guess: average response = (0 + 200 + 400) / 3 = 200, average turnaround = (200 + 400 + 600) / 3 = 400.

**Reasoning:** FIFO and SJF only differ in the order they pick jobs. With equal lengths, that difference disappears.

**Verified result:** FIFO: Response 200.00, Turnaround 400.00. SJF: Response 200.00, Turnaround 400.00.

**Analysis:** Exactly as predicted. This was a nice gentle start: with equal-length jobs, SJF simply degenerates into FIFO.

## Question 2

Task: Same as Q1, but with jobs of length 100, 200 and 300.

Commands: `-l 100,200,300 -p FIFO -c` (q2-fifo.txt) and `-l 100,200,300 -p SJF -c` (q2-sjf.txt)

**Prediction:** I noticed the jobs arrive in increasing order (100 → 200 → 300), which is exactly the order SJF would choose. So I expected FIFO and SJF to be identical again: average response = (0 + 100 + 300) / 3 = 133.33, average turnaround = (100 + 300 + 600) / 3 = 333.33.

**Verified result:** Both policies: Response 133.33, Turnaround 333.33.

**Analysis:** Matched. The jobs just happened to arrive in the best possible order. I also tried reversing the list in my head (300, 200, 100): FIFO would then suffer the convoy effect with average turnaround 466.67, while SJF would still give 333.33. That mental experiment is when the textbook's claim "SJF is optimal for turnaround" finally clicked for me.

## Question 3

Task: Run the same 100/200/300 workload with RR and a time slice (quantum) of 1.

Command: `-l 100,200,300 -p RR -q 1 -c` (q3.txt)

**Prediction:** RR cycles through the jobs, so every job should get its first slice almost immediately. I expected a tiny average response (around 1), but a noticeably worse turnaround, since no job gets to run to completion early and the last long job keeps being pushed back.

**Reasoning:** Response time is about "how long until I first run" — under RR with q=1 the third job first runs at time 2, so the average should be about 1. Turnaround is about "how long until I finish" — and under RR the long job finishes later than under SJF.

**Verified result:** Average Response 1.00, Average Turnaround 465.67 (Wait 265.67).

**Analysis:** Response dropped from 133.33 to 1.00 — a 133x improvement — but turnaround grew from 333.33 to 465.67, about 40% worse. This is exactly the fairness trade-off from the chapter: RR makes everyone start quickly, but everyone also finishes later.

## Question 4

Task: For what kind of workload does SJF provide the same turnaround time as FIFO?

**My answer:** When all jobs have the same length, or when jobs happen to arrive in increasing order of length (shortest first).

**Verification:** Q1 (all 200s) and Q2 (100, 200, 300) already showed FIFO = SJF. I ran one more check with three equal jobs of length 100 (`-l 100,100,100 -p FIFO / SJF -c`, q4-fifo.txt / q4-sjf.txt): both give Response 100.00, Turnaround 200.00.

**Analysis:** The only thing SJF changes compared to FIFO is the order of picking jobs. If the arrival order is already the order SJF would pick (all equal, or increasing), the two policies produce identical schedules and every metric matches. The moment a shorter job arrives after a longer one, FIFO suffers the convoy effect while SJF does not — that is the only case where they differ.

## Question 5

Task: For what kind of workload and quantum length do SJF and RR provide the same response time?

**My answer:** When the quantum is at least as long as the longest job, and the jobs arrive in increasing order of length. With such a large quantum, every job runs to completion in its first slice, so RR degenerates into plain FIFO; combined with shortest-first arrival, it becomes exactly SJF.

**Verification:** `-l 100,200,300 -p RR -q 300 -c` (q5-rr300.txt) gives Response 133.33, Turnaround 333.33 — identical to the SJF run (q5-sjf.txt), including every per-job number.

**Analysis:** RR's whole point is to interleave jobs with small slices. Once the quantum covers a job's entire length, that interleaving disappears and RR silently becomes FIFO. It reminded me that RR is only "fair" relative to the quantum you choose.

## Question 6

Task: What happens to SJF's response time as job lengths grow? Show the trend with the simulator.

Commands (q6-base.txt / q6-double.txt / q6-quad.txt): `-l 100,200,300`, `-l 200,400,600`, `-l 400,800,1200`, all with `-p SJF -c`

**Prediction:** Doubling every length should double every waiting interval, so the average response should scale linearly with the job lengths.

**Verified result:** Average Response: 133.33 → 266.67 → 533.33 — exactly 2x each time.

**Analysis:** Under SJF, the k-th job's response time is the sum of the lengths of all jobs scheduled before it. Since every term in that sum scales with job length, average response is a linear function of the lengths. SJF is great for turnaround but does nothing special for response — the last (longest) job always waits for everyone else.

## Question 7

Task: What happens to RR's response time as the quantum grows? Write an equation for the worst-case response time given N jobs.

Commands (q7-q1/q2/q5/q10.txt): `-l 100,100,100 -p RR` with q = 1, 2, 5, 10

**Verified result:** Average Response: q=1 → 1.00, q=2 → 2.00, q=5 → 5.00, q=10 → 10.00. Average turnaround moves the other way: 299.00 → 298.00 → 295.00 → 290.00.

**Analysis:** With 3 equal jobs the three jobs first run at times 0, q, 2q, so average response = q*(0+1+2)/3 = q — response time grows linearly with the quantum.

**Worst-case equation:** With N jobs arriving at the same time, each at least as long as the quantum, the unluckiest job is the one scheduled last: it must wait for the other N-1 jobs to each complete one full slice before its first slice. So:

  T_response(worst) = (N - 1) x q

and the average response time is (N - 1) x q / 2.

Checking with q=5, N=3 (q7-q5.txt): Job 2 first runs at time 10 = (3-1) x 5, and the average response is (0+5+10)/3 = 5 = (N-1)q/2. Both match.

**Analysis:** Larger quanta make RR behave more and more like FIFO: response times creep up linearly. Smaller quanta keep response low but cost more context switches in a real system. The quantum is essentially a knob that trades responsiveness against switching overhead.

Thank you for reading — this exercise made the scheduling trade-offs much more concrete for me. If any part of my reasoning is off, I would really appreciate any feedback.
