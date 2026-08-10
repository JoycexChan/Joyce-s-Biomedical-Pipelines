# Building a Pathway Analysis Workflow for Investigating Mutation Effects from Molecular Dynamics Trajectories
________________________________________
This analysis approach can be used to quickly assess whether molecular dynamics trajectories exhibit the expected **pathway**, as well as whether the trajectories contain structural indicators that differ between conditions.

However, it also has one obvious limitation:
**The user must be able to interpret possible interactions between amino acids from their positions in the 3D structure, and understand what these interactions may mean for structural motions.**

For graduate students who are just beginning to perform molecular dynamics analysis, this is usually not easy.

I developed this analysis approach during my first year after completing my master's degree. The main purpose at the time was to avoid manually inspecting large numbers of trajectories and calculating hydrogen-bond or other interaction percentages, only to discover afterward that a particular set of trajectories did not actually exhibit the expected phenomenon.
________________________________________
## 500 residues × multiple interaction sites × WT/MT comparison × manual Excel processing.

The sheer volume of data involved in the manual analysis was more than enough motivation to improve the workflow.

Many newer tools are likely available by 2026, so the specific tools used here should not be considered essential or fixed.

This article can therefore be viewed as a historical record of a molecular dynamics rapid-analysis workflow centered on **human visual inspection and literature-based interpretation**. The tools themselves are replaceable; the workflow and reasoning process are the main focus.

After writing this article, I also came up with an idea for further automating part of the analysis using Python. If this approach is useful to you, the next article will explore **how to identify potentially critical amino acids from differences in interactions between WT and MT**.

If this article uses **human visual inspection and literature information to infer the pathway**, the next article will use Python to **narrow down the range of amino acids potentially affected by the mutation**. This effectively introduces an analyzable intermediate layer between the starting and ending points of the pathway, making it easier for graduate students who are new to molecular dynamics analysis to infer the overall pathway without relying entirely on manual observation.

________________________________________
## Analysis Concept

The overall workflow can be summarized as:

**Literature → Structural Hypothesis → Simulation Trajectories → PCA → Structural Motion Analysis → Pathway Construction → Hypothesis Formation → Identification of Key Amino Acids**

The most important concept is:

> **Molecular dynamics simulation is not an isolated or fictional tool independent of experimental evidence and the literature. It is a further analysis built upon known structural, functional, and experimental results.**

Therefore, before beginning the analysis, it is necessary to establish a **“virtual framework”** for the protein's structure and function.

This can be thought of as assembling LEGO:

* The literature provides the known pieces.
* Structural and functional studies provide the relationships between those pieces.
* Molecular dynamics simulations generate large numbers of dynamic building blocks.
* These pieces are then assembled to construct a pathway describing how the mutation may alter the structure.

________________________________________

# Environment Setup

The following is the old environment used at the time. **It is not recommended to reproduce this environment for new analyses.**

### Python

`Anaconda2-2019.10-Windows-x86_64`

Python 2.7

Environment variable Path:

```text
C:\Users\joyce\Anaconda2
C:\Users\joyce\Anaconda2\Scripts
C:\Users\joyce\Anaconda2\Library\bin
```

Open:

```text
Anaconda Navigator (Anaconda2)
→ Home
→ CMD.exe
```

Install:

```bash
pip install prody
pip install Pmw
pip install Numpy
```

### Tools Used

```text
ProDy-1.9.3.win-amd64-py2.7
VMD 1.9.1 for Windows
PyMOL 0.99rc6v3
```

________________________________________
## Materials

At minimum, two sets of simulation results are required:

**WT (wild type)** and **MT (mutant)**

so that they can be compared.

### cMD

Sample approximately **500 frames** at equal intervals from a region where the system has already reached equilibrium, such as the final 100 ns.

### GaMD

For GaMD, frame selection is somewhat more involved. The energy landscape usually has specific structural or collective-variable definitions along the X and Y axes.

First, identify the minimum-energy point and use it as the reference point, defined as (0, 0). Based on the target X and Y ranges, identify the surrounding low-energy basin around this minimum and extract the region of interest.

The candidate frames within this basin are then ranked according to their distance from the minimum-energy reference point. Approximately 500 frames are selected at equal intervals from the ranked candidates.

________________________________________

## 1. Establish a Literature- and Structure-Based Virtual Framework

Before beginning the analysis, collect literature related to the protein, especially information concerning:

* Structural information at the amino-acid level
* Functional effects caused by mutations
* Known protein interactions
* Relationships between structure and function
* Experimental results

For example:

**Structural Basis for Assembly and Activation of the Heterotetrameric SAGA Histone H2B Deubiquitinase Module**

What information can be obtained from this paper?

For example:

* Sgf73 plays an important role in Ubp8 activation.
* Deletion of Sgf73 residues 1–89, equivalent to the H93A point mutation, abolishes DUBm activity.
* The ZnF domain of Sgf73 requires physical association with the assembly lobe (Sus1) to perform its function and cannot independently allosterically activate Ubp8.

Other information is omitted here.

These pieces of information form the **“virtual framework”** for the subsequent analysis.

### Why Start with the Literature?

Think of it as assembling LEGO.

You first need to collect all the known pieces:

> **Literature → Build the virtual framework**

Then molecular dynamics simulations generate large numbers of trajectories:

> **Simulation → Obtain dynamic building blocks**

Finally, these pieces of information are assembled to determine how the mutation may alter structural motions.

________________________________________

## 2. Obtain 3D Structural Trajectories After the Simulation

Open CMD on the computer where ProDy is installed.

First, align all trajectories:

```bash
prody align WT.pdb -m 1 -p ALG
```

Then perform PCA using the Cα atoms of all amino acids:

```bash
prody pca -a -A -F png -z -J "1 2 3 1,2" -s ca --aligned ALG_aligned.pdb
```

Finally, use IPython to obtain the variance of PC1–PC10:

```bash
ipython --pylab
```

```python
from prody import *
from pylab import *
ion()

pca = loadModel('ALG_aligned_pca.pca.npz')

for mode in pca[:10]:
    print(calcFractVariance(mode).round(3))
```

### What Each Step Does

The first command:

> Aligns all trajectories to the first structure.

The second command:

> Performs PCA using the Cα atoms of all amino acids.

The third code block:

> Obtains the proportion represented by each principal component from PC1 to PC10.

An NMD file will be generated.

Open the file using VMD to visualize the dynamic motion of the 3D structure.

________________________________________

## 3. Confirm That the Simulation Exhibits the Expected Features

First, examine whether WT and MT exhibit their expected characteristic features.

For example, suppose the expected result is:

> The finger closes after mutation, and the catalytic triad disappears.

Then, theoretically:

* WT should not exhibit this phenomenon.
* MT should exhibit this phenomenon.

If the simulation results do not match the expected behavior, first check and adjust the simulation settings.

**Do not continue building a pathway based on an incorrect simulation result.**

If the expected features are present, proceed to the next stage.

________________________________________

## 4. Infer the WT and MT Pathways

Open two separate VMD sessions and load the WT and MT trajectories.

Play the trajectories and observe the relationships between the motions of different amino acids and secondary structures.

One important observation principle is:

> When two amino acids form an interaction, the corresponding secondary structures should be examined for coordinated motion.

Conversely, when two amino acids do not form an interaction, the corresponding secondary structures should also be examined to determine whether they exhibit opposing motions.

Next, bring back the **“virtual framework”** established from the literature in the previous step.

Play the WT and MT structural trajectories and begin constructing the pathway:

> Mutation
> → Which interactions are formed?
> → Which interactions are disrupted?
> → Which structural elements begin to move?
> → How does a new MT pathway emerge?
> → How does it ultimately lead to disruption of the catalytic triad?

If the observed pathway corresponds with the structural and functional framework established from the literature, proceed to the next step.

________________________________________

## 5. Formulate a Mechanistic Hypothesis for the Mutation

For example, the following hypothesis can be formulated:

> After the mutation occurs, structural disorder in the Sgf73 ZnF reduces its ability to stabilize Sgf11, causing Sgf11 to favor a state with increased interaction with the Ubp8 Assembly lobe.

Therefore:

> Increased interaction between the N-terminus of the Sgf11 long loop and the Ubp8 Assembly lobe
> ↓
> Sgf11 moves toward the left / Ubp8 Assembly lobe
> ↓
> This may drag the β14–β15 loop (N443) away from H427
> ↓
> Disruption of the catalytic triad

The key point here is not to directly claim that **“a particular amino acid definitely causes the outcome.”**

Instead, the goal is to use:

**literature + structural motions observed in the trajectories + changes in interactions**

to gradually construct a mechanistic hypothesis that can be further tested.

________________________________________

## 6. Identify Key Amino Acids

Finally, return to the pathway and identify amino acids that may serve as key nodes.

Two main types of information should be examined:

### Increased Interactions

Which amino acids show increased interactions in MT?

### Disrupted Interactions

Which interactions present in WT disappear in MT?

These can then be further quantified using:

* Distance
* Interaction percentage
* Other appropriate structural or interaction-based metrics

Finally, compare these quantitative results with the pathway established above.

________________________________________

## Why Use 500 Trajectories?

This is also one of the key design considerations of this workflow.

If only a small number of trajectories, such as 25, are manually inspected, the process can easily become:

> Inspect a few trajectories → Guess the key amino acids → Extract data → Find that the hypothesis does not fit → Guess again → Extract more data

This process relies heavily on human intuition and can lead to repeated modification of the hypothesis.

By initially using approximately **500 trajectories** to establish the overall structural motion, and then identifying changes in interactions, it is possible to first construct a more comprehensive candidate pathway from the larger trajectory set and then perform quantitative validation at the key locations.

Therefore, the advantage is not:

> **“500 trajectories are guaranteed to be correct.”**

Rather:

> **Compared with inspecting only a small number of trajectories, establishing the overall motion pattern from a larger trajectory set before searching for key amino acids can substantially reduce the need to repeatedly guess based on a small number of frames.**

This changes the workflow from:

> **Inspect trajectories → Guess → Extract data → Guess again**

to:

> **Large trajectory set → Establish motion patterns → Construct pathway → Identify key interactions → Quantitative validation**

This was the actual problem this workflow was designed to address:

> **The goal was not to make the analysis more complicated, but to avoid manually analyzing a large number of trajectories only to discover that the phenomenon originally hypothesized was not actually present.**
