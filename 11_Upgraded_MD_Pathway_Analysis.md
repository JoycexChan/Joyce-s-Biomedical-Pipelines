# How to Identify Potentially Critical Amino Acids from Differences in Interactions Between WT and MT

This approach uses **MD-TASK** to obtain the percentage of frames in which each pair of nodes forms an edge in the dynamic structural network.

The interaction network is constructed by calculating the distance between the **Cα and Cβ atoms** of amino acids.

A distance of approximately **6.5–7.5 Å** can be used as the criterion for an interaction.

I usually use **6.5 Å**, but this threshold can be adjusted according to your preference and analysis requirements.

The overall idea is to progressively narrow down the search space:

> **Structure Unit → Residue → Atom**

Instead of manually checking a large number of amino-acid pairs, the analysis first identifies which structural regions are affected by the mutation, then drills down to the relevant residues, and finally examines atom-level distances for the remaining candidates.

---

## This level of analysis is sufficient to reduce the amount of domain-specific background knowledge required for manual interpretation.

Instead of asking the analyst to infer the entire pathway directly from a complex 3D trajectory, the protein can first be divided into structural building blocks. The analysis then helps identify the key intermediate amino acids connecting these blocks.

The resulting pathway can be simplified into:

Mutation Site → Key Intermediate Residues → Active Site

These structural building blocks provide the intermediate steps needed to connect the initial mutation to the final functional effect, allowing the complete pathway to be reconstructed as a coherent mechanistic story.

---

## Environment Setup

First, check your current working directory:

```bash
pwd
```

> **Note:** Do not install Python 3.8 for this workflow. It may cause compiler-related errors with the required environment.

---

### 1. Install Python 3.7

Install Python 3.7:

[Python 3.7 installation reference](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/711056/)

Set the path according to the actual installation location:

```bash
export PATH=$PATH:/usr/local/bin/python
```

---

### 2. Install MD-TASK

Install **MD-TASK**.

The installation may take some time, so it may be worth making a backup of the complete installation once everything is working correctly.

---

### 3. Install NumPy and Related Packages

Try upgrading NumPy first:

```bash
pip install --upgrade numpy
```

If additional system packages are required:

```bash
yum install blas-devel lapack-devel
```

Then:

```bash
pip install numpy
pip install scipy
pip install scikit-learn
```

---

## 4. Install R

For CentOS:

```bash
sudo yum install epel-release
sudo yum update
sudo yum install R
```

However, the repository version may not be the latest version of R available at the time.

If a specific version is required, download the corresponding R source package manually.

For example:

```bash
wget https://cran.r-project.org/src/base/R-3/R-3.3.2.tar.gz
```

Extract it:

```bash
tar -zxvf R-3.3.2.tar.gz
cd R-3.3.2
```

Install it into the default directory:

```bash
./configure --prefix=/opt/R --with-readline=yes --with-x=yes --enable-R-shlib
make
make install
```

R installation is then complete.

---

### 5. Virtual Environment

Enter the `env1` virtual environment:

```bash
virtualenv env1
```

Then activate it:

```bash
source env1/bin/activate
```

The exact virtual-environment setup may vary depending on the system configuration.

---

### 6. MD-TASK

For MD-TASK installation, refer to the official documentation:

[MD-TASK documentation](http://md-task.readthedocs.io/en/latest/index.html)

For example:

```bash
cd /root/MD-TASK/example/PINK1/618_PINK/WT
```

Activate the MD-TASK environment:

```bash
. /root/MD-TASK/venv/bin/activate
```

Then run:

```bash
/root/MD-TASK/calc_network.py \
--topology WT.pdb \
--threshold 6.5 \
--step 1 \
--generate-plots \
--calc-BC \
--calc-L \
--lazy-load WT.dcd
```

---

### 7. Install Gephi

On a Windows computer, install:

**Gephi**

Gephi will be used to visualize and inspect the dynamic structural network generated from MD-TASK.

---

## Materials

You will need:

```text
WT.pdb
WT.dcd
MT.pdb
MT.dcd
```

where:

* `WT.pdb` represents the first frame of the 500-frame trajectory.
* `WT.dcd` contains the 500-frame trajectory.
* `MT.pdb` and `MT.dcd` contain the corresponding mutant structure and trajectory.

### Important: Keep Only Cα and Cβ Atoms

Before running MD-TASK, keep only the **Cα and Cβ atoms**.

This can be handled directly using `cpptraj`.

The purpose is to simplify the structural representation used for the interaction network.

---

## Generate the Dynamic Structural Network with MD-TASK

Run MD-TASK on the WT and MT trajectories.

The important output for the next stage is the percentage of frames in which each pair of nodes forms an edge.

Conceptually:

```text
Node A ───────── Node B
       interaction
       percentage
```

For example:

```text
Residue A ↔ Residue B
        73.4%
```

This percentage represents how frequently the interaction exists throughout the trajectory.

### Correct the Amino-Acid Numbering

At this stage, make sure that the amino-acid IDs correspond to the **actual numbering of the protein**.

This correction is intentionally performed here rather than modifying the trajectory itself.

Changing the residue numbering inside the trajectory can be tedious, and making a mistake at that stage can potentially contaminate every subsequent analysis.

Therefore, instead of modifying the trajectory, use the WT PDB generated for this step and correct the amino-acid IDs here.

This gives you a clean mapping between:

```text
MD-TASK node
      ↓
Actual protein residue
```

---

## Prepare the Residue Annotation Table

Create a CSV file containing the amino-acid names and their correct residue numbers obtained from the previous step.

For example:

```text
GLU,1221
ASN,1222
...
```

Keep the **amino-acid name and residue number** as the search identifiers for the following analysis.

Then, based on the literature, annotate each amino acid with its corresponding secondary-structure or structural-unit label.

For example:

```text
H1
H2
H1-H2 loop
```

A structural unit such as H1 might contain:

```text
H1
├── GLU 1221
└── ASN 1222
```

The important point is that **each secondary structure or structural segment is treated as its own group**.

---

## The Three-Level Search Strategy

The analysis is performed in three levels:

```text
Level 1
Structural Unit
        ↓
Level 2
Residue
        ↓
Level 3
Atom
```

The purpose is to progressively reduce the search space.

---

### Level 1 — Structural Unit Interaction Network

The first layer treats each secondary structure or structural segment as a single block.

For example:

```text
H1
H2
H1-H2 loop
H3
H4
```

Calculate the total interaction difference between structural units in WT and MT.

Conceptually:

```text
        H1
       /  \
      /    \
    H2 ─── H3
      \
       \
       Loop
```

For each pair of structural units, calculate the accumulated interaction difference between WT and MT.

The goal at this stage is **not** to identify the exact residue.

The goal is to answer:

> **Which structural regions appear to be affected by the mutation?**

This dramatically reduces the search space.

---

### Level 2 — Drill Down to Residues

Once the affected structural units have been identified, move to the residue level.

For each structural unit, divide its amino acids into groups of three.

Then calculate interactions between:

> **Three amino acids from one structural unit**

and

> **Three amino acids from another structural unit**

For example:

```text
H1
├── Residue A
├── Residue B
└── Residue C

        ↕ interaction

H2
├── Residue X
├── Residue Y
└── Residue Z
```

The three-residue groups remain defined by their original structural-unit grouping.

This is important because otherwise interactions between neighboring residues may also be included and obscure the signal you are looking for.

The goal of this second layer is to narrow the affected region down to a manageable set of **candidate residue pairs**.

You may obtain several candidate pairs, but do not let the candidate list grow indefinitely.

Every candidate eventually becomes additional manual analysis.

---

### Level 3 — Drill Down to Atoms

Only after the candidate amino-acid pairs have been identified do we move to the atom level.

For example:

```text
GLU 1221
        ↕
ASN 2333
```

There may be multiple candidate residue pairs.

Rather than manually constructing CPPTRAJ input files for every pair, create a small **CPPTRAJ input generator**.

The generator takes the candidate amino-acid pairs and generates the corresponding CPPTRAJ distance calculations.

For example:

```text
GLU 1221
ASN 2333
```

The program determines the appropriate terminal atoms involved in potential interactions for the relevant amino-acid types and converts the residue pair into CPPTRAJ distance commands.

The generated input file can then be placed into the corresponding AMBER WT and MT simulation directories.

CPPTRAJ will calculate the distances for all frames.

---

## Generate WT/MT Distance Comparison Plots

Once the distance calculations are complete, the resulting distance files can be plotted using **gnuplot**.

For example, if the output contains:

```text
Frame    Distance1    Distance2
1        5.21         8.34
2        5.34         8.17
3        5.18         8.02
...
```

you can generate separate WT/MT comparison plots:

```text
WT vs MT — Distance 1
```

and:

```text
WT vs MT — Distance 2
```

The purpose is to visually inspect whether the candidate interaction shows a meaningful difference between WT and MT.

---

## The Complete Workflow

The entire process can therefore be summarized as:

```text
                    MD-TASK
                       │
                       ▼
             Dynamic interaction network
                       │
                       ▼
          WT / MT interaction percentage
                       │
                       ▼
          ┌─────────────────────────┐
          │ Level 1                 │
          │ Structural-unit network │
          └─────────────────────────┘
                       │
                       ▼
             Identify affected regions
                       │
                       ▼
          ┌─────────────────────────┐
          │ Level 2                 │
          │ Residue-level analysis  │
          └─────────────────────────┘
                       │
                       ▼
             Identify candidate pairs
                       │
                       ▼
          ┌─────────────────────────┐
          │ Level 3                 │
          │ Atom-level analysis     │
          └─────────────────────────┘
                       │
                       ▼
                CPPTRAJ distances
                       │
                       ▼
                WT / MT plots
                       │
                       ▼
             Human visual inspection
                       │
                       ▼
             Identify key residues
```

The important idea is that **the computer does not replace the entire mechanistic analysis**.

Instead, it progressively reduces the search space.

You start with potentially hundreds of amino acids and a very large number of possible interactions.

Rather than manually inspecting everything:

> **Structural unit → residue → atom**

At the final stage, only a small number of candidate interactions need to be manually reviewed.

If the WT/MT distance plots show a clear difference, the corresponding amino-acid pair becomes a strong candidate for further investigation.

This gives the original pathway analysis an additional intermediate layer:

> **Literature + visual pathway analysis**
> ↓
> **Structural-unit interaction network**
> ↓
> **Residue-level localization**
> ↓
> **Atom-level distance validation**

That is the main idea behind this Python/MD-TASK extension: **use computation to narrow down where to look, then use human interpretation to determine what the change means.**
