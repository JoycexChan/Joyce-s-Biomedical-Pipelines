# ETAP Pipeline Implementation — Kaplan–Meier Survival Analysis by Cancer Type × Stage × COVID-19 Definition

## Overview

**Note1: This article includes descriptions of medical data structures and data extraction methods. During the reverse engineering of historical programs, an issue was identified in which cohort files were not consistently fixed, which could affect analysis results.**

**Note2: Do not rely on your own memory when defining the study population. Fix the population-level IDs and save them as a CSV file; every analysis should start from this cohort file.**

**Fix the cohort first → prepare the phenotype → integrate the data → perform the analysis.**

This case demonstrates how to use an **ETAP Pipeline** to connect a cancer patient cohort, multi-source COVID-19 data, and Kaplan–Meier Survival Analysis into a reproducible analytical workflow.

The overall workflow can be simplified as:

```text
Prepare Cancer Patient Cohort
            ↓
Prepare COVID Cohort
            ↓
Cancer Cohort
     +
COVID Cohort
            ↓
Transformation
            ↓
KM Analysis
```

The core concept of this workflow is:

> **First fix the patient cohort, then prepare the COVID phenotype, and finally map the COVID information back to the cancer patient cohort during the Transformation stage to create the analysis dataset required for KM analysis.**

---

# 1. Data Processing Principles

Real-world medical data are typically composed of many different data tables.

The fields and structures of each table are generally documented in the corresponding data specifications. If the data use `CONCEPT_ID`, the appropriate Concept Dictionary should be consulted for mapping.

In actual analysis, it is not necessary to repeatedly operate directly on all original data tables.

The core principle is:

> **First fix the cohort IDs, then use those IDs to extract the required data from each table and create mini datasets.**

For example:

```text
Raw Medical Data

├── A Table
├── B Table
├── C Table
└── D Table
```

If the current study requires:

```text
A Table + predefined patient cohort
```

the recommended approach is:

```text
Fixed Cohort IDs
       ↓
A Table
       ↓
01_A_mini.csv
```

rather than re-filtering the patient population from the raw data independently for every analysis.

---

## Why Fix the Cohort First?

Medical datasets are usually very large, and different stages of analysis may use different data tables.

If every program independently re-selects the patient population, the following situation can easily occur:

```text
Script A
→ filters the cohort

Script B
→ filters the cohort again

Script C
→ filters the cohort again
```

Eventually, different analyses may use different patient populations.

Therefore, this workflow adopts the following principle:

> **Once the cohort IDs are fixed, subsequent data extraction should use this cohort ID CSV as the reference whenever possible.**

In simple terms:

> **Do not trust every script to correctly redefine the cohort. Fix the cohort first, then replicate the rule.**

This is also an important mechanism for reducing manual errors in the ETAP Pipeline.

---

# 2. Required Materials

Before starting KM Analysis, two types of data need to be prepared:

1. **Cancer patient cohort**
2. **COVID cohort**

These two components are integrated during the subsequent Transformation stage.

---

## 2.1 Fix the Patient Cohort First

The first step in KM Analysis is to establish the analysis cohort.

The patient cohort in this case is defined as:

* Exclude unknown cancer stages, such as `999 / BBB`
* Exclude patients with records indicating a second cancer diagnosis
* Temporarily retain all other stages, including Stage 0

The purpose of this step is to determine:

> **Which patients are eligible to enter the subsequent cancer analysis.**

---

## 2.2 Organize Data by Cancer Type

For example:

```text
Cohort/

├── ColorectalCancer/

├── LiverCancer/

├── LungCancer/

└── threeCancer/
```

This allows each cancer type to maintain its own cohort data.

---

## 2.3 `01_T02_mini_ALL.csv`

This file represents:

> **All patients within the target cancer category.**

It is a mini dataset extracted from the original T02 data using the required fields.

The meaning of `ALL` is:

> **Data within the target cancer scope; it does not necessarily represent the final analysis cohort.**

---

## 2.4 `01_T02_mini_cohort.csv`

This file represents:

> **The cohort that remains after applying the predefined patient cohort rules and is used for subsequent analysis.**

Conceptually:

```text
01_T02_mini_ALL.csv
        ↓
Cohort Rules
        │
        ├─ Exclude unknown cancer stages
        └─ Exclude patients with a second cancer diagnosis
        ↓
01_T02_mini_cohort.csv
```

This `mini_cohort` file becomes an important entry point for subsequent data extraction.

---

# 3. Prepare the COVID Cohort

After the cancer patient cohort has been established, the COVID cohort is prepared separately.

COVID information does not come from a single data table. Instead, it is derived from multiple medical data sources.

In this case, the detailed configuration is omitted because it represents study-specific settings rather than the general program structure.

The sources include:

```text
Measurement = 01_Extraction_T07_four.py
Condition   = 01_Extraction_T05_four.py
Drug        = 01_Extraction_T06_four.py
Discharge   = 01_Extraction_T10_four.py
history_cov = 01_Extraction_T08_four_788.py
```

These programs extract COVID-related patient information from different data sources.

Conceptually:

```text
T05 Condition
      │
T06 Drug
      │
T07 Measurement
      │
T08 Procedure
      │
T10 Discharge
      │
      ▼
COVID-related patient sets
```

These sources can then be analyzed using:

```text
03_Analysis_UpSet.py
```

to perform patient-level set intersection analysis and generate an UpSet plot.

The primary purpose of the UpSet analysis is:

> **To examine which patients overlap across the different COVID data sources.**

---

# 3.1 UpSet Execution Environment

This component depends on the third-party open-source package `upsetplot`.

Due to historical package compatibility issues, this case uses an independent Conda environment:

```text
covid_310_universe
```

with Python 3.10.

The main package versions used are:

```text
python == 3.10.*
numpy == 1.26.4
pandas == 2.2.3
upsetplot == 0.9.0
matplotlib
```

Before running the program in VS Code, switch the interpreter to:

```text
covid_310_universe
```

This environment configuration represents the historical execution environment used in this case. Its primary purpose is to ensure that the existing UpSet program can run correctly.

---

# 3.2 The COVID Cohort Is Not a Single Data Table

This point is important.

It is not simply:

```text
One COVID table
       ↓
COVID = 1
```

Instead:

```text
T05 ─┐
T06 ─┤
T07 ─┼──→ COVID-related patient sets
T08 ─┤
T10 ─┘
```

These sources are then organized into a COVID cohort that can be used for subsequent analysis.

Therefore, in this case, COVID represents a:

> **Multi-source phenotype**

---

# 4. Kaplan–Meier Analysis

After the following have been prepared:

* Cancer Cohort
* COVID Cohort

the KM Pipeline can begin.

The workflow is divided into:

1. **Extraction**
2. **Transformation**
3. **Analysis**

---

## 4.1 ① Extraction — Build T01/T02 Data for the Cancer Cohort

Program:

```text
01_Extraction_T01T02_four.py
```

This program extracts the T01/T02 data required for KM analysis according to the already established Cancer Cohort.

Conceptually:

```text
Cancer Cohort
      │
      ▼
T01 / T02
      │
      ▼
01_T01_mini_cohort.csv
01_T02_mini_cohort.csv
```

This layer primarily:

> **Maps the cancer patients already determined to be part of the analysis to the T01/T02 data required for subsequent analysis.**

This step does not redefine COVID and does not perform KM statistics.

---

# 4.2 ② Transformation — Map COVID Back to the Cancer Cohort

This is the most important data integration step in the workflow.

`02_Transformation.py` first reads:

```text
01_T01_mini_cohort.csv
01_T02_mini_cohort.csv
```

That is:

> **The Cancer Cohort data already established during Extraction.**

It then reads the previously established:

```text
Cohort/COVID/
```

which contains:

```text
T05
T06
T07
T08
T10
```

Patient-level matching is then performed using `PERSON_ID`.

---

## COVID Mapping

```text
Cancer Cohort
      │
      │ PERSON_ID
      ▼
┌────────────────────┐
│ COVID Cohort       │
│                    │
│ T05                │
│ T06                │
│ T07                │
│ T08                │
│ T10                │
└─────────┬──────────┘
          │
          ▼
     Patient Match
          │
          ▼
       covid_n
        1 / 0
```

The program first organizes patients appearing in the COVID sources into `patient_sources`, and then compares each patient in the Cancer Cohort against those sources.

The resulting fields include:

```text
covid_n
COVID_source
```

where:

* `covid_n = 1`: the patient is present in the COVID phenotype
* `covid_n = 0`: the patient is not present in the COVID phenotype

This step can therefore be summarized as:

> **Take the Cancer Cohort produced by ① Extraction and mark whether each patient has the predefined COVID phenotype.**

---

# 4.3 Transformation — Create KM Variables

In addition to COVID mapping, the Transformation stage creates the analytical variables required for KM analysis.

## Stage

The original fields:

```text
CR_0307
CR_0313
```

are transformed into:

```text
Stage_fine
Stage_4
Stage_5
```

according to the predefined Stage rules.

---

## Survival Variables

Transformation also creates:

```text
diagnosis_date
death_date
last_date
followday
followmonth
event
death
censor
```

These variables become inputs for the subsequent survival analysis.

---

# 4.4 Final Transformation Output

The Transformation stage therefore combines:

```text
Cancer Cohort
       +
COVID Cohort
       +
Stage
       +
Survival Variables
       ↓
02_transformed_dataset.csv
```

This file becomes the entry point for the Analysis stage.

---

# 4.5 ③ Analysis — Kaplan–Meier Analysis

After Transformation is complete, Analysis does not need to return to the raw data or independently determine COVID status.

It directly uses:

```text
02_transformed_dataset.csv
```

which already contains:

* `Stage_5`
* `covid_n`
* `followmonth`
* `event`

Analysis groups can therefore be directly constructed, for example:

```text
Stage I + non-COVID
Stage I + COVID

Stage IV + non-COVID
Stage IV + COVID
```

and then used for:

**Kaplan–Meier Survival Analysis**

---

# 5. Complete Workflow

The entire process can be connected as follows:

```text
                         Raw Medical Data
                                │
                                ▼
                       ┌─────────────────┐
                       │ 1. Fix Cohort   │
                       │ Cancer Cohort   │
                       └────────┬────────┘
                                │
                                ▼
                       01_T02_mini_cohort
                                │
                                │
                 ┌──────────────┘
                 │
                 ▼
          ┌───────────────────┐
          │ 2. COVID Cohort   │
          │                   │
          │ T05 / T06 / T07   │
          │ T08 / T10         │
          └─────────┬─────────┘
                    │
                    ▼
              COVID Cohort
                    │
                    │
                    └──────────────┐
                                   ▼
                         ┌──────────────────┐
                         │ 3-① Extraction   │
                         │ T01 / T02        │
                         └────────┬─────────┘
                                  │
                                  ▼
                         Cancer mini dataset
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ 3-② Transformation│
                         │                  │
                         │ Stage            │
                         │ COVID_n          │
                         │ Survival         │
                         └────────┬─────────┘
                                  │
                                  ▼
                           Analysis Dataset
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ 3-③ KM Analysis  │
                         └────────┬─────────┘
                                  │
                                  ▼
                           KM Survival Curve
```

---

# 6. The Most Important Data-Flow Concept

The easiest part of this case to misunderstand is assuming:

> **The KM program searches the raw medical data for COVID information by itself.**

It does not.

The more accurate data flow is:

```text
① Fix the Cancer Cohort
        ↓
② Separately establish the COVID Cohort
        ↓
③ Extraction maps the Cancer Cohort to T01/T02
        ↓
④ Transformation uses PERSON_ID
   to map the COVID Cohort back to the Cancer Cohort
        ↓
⑤ Generate covid_n
        ↓
⑥ KM uses covid_n for grouping
```

Therefore:

> **Extraction determines which cancer patients enter the analysis dataset.**

> **Transformation determines the Stage, COVID, and survival variables for those patients.**

> **Analysis uses the already-defined variables to perform KM analysis.**

---

# Core Principle

The entire workflow can be condensed into one sentence:

> **Fix the cohort first, then extract data from the fixed cohort; independently establish different phenotypes, and finally map them back to the fixed cohort during Transformation before performing the analysis.**

This means that even if the following are changed later:

* Cancer type
* Stage definition
* COVID phenotype
* Follow-up period
* KM grouping method

the entire raw medical dataset does not need to be reprocessed from scratch.

The core workflow remains:

> **Fix Cohort → Build Mini Dataset → Build Phenotype → Mapping → Analysis**
