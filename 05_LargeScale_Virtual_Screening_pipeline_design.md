# How to Build a 10-Million-Compound Screening Pipeline for Under $6,000 (Pipeline Design)
## — and Why You Probably Shouldn’t Screen All 10 Million Compounds Anyway 
________________________________________
## This article has two sections.
When my former PI moved to a new university, I rebuilt the laboratory from scratch and independently handled the entire planning and procurement process. 

I managed servers, workstations, RTX 3090 gaming PCs, Linux systems, and Windows systems.

* The first part covers hardware recommendations.
* The second part covers the actual pipeline design: how to classify compounds before large-scale virtual screening in order to reduce computation time.
________________________________________
## Chemical Space Navigation Pipeline v1.0
Let’s take a look at the actual pipeline.
(There will probably never be a Version 2. I’ve already escaped into the medical data field.)

Construct a searchable molecular index using shape, volume, and protonation-state annotations, then use neutral shape probes to estimate pocket preferences and eliminate obviously incompatible compounds before large-scale docking.

We have ten million little cars (Ligand).

Let’s start by removing the ones that obviously can’t fit into the parking lot (pocket).
________________________________________
### Step 00 — Download and Prepare the Compound Library
Download your compound library and perform energy minimization.

(For very large databases, choosing one that already provides preprocessed structures is probably more convenient. You can do it yourself if you really want to.)
________________________________________
## Step 01 — Build the Index

Create a searchable catalog.

You can store everything in a single database if you want.

For ten million compounds, however, separating the information may be more practical:
### File A
•	unique ID
•	Compound information

### File B
•	unique ID
•	Shape garage ID
•	Molecular volume
•	Charge category

Assign each compound a unique ID so the two files can be merged later.

The charge categories are:
•	Neutral
•	Positive
•	Negative
•	Zwitterionic (contains both positive and negative atoms)
________________________________________
## Step 01–1 — Generate Shape Garage IDs Using USR
(Ultrafast Shape Recognition)
The basic idea is:
Center of mass
→ Select representative points
→ Measure distance distributions
→ Convert them into a fixed-length vector
Using RDKit:
rdMolDescriptors.GetUSR()
This generates a shape descriptor.
Next, perform clustering and assign a Cluster_ID.
There are many clustering methods available.
Pick whichever one makes you happy.
The workflow is:
USR shape vector
→ Clustering
→ Cluster_ID
Congratulations.
You now have a shape garage number.
(You can call them BMW, Ferrari, or whatever makes your heart happy.)
________________________________________
## Step 01–2 — Calculate Molecular Volume
Use RDKit:
ComputeMolVolume()
or
MolVolume
The required 3D conformers should already exist from the energy-minimization step.
Output:
•	Compound_ID
•	Volume
Now every car knows whether it’s a compact car or a giant truck.
________________________________________
## Step 01–3 — Charge-State Classification
For this example, I will use physiological pH 7.4.
If your experiment uses a different pH, adjust accordingly.
Use:
Dimorphite-DL
to predict protonation states.
Examples:
CC(=O)[O-]
C[NH3+]
[NH3+]CC(=O)[O-]
You can write a simple Python script to classify compounds.
(I only know Python, so that’s what I’d do.)
Rules:
•	Positive charge only → Positive
•	Negative charge only → Negative
•	Both positive and negative charges → Zwitterionic
•	No formal charges → Neutral
Classification complete.
At this point, every compound has:
•	Shape garage ID
•	Volume ranking
•	Charge label
In other words:
•	Garage number
•	Vehicle size
•	Vehicle color
Perfect.
The parking lot management system is now operational.
________________________________________
## Step 02 — Two Possible Starting Points
There are two ways to begin.
Route A — You Already Have a Lead
Maybe you already know what kind of compounds you’re interested in.
Or maybe you’ve run an FDA drug screening campaign and found something that looks promising.
In that case:
Throw that compound into USR (Ultrafast Shape Recognition) and find its shape garage ID.
Maybe it’s a BMW.
Next:
Check whether your target pocket contains any important positively or negatively charged residues.
Use that information to choose the appropriate charge category (different paint colors).
For example:
 
If the pocket contains an important positively charged residue,
pull out the positively charged BMW cars.
Then:
Start crashing cars into the parking lot in order of molecular volume.
Check every morning to see how far you’ve gotten.
Eventually you’ll reach compounds so large that they couldn’t fit into the pocket even if they curled themselves into a ball.
Congratulations.
Work finished.
Time for analysis.
(Of course, if there aren’t many cars in that garage, you can always dock all of them.)
Route B — You Only Have a Pocket
You know absolutely nothing.
You have a pocket.
You want to dock everything.
In that case:
Pull out all the neutral-colored cars from every shape garage.
If one garage doesn’t contain any neutral-colored cars, maybe it’s some weird mutant branch of a miracle drug family and you can design a scaffold for it later.
Personally, I’m too lazy to deal with that, so I’d just include everything.
You can also put the garage ID directly into the compound name if that helps with tracking.
Honestly though, we already built a catalog, so whether you do this or not doesn’t really matter.
Deploy the Probe Cars
Launch the probe cars.
This stage should be run completely.
Process them in order of molecular volume.
Check every morning to see how far you’ve gotten.
Eventually you’ll reach compounds so large that they couldn’t fit into the pocket even if they curled themselves into a ball.
Congratulations.
Work finished.
Time for analysis.
(Of course, if there aren’t many cars, you can still dock everything.)
________________________________________
Analysis
A. Volume Analysis
At some point you’ll discover the largest compound size the pocket can realistically accommodate.
Once you know that number, go back to your catalog.
Every compound larger than that volume can be eliminated immediately.
No docking required.
B. Score Ranking
Hmm… let me think.
B-1
Take the top 10,000 ranked compounds.
B-2
Look at those top-ranked cars and analyze which garages appear most frequently.
Now you can generate a ranking based on:
•	High-scoring garages
•	Frequently appearing garages
Those rankings can be used as references for the next round.
Now we can start docking for real.
Pull out the garages that performed well.
BMW.
Ferrari.
Maserati.
Whatever showed up near the top.
Then:
Check whether your pocket contains important positively or negatively charged residues.
Use that information to choose the appropriate charge category.
For example:
If an important residue is positively charged,
pull out the negatively charged cars.
And don’t forget:
Eliminate everything larger than the maximum volume identified during Volume Analysis.
Dock.
________________________________________
Analyze.
Go home.
You really don’t need to screen all ten million compounds.
What Are the Neutral Probe Cars Actually For?
The purpose of the neutral-colored probe cars is:
1.	To estimate the pocket’s shape preference
2.	To estimate the pocket’s maximum volume
Once you’ve identified strong van der Waals packing preferences, you can then introduce charged compounds to capture important residues.
(You should already know what kind of electrical outlets exist inside your parking lot.)
That’s how you obtain high-affinity docking candidates.
At this point you can begin analyzing the data.
________________________________________
If you’re still worried, you can always keep the full ten-million-compound screening running in the background while you analyze.
But remember:
You already know:
•	The maximum acceptable molecular volume
•	The preferred charge characteristics of the pocket
Therefore:
•	Compounds larger than the volume limit don’t need to be screened.
•	Compounds with incompatible charge states don’t need to be screened.
A large fraction of the library can be removed before docking even begins.
________________________________________
But Honestly?
I genuinely don’t think you need to screen everything.
I’ve spent 11.5 years working in molecular dynamics.
You’re asking me to believe that a pocket has enough superhuman gripping power to hold onto the head of a ligand while the rest of the molecule hangs outside waving around in the wind —
and somehow the ligand still doesn’t leave?
At that point, I start suspecting the protein might be an alien life form.
I’d be very interested in meeting whatever organism owns that protein.
Because docking isn’t the end of the story.
The ligand still has to do its job.
It has to remain bound long enough for inhibition or activation to occur.
So you’re asking me to believe that:
•	The dangling tail never gets pulled away by something else.
•	The flexible tail never wiggles the ligand out of the pocket.
•	The entire complex stays stable despite half the molecule floating around outside.
=_= ????????
That’s not a protein.
That’s a Saiyan protein.
