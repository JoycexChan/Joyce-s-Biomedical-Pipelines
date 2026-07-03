A $170k (NT$5.4M) Virtual Screening Pipeline (The Upgraded Pipeline)
________________________________________

My Basic View of LLMs
LLMs (Large Language Models) are AI systems based on deep learning.

My current view is:

If a problem already has a correct solution somewhere in human knowledge, then with sufficiently precise prompting, an LLM can often retrieve or reconstruct that answer from its learned knowledge space.

If a problem does not yet have a known correct answer, but you possess the “crowbar” capable of moving that domain forward, then there is still a chance that large-scale compute clusters can help explore and approximate new solutions.

The functions described below have already appeared in different software ecosystems across related fields, so I consider them examples of “answers that already exist.”

As for electrostatic-charge-related optimization, I have not fully tested that yet. That problem still does not have a definitive correct answer. Just because an AI thinks my direction is reasonable does not mean it is guaranteed to be correct. I only consider it worthy of experimentation.

________________________________________

The Two Major Camps in Drug Screening
Drug screening roughly splits into two worlds:

The chemistry-oriented world
The bioinformatics-oriented world
The chemistry side achieves higher precision, but suffers from low throughput.

The bioinformatics side achieves higher throughput, but sacrifices some precision.

My own background is primarily in molecular dynamics simulations for proteins. My chemistry knowledge has long degraded into “common-sense residue.” For example, yes, I took organic chemistry in sophomore year, but after more than ten years, who really remembers all of it?

So my approach to batch drug screening is fundamentally bioinformatics-oriented.

For anyone who can code, this is not actually difficult. Once you understand the data formats, the entire thing becomes surprisingly straightforward.

You can even ask Discovery Studio vendors for trial access pretending you are interested in purchasing the software (XDD). Their GUI features are often available for testing. Some labs also rent DS environments by time slot. Go play with it.

________________________________________

Overall Workflow Overview (Rabbit Way)
Step 1｜Protein Processing: From PDB to a Computable Model
PDB files are only the starting point.

A raw .pdb downloaded directly is essentially “dirty data.”

Preprocessing Automation (The “Rabbit” Way)
Downloading
Do not manually click downloads.

Use:

Biopython
PDB RESTful APIs
for batch retrieval.

Cleanup
You must:

remove water molecules
remove heteroatoms
repair missing side chains
Protein Preparation (Equivalent to DS pH Adjustment)
Discovery Studio’s “Prepare Protein (pH = 7.4)” can actually be replicated under Linux using:

pdb2pqr30
Format Conversion
Batch-convert .pdb into .pdbqt (AutoDock format).

This can be done using:

prepare_receptor
from the ADFRsuite package.
Run it in the Linux background for bulk processing.

________________________________________

Step 2｜Drug Sources: Where Thousands of  Small Molecules Come From
When updating databases, simply compare existing records and only download new compounds.

If you are extremely lazy, rerunning the entire dataset is honestly not a disaster either.

A. ZINC20 / ZINC-22 (Recommended)
Advantages
The world’s largest screening database
Already provides:
3D structures
partial electrostatic preparation
Access
tranche downloads
Python API (zinc_linker)
B. PubChem
Advantages
Most authoritative
Most up-to-date
API
PUG-REST API
Note
Do not scrape webpages directly.

That is painfully inefficient.

C. DrugBank (Higher Barrier)
Situation
Academic access requires application
API access costs money
Strength
Its metadata is extremely rich, especially:

drug-target relationships

________________________________________

Step 3｜Small-Molecule Cleanup and Standardization
Downloaded SDF files often contain:

salts
solvent fragments
incorrect stereochemistry
Tool
RDKit (Python library)
Logic
Fragment cleanup
keep only the largest fragment
remove salts and solvents
2. Standardization

normalize valence structures
ensure chemically valid bonds

________________________________________

Step 4｜Energy Minimization (Bulk Mode)
This can be performed directly from the command line using:

Open Babel

________________________________________

Step 5｜Format Conversion and Docking Preparation
AutoDock Vina does not accept:

.sdf
.pdb
It only accepts:

.pdbqt
This format additionally contains:

partial charges
rotatable bond information
Small-Molecule Conversion
If Open Babel was already used earlier:

obabel minimized_drug.sdf -O drug.pdbqt
Protein Conversion
If using PDB2PQR, the output will be .pqr.

You then need:

prepare_receptor
from the AutoDock toolkit to convert it into .pdbqt.

________________________________________

Step 6｜Grid Box: The Most Common Failure Point
You must tell Vina where the binding pocket is.

If the original protein already contains a ligand:

compute the ligand centroid using Python
Alternatively:

use Discovery Studio to directly read:
ligand coordinates
sphere size
That feature is free.

________________________________________

Step 7｜Batch Docking Script (AutoDock Vina)
Write a Python script that:

traverses all compounds in a folder
docks them one by one automatically
using AutoDock Vina.

________________________________________

Step 8｜Result Ranking and Preliminary Filtering
Remember:

What we actually need is:

drug name
docking score
model number
Script Logic

Parse:

MODEL

REMARK

fields from the output files.

Finally generate a:

CSV report

containing:

drug_name
score
model
Recommended Additional Column

Add:

database_url + drug_name

so you can jump directly back to the original database entry.

I didn’t think of this back then:

I only added it later in Excel. (laughs)

Combine PDB Structures

Using the ranked CSV:

Automatically extract specific models and merge them into:

combine.pdb

At this point:

Your ligands will load together according to ranking.

Adjust the number based on your computer specs.

Too many structures may cause lag.

________________________________________

Step 9｜PyMOL Visualization and “Information Hand-Off to Synthetic Chemists”
You can always skim a few 3D-QSAR papers, memorize some terminology, and sound highly professional.

But the core logic is roughly this:

________________________________________

Steric Field Analysis (Steric Map / Van der Waals Clashes)
Logic
Observe distances between:

ligand atoms
protein residues

________________________________________

Analysis
Steric Clash
If atomic distance is smaller than the sum of van der Waals radii:

usually shown as red collisions
then the region is overcrowded.

Suggestion
Reduce side-chain volume.

Example:

-CH3 → -H
Empty Space
If the pocket still contains large unoccupied regions:

Suggestion
Add hydrophobic groups such as:

phenyl rings
alkyl chains
to increase hydrophobic interactions.

________________________________________

Electrostatic Field Analysis (Electrostatic / Hydrogen Bond Map)
Logic
Use:

PyMOL Vacuum Electrostatics
APBS plugin
Analysis
Positive Region (Blue)
The protein region lacks electrons.

Suggestion
Add electronegative groups:

-F
-OH
=O
to form:

hydrogen bonds
electrostatic attractions
Negative Region (Red)
The protein region is electron-rich.

Suggestion
Add positively charged or electron-donating groups:

-NH3

________________________________________

Extension｜If the Synthetic Chemist Wants to Modify Functional Groups
At that point, the structure can be handed over to a synthetic chemist.

If they want to modify substituents:

simply build a fragment library
attach fragments to desired ligand positions
The process is roughly:

Specify atom indices
Compute vector-coordinate offsets
Force-insert fragment atoms
Rewrite bond topology tables
Run Open Babel energy minimization
Let the force field relax the structure back into a valid energy surface

________________________________________

Finally:

feed the modified structures back into the batch docking pipeline again.
