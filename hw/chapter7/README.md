# OSTEP Chapter 7: CPU Scheduling Homework

This directory contains my completed OSTEP Chapter 7 (CPU Scheduling) homework.

- `analysis.md` — written answers to all 7 questions, each with my Prediction, Reasoning, Verified result and Analysis
- `data/` — raw simulator output for every question, produced by the official OSTEP `scheduler.py`

All results were generated with the real OSTEP scheduler simulator, following the two-step workflow from the assignment guide: first run each command without `-c` and write down my prediction, then rerun with `-c` to verify.

## Results Summary

| Question | Workload / Policy | Avg Response | Avg Turnaround | Key Finding |
|---|---|---|---|---|
| Q1 | 200,200,200 — FIFO and SJF | 200.00 | 400.00 | With equal-length jobs SJF degenerates into FIFO: identical numbers |
| Q2 | 100,200,300 — FIFO and SJF | 133.33 | 333.33 | Jobs arrive in increasing order, so FIFO happens to match SJF |
| Q3 | 100,200,300 — RR, q=1 | 1.00 | 465.67 | Fairness trade-off: response improves 133x, turnaround ~40% worse |
| Q4 | 100,100,100 — FIFO and SJF | 100.00 | 200.00 | SJF gives the same turnaround as FIFO when all jobs are equal length (or arrive shortest-first) |
| Q5 | 100,200,300 — RR q=300 vs SJF | 133.33 | 333.33 | When the quantum covers the longest job, RR degenerates to FIFO; with shortest-first arrival it equals SJF exactly (even per-job numbers) |
| Q6 | 100/200/300 -> 200/400/600 -> 400/800/1200, SJF | 133.33 -> 266.67 -> 533.33 | 333.33 -> 666.67 -> 1333.33 | SJF response time grows linearly with job length |
| Q7 | 100,100,100 — RR, q = 1/2/5/10 | 1 / 2 / 5 / 10 | 299 -> 290 | RR response time grows linearly with the quantum. Worst case with N equal jobs: T_response = (N-1) x q, average = (N-1) x q / 2 |

## File Index (data/)

| File | Content |
|---|---|
| q1_fifo.txt, q1_sjf.txt | Q1: three 200-length jobs under FIFO and SJF |
| q2_fifo.txt, q2_sjf.txt | Q2: 100/200/300 jobs under FIFO and SJF |
| q3_rr.txt | Q3: RR with quantum 1 on 100/200/300 |
| q4_fifo.txt, q4_sjf.txt | Q4 verification: equal-length jobs under both policies |
| q5_rr_q300.txt, q5_sjf.txt | Q5 verification: RR with quantum 300 vs SJF |
| q6_base.txt, q6_double.txt, q6_quad.txt | Q6: SJF trend as job lengths double and quadruple |
| q7_q1.txt, q7_q2.txt, q7_q5.txt, q7_q10.txt | Q7: RR response time with quantum 1/2/5/10 |
