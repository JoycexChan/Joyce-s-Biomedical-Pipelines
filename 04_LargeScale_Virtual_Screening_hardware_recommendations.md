# How to Build a 10-Million-Compound Screening Pipeline for Under $6,000 — and Why You Probably Shouldn’t Screen All 10 Million Compounds Anyway (Hardware Recommendations)
________________________________________
## This article has two sections.
* The first part covers hardware recommendations.

When my former PI moved to a new university, I rebuilt the laboratory from scratch and independently handled the entire planning and procurement process. 

I managed servers, workstations, RTX 3090 gaming PCs, Linux systems, and Windows systems.

* The second part covers the actual pipeline design: how to classify compounds before large-scale virtual screening in order to reduce computation time.
________________________________________
## Hardware Recommendations

### 10,000 FDA-approved drugs at exhaustiveness 16, the job took roughly three days.

Let’s start with AutoDock.

* According to this paper:
Large benchmark tests show that Vina-GPU reaches an average of 21.66× and a maximum of 50.80× speed-up on NVIDIA GeForce RTX 3090 against the original AutoDock Vina on a 20-threaded CPU while ensuring their comparable docking accuracy.
https://pmc.ncbi.nlm.nih.gov/articles/PMC9103882/

We can see that using an RTX 3090 provides approximately 21.66×–50.80× acceleration compared with CPU docking.

The RTX 5090 (memory bandwidth: 1792 GB/s) has approximately 1.91× the memory bandwidth of the RTX 3090 (936 GB/s).

Of course, docking workloads are not entirely GPU-bound, so actual performance gains will not scale perfectly. There will always be some overhead.

Back when I was running a library of 10,000 FDA-approved drugs at exhaustiveness 16, the job took roughly three days.
________________________________________
### 10 million compounds at exhaustiveness 16, the job took roughly one month to two and a half months.

For ultra-large virtual screening projects, you can usually lower the exhaustiveness setting a bit. Screening ten million compounds doesn’t require the same level of precision as a final lead optimization stage.

* Using the acceleration numbers above:
10,000 compounds on an RTX 3090 would likely take approximately 1.42–3.32 hours.

* Scaling that estimate to an RTX 5090:
10,000 compounds may take roughly 0.74–1.74 hours.

* What about 10 million compounds?
Approximately:
740–1740 hours or 30.8–72.5 days
  
In other words:
One month to two and a half months.

That’s already within a range many academic labs can tolerate.
________________________________________
## A gaming workstation (RTX 5090) usable for around five years
A gaming workstation equipped with an RTX 5090 costs roughly NT$200,000 and should remain usable for around five years.

If it breaks during the warranty period, send it back for repair.

After five years? 

Cross your fingers and pray to the hardware gods. On the bright side, academia usually foots the electricity and air conditioning bills, so you can run that 5090 at full throttle without burning a hole in your own pocket.

I also recommend setting up a system recovery image. 

If the problem is purely software-related, restoring the system from the recovery image is much faster than waiting two weeks for IT support.
________________________________________
## Pipeline Design (Reducing Runtime to a Few Weeks)

However, our actual goal is preventing the PI from constantly asking:
“Is it done yet?”

Ideally, the runtime should be reduced to a few weeks rather than a few months.

That’s where Part 2 comes in.

The compound classification strategy described in Part 2 should significantly reduce the overall runtime.
________________________________________
## Hundred-million-compound Libraries (Hardware Recommendations)

If you want to move into hundred-million-compound libraries, you’ll probably need a NAS and multiple RTX 5090 systems running in parallel.

Network bandwidth starts to matter at that point.

If expansion is part of your future plans, upgrade the NAS first.

Also pay attention to the network cards in your workstations.

Otherwise, you might discover that your GPUs are fast but your data transfer isn’t.

A setup like this can easily cost over $30,000.

Not exactly cheap.
