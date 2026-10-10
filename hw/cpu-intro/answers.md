# OSTEP Homework: CPU Intro (process-run.py)

This is my first time learning how a CPU and I/O actually interact, so I wrote down my predictions honestly before running each command. All results below were produced with `python3 process-run.py ... -c -p` in the Codespace.

## Question 1

Command: `python3 process-run.py -l 5:100,5:100 -c -p`

**Prediction:** To be honest, this one felt simple to me. Both processes only use the CPU and neither has any I/O, so I guessed Process 0 would run its 5 instructions first, then Process 1 would run its 5. That would be 5 + 5 = 10 ticks total, with the CPU busy the whole time (100%).

**Reasoning:** Since there is no I/O at all, I thought the CPU would never have a reason to sit idle. The two processes just take turns.

**Verified result:** Total Time 10, CPU Busy 10 (100.00%), IO Busy 0 (0.00%).

**Analysis:** The result matched my guess, which honestly gave me a little confidence going into the later questions. With only CPU work, the total time is just the sum of all CPU instructions.

## Question 2

Command: `python3 process-run.py -l 4:100,1:0 -c -p`

**Prediction:** This is where I started to feel unsure. Process 0 runs 4 CPU ticks first. At tick 5, Process 1 issues its single IO and then has to wait 5 ticks for it to finish. Since there is no other process left to run, my guess was that the CPU just sits there doing nothing: 4 + 1 + 5 + 1 = 11 ticks, and the CPU is only busy about 54.5% of the time.

**Reasoning:** Under the default SWITCH_ON_IO policy, issuing an IO switches the CPU away from that process, but if nobody else is ready, the CPU can only wait. This part took me a while to truly picture in my head.

**Verified result:** Total Time 11, CPU Busy 6 (54.55%), IO Busy 5 (45.45%).

**Analysis:** The result matched. I had never really thought about how much a single IO-type process can drag down CPU utilization — just one IO dropped it to about 55%. It made me realize why I/O matters so much in real systems.

## Question 3

Command: `python3 process-run.py -l 1:0,4:100 -c -p`

**Prediction:** Now the IO process runs first and the CPU process runs after it. My first guess was that the CPU would switch to Process 1 while the IO runs in the background, so a good part of the IO time overlaps with CPU work: about 7 ticks in total, CPU around 85%.

**Reasoning:** To me, this felt like the first "overlap" case: the I/O and the CPU work happen at the same time instead of one waiting for the other.

**Verified result:** Total Time 7, CPU Busy 6 (85.71%), IO Busy 5 (71.43%).

**Analysis:** It came out as 7 ticks and 85.71%, with the same total work as Q2 but a completely different utilization. Comparing Q2 and Q3 side by side was the moment things started to click for me: it is not the workload that matters most, but whether the CPU can find something to do while the IO runs.

## Question 4

Command: `python3 process-run.py -l 1:0,4:100 -S SWITCH_ON_END -c -p`

**Prediction:** SWITCH_ON_END was new to me. My understanding is that the CPU is not handed over when the IO is issued; Process 1 just stays READY during the whole 5-tick IO. So my guess: 1 (io) + 5 (IO running while the CPU idles) + 1 (io_done) + 4 (CPU) = 11 ticks, CPU about 54.5%.

**Reasoning:** As I understood it, SWITCH_ON_END only switches when a process finishes completely, so the IO process keeps blocking the CPU process.

**Verified result:** Total Time 11, CPU Busy 6 (54.55%), IO Busy 5 (45.45%).

**Analysis:** The numbers came out at 11 ticks and 54.55%. What surprised me was how big the difference is between Q3 and Q4: the same two processes, the only change is the switching policy, and utilization drops from 85.71% back to 54.55%. I did not expect one flag to matter that much.

## Question 5

Command: `python3 process-run.py -l 1:0,4:100 -S SWITCH_ON_IO -c -p`

**Prediction:** Honestly, I expected this to be identical to Q3, because SWITCH_ON_IO is just the default policy written out explicitly: 7 ticks, CPU 85.71%.

**Verified result:** Total Time 7, CPU Busy 6 (85.71%), IO Busy 5 (71.43%).

**Analysis:** Exactly the same numbers as Q3. This was a small but satisfying confirmation for me: the default behavior of process-run.py really is SWITCH_ON_IO.

## Question 6

Command: `python3 process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p`

**Prediction:** With more processes, this one got harder for me to predict. Process 0 issues its first IO at tick 1, and the three CPU processes run their 15 ticks. Under IO_RUN_LATER, after each IO completes, Process 0 goes to the back of the ready queue. So I guessed its later IOs would overlap poorly: around 31 ticks, CPU about 68%.

**Reasoning:** My understanding is that LATER keeps pushing the IO process to the end of the line, so its second and third IOs mostly end up idling the CPU near the end.

**Verified result:** Total Time 31, CPU Busy 21 (67.74%), IO Busy 15 (48.39%).

**Analysis:** The result was 31 ticks and 67.74%. Tracing the timeline step by step in the output took me some time, but it helped me see how "go to the back of the queue" quietly wastes the chances to overlap I/O with CPU work.

## Question 7

Command: `python3 process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p`

**Prediction:** Same workload, but now IO_RUN_IMMEDIATE: each time Process 0's IO finishes, it jumps straight back to the CPU and issues its next IO immediately. My guess was about 21 ticks, with the CPU close to 100%.

**Verified result:** Total Time 21, CPU Busy 21 (100.00%), IO Busy 15 (71.43%).

**Analysis:** The result was 21 ticks and exactly 100%. Compared with Q6 (31 → 21, saving 10 ticks), this is the clearest evidence I have seen so far that simply letting the IO process come back right away keeps both the CPU and the I/O device busy.

## Question 8

Commands: `-s 1/2/3 -l 3:50,3:50` combined with the three policies (IO_RUN_LATER, IO_RUN_IMMEDIATE, SWITCH_ON_END), one output per policy appended to q8-s1/s2/s3.txt.

**Prediction:** For every seed, I expected SWITCH_ON_END to be the worst, since the CPU just sits idle during the IO. For IO_RUN_IMMEDIATE versus IO_RUN_LATER, I honestly was not sure — with only two processes, I guessed the difference would be small.

**Verified result (seed 1):** LATER: Total 15, CPU 53.33% | IMMEDIATE: Total 15, CPU 53.33% | SWITCH_ON_END: Total 18, CPU 44.44%
**Verified result (seed 2):** LATER: Total 16, CPU 62.50% | IMMEDIATE: Total 16, CPU 62.50% | SWITCH_ON_END: Total 30, CPU 33.33%
**Verified result (seed 3):** LATER: Total 18, CPU 50.00% | IMMEDIATE: Total 17, CPU 52.94% | SWITCH_ON_END: Total 24, CPU 37.50%

**Reasoning:** SWITCH_ON_END never overlaps IO with CPU work, so it is always the slowest. LATER and IMMEDIATE only differ in where the IO process is placed after its IO finishes; with two processes the queue is short, so the order barely matters.

**Analysis:** The pattern was the same for all three seeds: SWITCH_ON_END was 3, 14 and 6 ticks slower than the best policy, while IMMEDIATE tied LATER or won by at most 1 tick. This actually cleared up my own confusion: with few processes IMMEDIATE hardly helps, but in Q7, with four processes, it saved 10 ticks. My takeaway is that IO_RUN_IMMEDIATE becomes more valuable as more processes join, and SWITCH_ON_END is a policy I would simply never use.

Thank you for reading — this exercise genuinely helped me understand how the CPU and I/O interact. If any part of my reasoning is off, I would really appreciate any feedback.