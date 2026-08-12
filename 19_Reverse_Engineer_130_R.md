## How to Reverse-Engineer 130 R Scripts: Reverse Engineering a Legacy Analysis Workflow

### Overview

This was an extension of an existing research project. The PI was not the original author of the analysis workflow. The previous researcher left approximately 130 R scripts with incomplete documentation, along with a master's thesis.

Rather than trying to reconstruct the original author's implementation from the PI's vague vision of the previous study, **it is easier and more time-efficient to reverse-engineer the existing 130 R scripts directly**.

The reason is simple:

> **The existing scripts are concrete implementation evidence left behind by the original analysis workflow.**

Therefore, this method takes the following approach:

> **Build the data-flow map first, then begin reconstructing the analysis workflow.**

The 130 scripts are converted into 130 structured Nodes. Their Input / Output relationships are then used to construct a data-flow network, which can subsequently be analyzed with AI for system-level reverse engineering.

---

# 1. Treat the Scripts as a Data-Flow Network (NODE & EDGE)

Data analysis workflows usually have a clear direction:

```text
Input
  ↓
Processing
  ↓
Output
```

Therefore, the entire analysis system can be represented as:

```text
Program A
   ↓
Output File
   ↓
Program B
   ↓
Output File
   ↓
Program C
```

Under this representation:

* Each script can be treated as a **Node**
* The data-flow relationships established through Input / Output between scripts can be treated as **Edges**

## Basic Node Information

Each program can be described using at least:

* Program Name
* Input Files
* Output Files
* Logic Summary

The key idea is:

> **The program name and summary describe what the Node is; Input / Output describe how the Node connects to other Nodes.**

This allows code that would otherwise require line-by-line reading to first be transformed into a higher-level representation of the data flow.

---

# 2. Compress 130 Scripts into an Analyzable Structure

If approximately 130 R scripts are read manually one by one, the amount of information becomes substantial.

Therefore, the first stage is not to immediately understand the entire research project. Instead, compress:

```text
130 Scripts
```

into:

```text
130 Structured Nodes
```

Each Node retains only information relevant to reconstructing the system:

* Name
* Source Tables
* Fields Used
* Logic Summary
* Output
* Type
* Redundancy
* Core / Noise

The most important component is **Logic Summary**.

The goal is not to explain the code line by line, but to transform:

> **Code → Data Transformation Rule**

For example, instead of describing:

> Line 1 imports a package, line 5 creates a dataframe, line 12 performs a merge...

describe:

> Extract patient data from a specific data table, construct a cohort according to defined conditions, calculate survival time, and generate a dataset for Kaplan–Meier analysis.

This greatly reduces the amount of information while preserving the structure required to reconstruct the system.

---

# 3. The Role of AI: Code Information Compression Rather Than Directly Guessing Research Intent

In this workflow, AI is not primarily responsible for:

> **“Guessing what the PI wanted to do at the time.”**

A more appropriate approach is:

> **First compress the code into structured rules, then use those rules for system-level analysis.**

Therefore, the first task for AI is not to generate research conclusions, but to create a:

> **System Reconstruction Intermediate Representation**

This is a structured representation between the original source code and the complete system.

The core prompt used is:

```text
You are a Data Pipeline Reverse Engineering Analyst.

Your task is not to explain code-level details, but to
“compress the code into structured rules.”

For the code I provide, output the following:

[0] Name

Write the program filename.

[1] Source Tables

List the data tables used (such as *.csv).

[2] Fields Used

List the fields actually used by the code.
Do not list the entire schema; only list fields referenced
by the code.

[3] Logic Summary (Most Important)

Describe what this program does in 2–4 sentences.

Avoid line-by-line explanations. Focus on
“how the data is transformed.”

[4] Output (if determinable)

Describe the output data or fields.

[5] Type (Classification)

Choose one:

- Extraction
- Transformation
- Analysis
- Presentation

[6] Redundancy Check (Optional)

Determine whether this logic may overlap with other scripts.

[7] Core or Noise (Optional)

Determine whether this is:

- Core Logic
- Redundant
- Noise

[Rules]

- Do not explain the code line by line.
- Do not rewrite the code.
- Do not discuss syntax.
- Focus on data flow and rules.
- Use the fewest words with the highest information density.

[Goal]

Compress a piece of messy code into a
“rule node that can be used to reconstruct the system.”
```

---

# 4. From Script Nodes to Batch / Node Groups

Once all 130 scripts have been structurally summarized, scripts with common data sources or common analytical purposes can be grouped together.

Thus:

```text
130 Script Nodes
        ↓
7 Batch / Node Groups
```

At this point, the analysis question changes from:

> **“What does each of the 130 scripts do?”**

to:

> **“What role does each of these 7 program groups play in the overall research system?”**

This substantially reduces the complexity of the reverse-engineering problem.

---

# 5. Use File Names and Folder Structure to Recover Missing Information

During reverse engineering, some original data files may be missing.

In such cases, information can still be recovered from other structural clues:

* Program Name
* File Name
* Folder Name
* Input / Output relationships
* File names referenced within the scripts
* Which scripts contain references to the same file name

Programmers generally retain some degree of semantic and organizational structure when naming programs, data files, and folders. Otherwise, maintaining the system itself would become difficult.

For example, search for where a particular data file name appears across the scripts, then examine the Input / Output relationships of those scripts to gradually reconstruct its likely position in the data flow.

The purpose is not to claim that missing data can be completely recovered.

Rather:

> **When the original materials are incomplete, maximize the use of the structural information that still exists.**

---

At approximately this point, an initial system model can usually be established. Depending on the complexity of the programs, the following stages may not be necessary and are therefore described conceptually rather than with detailed prompts.

---

# 6. Identify Rule Conflicts Across Node Groups

Once the 130 scripts have been compressed into several batches, the next question is no longer:

> **“What does each script do?”**

Instead:

> **“Do different programs use different definitions for the same research concept?”**

The purpose of this stage is not to immediately correct the problem.

It is to:

> **Expose all versions of a rule that were previously hidden across different scripts.**

---

# 7. Conflict Detection: Identify Conflicting Batches

In addition to comparing individual concepts, cross-comparison can also be performed at the batch level.

The goal is to determine:

> **Which batches can reasonably be treated as the same workflow, and which batches may represent different analytical definitions?**

This stage is responsible only for exposing conflicts.

It does not directly determine which definition is correct.

---

# 8. Rule Alignment and the Global System

After Conflict Detection is complete, the process moves to system-level reconstruction.

If the core definitions have been aligned, multiple batches can be compressed again into:

> **Global System**

At this point, the goal is no longer to describe individual scripts.

Instead, construct the core modules of the overall analysis workflow.

---

# 9. Overall Reverse-Engineering Architecture

The core of this method is therefore not simply using AI to read a large amount of code.

It is:

> **Reducing problem complexity layer by layer.**

```text
PI's Vague Vision
        │
        │ Do not guess blindly
        ↓
Existing 130 R Scripts
        ↓
Script → Structured Node
        ↓
Node + Input/Output
        ↓
Data-flow Network
        ↓
Batch / Node Groups
        ↓
Initial System Model
        │
        ├── Sufficient → End
        │
        └── Conflicts Found
                  ↓
            Rule Conflict
                  ↓
            Rule Alignment
                  ↓
            Global System
```

---

# Core Principle

> **Build the map first, then begin reverse engineering.**

When dealing with a large collection of poorly documented legacy analysis scripts, starting from the PI's vague vision and attempting to guess the original implementation can consume substantial time reconstructing context that has already been lost.

By contrast, the existing 130 R scripts are concrete implementation evidence left by the original analysis workflow.

By first using:

* Input / Output relationships
* Program names
* File names
* Folder structures
* Program summaries

to construct a data-flow network, and then progressively compressing the network into batches, concepts, and a global system, the original problem of dealing with 130 scripts can be transformed into a much smaller and more structured analytical problem.

The primary value of AI in this workflow is not to replace researcher judgment, but to:

> **Compress large amounts of code-level information into a system-level representation that can be jointly analyzed by humans and AI.**

Ultimately, the problem is transformed from:

> **“The PI says what they want; I have to guess how the previous researcher implemented it.”**

into:

> **“The previous researcher's scripts are still here. First reconstruct the data-flow map they left behind, then rebuild the system layer by layer from the data flows, rules, and analytical modules.”**
