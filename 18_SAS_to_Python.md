# Data Sanitization and Quality Audit Methods for Medical Data After Receipt

## Overview

The main difficulty in implementing **Data Sanitization** and **Data Quality Audit** methods is usually not the Python code itself, but:

> **How to fully translate the analytical vision in a SAS user's mind into rules that can be implemented programmatically.**

Researchers who use commercial software such as SAS are often accustomed to the commercial modules and established analytical workflows provided by SAS. Many processing rules that naturally exist within the SAS environment may not be fully articulated when requirements are described verbally.

Therefore, when a user translates a SAS workflow into Chinese requirements, substantial information loss may occur.

For example, a user may be able to clearly describe the desired data quality checks, such as:

* Number of records in each data file
* One record per patient vs. multiple records per patient
* Patient data completeness across different data tables
* Gender distribution and missing values
* Age calculation and outlier checks
* Statistics such as mean, min, max, P25, median, and P75

However, when the requirements involve an existing SAS analysis workflow, **verbal descriptions may still fail to fully replace the operational details contained in the original program**.

A SAS program may additionally contain:

* Data filtering conditions
* Field-level decision rules
* Missing-value handling
* Classification and coding rules
* Processing logic implemented by specific analysis modules
* Outlier detection rules
* Output data structures and field requirements

Therefore, when the goal is to convert an existing SAS workflow into Python, it is recommended to use the following together as the basis for reconstructing the requirements:

**Explicitly stated research objectives + SAS target program + user-provided supplementary explanations**

This can reduce information loss during requirement translation.

In addition, if the SAS program itself can be provided to AI for program analysis, AI can first help explain:

> **“What does this SAS program actually do?”**

In some cases, this program-level analysis may provide more detail than relying solely on the user's verbal description.

However, AI-generated program explanations should still be treated as supporting information for requirement analysis. The actual research objectives, data definitions, and user intent should ultimately be confirmed by the researchers.

The overall process can therefore be understood as:

```text id="r7j5xz"
SAS Target Program
        +
User Supplement
        ↓
Requirement Reconstruction
        ↓
Python Implementation
        ↓
Data Sanitization
        ↓
Quality Audit
```

---

Medical research data typically needs to undergo **Data Sanitization** and **Data Quality Audit** before being formally used for analysis.

The purpose of this stage is not to perform statistical analysis directly, but to gradually transform raw data from different sources and formats, which may contain various data quality issues, into datasets that are:

**format-consistent, traceable when errors occur, patient-identifiable, assessable for completeness, and suitable for subsequent analysis.**

This method divides the post-receipt data processing workflow into six stages:

```text id="yn6q7w"
Data Received
      ↓
Step 1  Format Conversion
      ↓
Step 2  Data Sanitization
      ↓
Step 3  Patient ID Extraction
      ↓
Step 4  Data Completeness Audit
      ↓
Step 5  Study_ID Mapping
      ↓
Step 6  Automated Audit Report
```

---

# Step 1 — Format Conversion

## Purpose

Convert the original data provided by the user into a standard format required for subsequent data processing.

The actual **Input / Output formats do not need to be fixed**.

For example:

```text id="k8l2dv"
Excel → TXT

Excel → CSV

CSV → Database

Database → DataFrame
```

can all be used depending on the research environment and user requirements.

**For the data-processing program, the input and output formats themselves are not the core methodological constraint.**

What must be preserved is:

> **Physical Structure Validation**

After format conversion, it is necessary to verify that:

> **The conversion itself did not cause data loss.**

At minimum, structural comparisons should be performed before and after conversion, including:

* Row count
* Column count
* Column structure
* Overall data volume
* Presence of required fields

The purpose is to verify:

```text id="f0eq0m"
Original Data
      ↓
Format Conversion
      ↓
Converted Data
      ↓
Physical Structure Validation
```

Only after confirming that the conversion caused no structural loss should the process proceed to the next stage.

---

# Step 2 — Data Sanitization

## Purpose

Handle formatting errors, inconsistent coding, and inconsistent data representations discovered after data receipt.

This stage can be understood as:

> **“All data problems that can be corrected through explicit rules are handled here.”**

For example:

## Year Normalization

Different sources may use different year representations:

```text id="1q5br5"
ROC Year 114
```

converted to:

```text id="g1qk7u"
2025
```

---

## Gender Normalization

The same category may have different representations:

```text id="7u3qpi"
1
01
1.0
```

which are normalized to:

```text id="5w7s7n"
01
```

---

## Audit Trail

Data corrections should not simply overwrite the original values.

Instead, preserve:

```text id="ohwq4u"
Original Value
      ↓
Correction Rule
      ↓
Corrected Value
```

and generate:

* Anomalous data list
* Corrected data list
* Number of corrected records

for subsequent manual review.

Therefore, the core of this stage is not:

> **Change the errors.**

It is:

> **Correct errors according to explicit rules while preserving a record of the corrections.**

---

# Step 3 — Patient ID Extraction

## Purpose

Establish a patient-level index across the entire dataset.

After obtaining patient identifiers from each data table, create a unified Patient ID according to the research data definition.

Then combine the Patient IDs from all data tables and remove duplicates:

```text id="6s9kq4"
Table A
Patient IDs
     │
Table B ─────┐
Patient IDs  │
             ├──→ Union
Table C ─────┤
Patient IDs  │
             ↓
        Deduplication
             ↓
    Unique Patient Index
```

The final output is:

> **A unique patient index for the entire dataset.**

This index becomes the foundation for subsequent data completeness and quality audits.

---

# Step 4 — Data Completeness Audit

Once the Patient Index has been established, construct a:

> **Patient × Data Table completeness matrix**

For example:

```text id="sp7f4f"
             Table A   Table B   Table C

Patient 001      1         1         1
Patient 002      1         0         1
Patient 003      1         0         0
```

where:

```text id="7t4t8c"
1 = Data exists in the table

0 = Data is missing from the table
```

This matrix transforms data presence information that is distributed across multiple tables into patient-level completeness information.

---

## 4.1 Complete Records

First, output:

> **Patients for whom all required data tables are present.**

For example:

```text id="a5q8h4"
Table A = 1
Table B = 1
Table C = 1
...
```

forming:

```text id="f9c2ls"
Complete Patient List
```

---

## 4.2 Incomplete Records

Next, output:

> **Patients who do not meet the completeness criteria.**

These patients should not automatically be treated as “invalid data.”

Different missing-data combinations may still have different levels of research value.

Therefore, patients can be further classified according to their combinations of available data.

---

## 4.3 Duplicate Records

Finally, check:

> **Whether the same Patient ID appears multiple times within the same data table.**

For example:

```text id="1op8o4"
Patient 001
     ↓
Table A
├── Record 1
└── Record 2
```

Such records should be independently flagged to prevent duplicated records from causing incorrect interpretations of record counts or patient counts in subsequent analyses.

---

Therefore, the Step 4 audit results can at minimum be divided into:

```text id="q7d5tq"
Data Audit
     │
     ├──────────────┬──────────────┐
     ↓              ↓              ↓
Complete        Incomplete      Duplicate
Records          Records         Records
```

---

# Step 5 — Study_ID Mapping

After completing the data quality audit, process the identifiers used in the research dataset.

## Purpose

Convert real-world identifiers in the original data into:

> **Study_ID**

The basic concept is:

```text id="f5bbyh"
Original Identifier
        ↓
Mapping Table
        ↓
Study_ID
```

`Study_ID` should be used as the identifier for subsequent research data processing and analysis.

At the same time, original system identifiers that are not required for subsequent analysis should be removed to reduce unnecessary identifying information in the research dataset.

---

# Step 6 — Automated Audit Report

Finally, integrate the results from the previous stages into a:

> **Final Audit Report**

The report can include at least:

## Data Structure

* Data tables
* Number of columns
* Number of records

## Data Sanitization

* Correction items
* Number of anomalies
* Number of corrections

## Completeness

* Number of unique patients
* Number of complete patients
* Number of incomplete patients

## Duplicate / Conflict

* Duplicate records
* ID conflicts

## Execution Information

* Execution time
* Execution status
* Processing workflow

The final output is therefore not simply:

> **“Data processing completed.”**

Instead, it should show:

> **“What processing was performed, what was corrected, what problems were identified, and what the final state of the data is.”**

---

# Pipeline Automation

The six stages above can be centrally managed by a single Controller.

Users do not need to execute each Python module individually.

They only need to run:

```text id="tx7z6k"
00_Master_Controller.py
```

The Controller sequentially executes:

```text id="sp8hpl"
01 Format Conversion
        ↓
02 Data Sanitization
        ↓
03 Patient ID Extraction
        ↓
04 Data Completeness Audit
        ↓
05 Study_ID Mapping
        ↓
06 Automated Audit Report
```

The execution status and processing time for each stage are recorded in a centralized Execution Log.

The current implementation uses a Master Controller to execute the six modules sequentially and stops the downstream workflow if any stage fails.

---

# Methodological Summary

The entire method can be summarized as:

```text id="0hrv4a"
┌──────────────────────────┐
│      Medical Data        │
│        Received          │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ 1. Format Conversion     │
│    + Physical Validation │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ 2. Data Sanitization     │
│    + Error Audit Trail   │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ 3. Patient ID Extraction │
│    + Unique Index        │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ 4. Completeness Audit    │
│    Complete / Incomplete │
│    / Duplicate           │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ 5. Study_ID Mapping      │
│    + De-identification   │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ 6. Audit Report          │
│    + Execution Log       │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│    Analysis-ready Data   │
└──────────────────────────┘
```

# Core Principle

The core of Data Sanitization and Data Quality Audit is not simply writing a data-processing program. It is first **reconstructing the researchers' expectations and decision rules for the data**.

First verify that no data was lost during format conversion. Then correct definable data errors according to explicit rules. After establishing a patient index, perform completeness and duplication audits. Only then should research identifiers be transformed and the final audit report generated.

When an existing analytical workflow originates from a commercial analytical environment such as SAS, the original SAS program should be obtained whenever possible and used as a basis for requirement analysis, together with supplementary explanations from the user, to reduce information loss caused by natural-language translation.

On this basis, the role of Python is to:

> **Transform clarified research logic into a reproducible, traceable, and auditable data-processing workflow.**
