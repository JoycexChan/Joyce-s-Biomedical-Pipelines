A $170k (NT$5.4M) Virtual Screening Pipeline 
(Currently supporting two ongoing NSTC-funded research projects.)
________________________________________
🛠 The “Beginner Version” Pipeline Logic
(The Core Architecture Behind a $170k USD / NT$5.4M Workflow)
The main goal of this pipeline is actually very simple:
To screen large compound libraries without manually clicking through a GUI thousands of times.
________________________________________
🧪 The Two Major Camps in Virtual Screening
Virtual screening roughly splits into two worlds:
Field	Characteristics
Computational Chemistry	High precision, low efficiency
Bioinformatics / Computational Biology	High efficiency, lower precision
My own background is mainly in molecular dynamics simulations, especially protein systems.
As for chemistry…
Honestly, it has degraded into “common sense level knowledge” over the years.
For example:
Yes, I did take organic chemistry in college — but after more than a decade, who still remembers all those reaction mechanisms? (laughs)
So this entire batch virtual screening workflow was essentially built from a:
Bioinformatics-driven perspective.
________________________________________

But honestly:
Anyone who can code should be able to reconstruct it.
And nowadays:
You could probably just let AI generate most of the code for you.
Back then, I actually built large parts of this workflow together with GPT.
Once you understand:
•	file formats
•	workflow order
•	how data flows between stages
…it really is that simple.
________________________________________
🐇 Overall Workflow Overview (Rabbit Way)
________________________________________
Step 1｜Protein Preparation: From PDB to a Computable Model
Search for Discovery Studio tutorials:
Prepare Protein
________________________________________
Protein Preparation Workflow
1. Import the Structure
Use:
•	File > Open
•	or Open URL
Then input a PDB ID (for example: 2IRZ).
________________________________________
2. Cleanup (Important)
You must:
•	remove water molecules
•	remove heteroatoms
•	fix missing side chains
________________________________________
3. Prepare Protein
Use Discovery Studio’s:
Prepare Protein
Default settings are usually acceptable:
•	pH = 7.4
(adjust according to experimental conditions)
________________________________________
4. File Conversion
AutoDock Vina ultimately requires:
.pdbqt
So the .pdb structures need batch conversion.
This can be done using:
•	ADFRsuite
•	prepare_receptor
running in the Linux background.
________________________________________
Step 2｜Compound Databases
The following three databases were the primary compound sources used in this workflow.
________________________________________
A. ZINC20 / ZINC22 (Recommended)
Advantages
•	The largest virtual screening database in the world
•	Many molecules already include:
o	3D conformations
o	partial charge processing
________________________________________
B. PubChem
Advantages
•	Most authoritative
•	Most up-to-date
________________________________________
C. DrugBank (Higher Barrier)
Situation
•	Academic access requires registration
•	API access is paid
However:
Its metadata is extremely rich.
Including:
•	drug targets
•	mechanisms
•	pathways
•	pharmacology information
________________________________________
Step 3｜Ligand Cleaning and Standardization
Search for Discovery Studio tutorials:
Prepare Ligands
________________________________________
Energy Minimization
Tool location:
Simulation → Minimization → Minimize Ligands
________________________________________
Downloaded SDF files often contain:
•	salts
•	solvent molecules
•	incorrect stereochemistry
Honestly:
Just dump everything into DS and process it.
Discovery Studio is commercial software after all.
A lot of things are easier than expected.
________________________________________
Standard Workflow
1. Fragment Cleanup
Keep only the largest fragment:
•	remove salts
•	remove solvents
________________________________________
2. Prepare Ligands
Functions include:
•	3D structure generation
•	stereoisomer enumeration
•	charge assignment
•	energy minimization
Important note:
I only used the default settings here.
I did not perform advanced protonation or charge-state optimization.
If you care about ionization states:
Handle it at this stage.
________________________________________
3. Export as .mol2
After processing:
Save as .mol2
________________________________________
Step 4｜Splitting and Conversion
Next, write a Python script to split the large .mol2 file into:
One molecule per file.
________________________________________
Naming Strategy
Directly extract the line following:
@<TRIPOS>MOLECULE
and use it as the filename.
________________________________________
This step is extremely important because:
It solves traceability for tens of thousands of molecules.
Later, every docking result:
•	filename
•	score
•	structure
can be directly mapped back to the original compound.
________________________________________
Encoding Recommendation
Use:
ISO-8859-1
to avoid crashes caused by special characters.
________________________________________
Step 5｜Convert to pdbqt
AutoDock ultimately requires:
.pdbqt
So the .mol2 files must be converted again.
Use:
prepare_ligand4.py
from MGLToolsPckgs.
________________________________________
Parameters I Used
-U none
Purpose:
Preserve hydrogen atoms.
________________________________________
Mainly because:
1.	My molecular dynamics background prefers complete structural information
2.	Modern computers are fast enough anyway
Why save those extra few minutes? (laughs)
________________________________________
For roughly ten thousand compounds:
The whole conversion process took about three days.
________________________________________
🔧 Troubleshooting (Important)
Because the dataset size is huge:
Manual checking is impossible.
So write a script that checks whether:
.mol2 → .pdbqt
conversion succeeded.
________________________________________
If conversion fails:
•	list them in unconverted_mol2.txt
•	automatically copy them into another folder
•	reprocess them separately
________________________________________
📁 Why Separate Databases Into Different Folders?
I ran:
•	ZINC
•	DrugBank
•	PubChem
in separate directories.
The reason was simple:
Easier traceability back to the original source database.
Later during analysis:
•	click compound name
•	return to source database
•	inspect full metadata
Very convenient.
________________________________________
Step 6｜Grid Box Definition
Search for:
CDOCKER
________________________________________
If your binding pocket already contains a co-crystallized ligand:
Select the ligand and navigate to:
Receptor-Ligand Interactions
→ Define and Edit Binding Site
________________________________________
Create a sphere.
Then right-click to retrieve:
•	center coordinates
•	radius
This feature is available even in the free functionality.
________________________________________
Also:
You can manipulate the sphere using:
Ctrl + Middle Mouse Drag
which is very intuitive for beginners.
________________________________________
Step 7｜Batch Docking (AutoDock Vina)
Example configuration:
receptor = receptor.pdbqt
ligand = ligand.pdbqt

out = results.pdbqt

center_x = -3.713297
center_y = -57.316790
center_z = 36.899605

size_x = 10
size_y = 10
size_z = 10

exhaustiveness = 16
________________________________________
About exhaustiveness
Default is usually:
8
I used:
16
mostly because:
I didn’t want to rerun the docking repeatedly.
Later I realized:
The runtime increase wasn’t even that dramatic.
Adjust according to your hardware.
________________________________________
Python Batch Docking Logic
The script should:
1.	Create an output folder
2.	Find all .pdbqt ligands
3.	Exclude receptor.pdbqt
4.	Automatically run Vina on each ligand
5.	Generate matching output filenames
________________________________________
Automated Naming
Example:
.replace(".pdbqt", "_out.pdbqt")
This guarantees:
Input and output files always remain traceable.
________________________________________
🦥 Lazy Optimization Trick (Surprisingly Useful)
Binding pockets have size limitations.
So you can:
1.	Find one ligand that barely fits
2.	Check its file size
3.	Sort all ligands by file size
________________________________________
Anything significantly larger:
Skip it entirely.
Because it physically won’t fit into the pocket anyway.
________________________________________
Step 8｜Result Analysis
Remember:
What we actually need is:
•	drug name
•	docking score
•	model number
________________________________________
Script Logic
Parse:
MODEL
REMARK
fields from the output files.
________________________________________
Finally generate a:
CSV report
containing:
•	drug_name
•	score
•	model
________________________________________
Recommended Additional Column
Add:
database_url + drug_name
so you can jump directly back to the original database entry.
I didn’t think of this back then:
I only added it later in Excel. (laughs)
________________________________________
Step 9｜Combine PDB Structures
Using the ranked CSV:
Automatically extract specific models and merge them into:
combine.pdb
________________________________________
At this point:
Your ligands will load together according to ranking.
Adjust the number based on your computer specs.
Too many structures may cause lag.
________________________________________
Step 10｜PyMOL / Discovery Studio Visualization
This is where we enter:
“How to sound like a medicinal chemist.” (laughs)
________________________________________
You can always skim a 3D-QSAR paper and memorize some buzzwords.
But honestly, the core ideas are mostly:
________________________________________
Steric Field Analysis (Steric Map)
Discovery Studio path:
Receptor-Ligand Interactions
→ View Interactions
→ Show 2D Diagram
________________________________________
Main Analysis
Van der Waals Clashes
If atoms are too close:
•	red lines appear
•	steric collisions occur
Meaning:
The region is overcrowded.
________________________________________
Suggested Optimization
Reduce side-chain bulk.
For example:
-CH3 → -H
________________________________________
Empty Pocket Regions
If there is still large empty space in the pocket:
Consider adding:
•	aromatic rings
•	alkyl chains
to enhance:
hydrophobic interactions
________________________________________
Electrostatic Field Analysis
________________________________________
Visualizing Surface Potential
In the Properties panel:
Display Style
→ Surface / Mesh
Color Mode:
Color By Potential
Typically:
Color	Meaning
Blue	Positive charge
Red	Negative charge
White	Neutral
________________________________________
Electrostatic Optimization Logic
Blue Regions (Electron Deficient)
Suggested modifications:
Add electronegative groups such as:
•	-F
•	-OH
•	=O
to form:
•	hydrogen bonds
•	electrostatic attractions
________________________________________
Red Regions (Electron Rich)
Suggested modifications:
Add:
•	-NH3+
•	electron-donating groups
to achieve charge complementarity.
________________________________________
🧠 Final Thoughts
At its core, this entire workflow is not:
“Some magical drug discovery technique.”
Instead, it is:
“Turning GUI labor into a reproducible pipeline.”
________________________________________
The real value lies in:
•	traceability
•	batch automation
•	reproducibility
•	structural organization
________________________________________
In other words:
Transforming virtual screening from
“manual GUI hell”
into
“a scalable computational pipeline.”
