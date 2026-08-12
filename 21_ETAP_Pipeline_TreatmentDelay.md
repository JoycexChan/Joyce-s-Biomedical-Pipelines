## COVID-19 Cancer Treatment Delay Analysis Program Documentation

### 1. Overview

This program analyzes **Treatment Delay** among patients with three types of cancer before and after the COVID-19 pandemic, with statistical comparisons performed across different cancer stages and time periods.

The current analysis includes:

* Lung Cancer
* Colorectal Cancer
* Liver Cancer

The overall workflow is divided into:

**Data Extraction → Data Transformation → Statistical Analysis → Results Presentation → Excel Integration**

In addition, the correspondence between Clinical Stage and Pathological Stage is separately organized into a Stage Matrix.

Some analysis specifications are centralized in `config.py`. Therefore, if the analysis grouping or presentation order needs to be modified in the future, the configuration can generally be adjusted first without modifying every analysis module.

The overall workflow is centrally controlled by `00_RUN.py`, which executes the analysis modules in a predefined sequence.

---

# 2. Overall Analysis Architecture

The system uses a modular design.

```text
T02 Cancer Registry
        │
        ▼
01_Extraction_T02_Three.py
        │
        ├──────────────┬──────────────┐
        ▼              ▼              ▼
   Lung Cancer   Colorectal Cancer   Liver Cancer
        │              │              │
        └──────────────┼──────────────┘
                       ▼
              02_Transformation.py
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
    Treatment        Stage       COVID Period
      Delay         Framework     Before/After
          │            │            │
          └────────────┼────────────┘
                       ▼
                03_Analysis.py
                       │
             Descriptive Statistics
                       │
                       ├──────────────┐
                       ▼              ▼
          03_Analysis_pvalue.py   Stage Matrix
                       │              │
                       ▼              ▼
                    F2A / F2B    StageMatrix
                       │              │
                       ▼              ▼
                   Excel Output
```

---

# 3. Analysis Units and Key Research Concepts

## 3.1 Cancer Domain

The system currently processes three cancer types:

| Cancer            | ICD Code      |
| ----------------- | ------------- |
| Lung Cancer       | C33, C34      |
| Colorectal Cancer | C19, C20, C21 |
| Liver Cancer      | C22           |

Cancer classification is managed by `cancer_config` in `01_Extraction_T02_Three.py`.

---

## 3.2 Treatment Delay

Treatment Delay is defined as:

> **Treatment Date − Diagnosis Date**

The program uses:

* `CR_0205`: Diagnosis Date
* `CR_0401`: Treatment Date

to construct the relevant dates and calculate the number of days between them, producing:

```text
delay
```

Records for which a valid Treatment Delay cannot be obtained are excluded from subsequent analysis.

---

## 3.3 Stage Framework

The program retains both a more detailed Stage classification and higher-level Stage groupings.

### Fine Stage

Examples include:

* Stage IA
* Stage IB
* Stage IIA
* Stage IIIA
* Stage IIIB
* Stage IVA
* Stage IVB

and others.

### Stage 4

The detailed stages are consolidated into:

* Stage I
* Stage II
* Stage III
* Stage IV

### Stage 5

Some Stage III subdivisions are retained:

* Stage I
* Stage II
* Stage III/IIIA
* Stage IIIB/IIIC
* Stage III (NOS)
* Stage IV

The above Stage fields are generated for both Clinical Stage and Pathological Stage.

---

# 4. Module 01 — Data Extraction

## `01_Extraction_T02_Three.py`

### Function

This module creates the analysis workspace for the three cancer types from the raw T02 cancer registry data.

Input:

```text
T02_cancer_registry_20250725.csv
```

The program first reads the T02 data and then separates the data according to cancer ICD codes.

---

## 4.1 Cancer Stratification

The current configuration uses:

```text
Lung Cancer
    C33 / C34

Colorectal Cancer
    C19 / C20 / C21

Liver Cancer
    C22
```

A separate directory is created for each cancer type.

---

## 4.2 Output Data

Each cancer directory contains:

```text
01_T02_mini_ALL.csv
01_T02_mini_cohort.csv
```

### `01_T02_mini_ALL.csv`

Contains records matching the ICD codes for the corresponding cancer.

### `01_T02_mini_cohort.csv`

Further filters the above data using:

```text
CR_0202 == "01"
```

and uses the resulting records as the cohort for subsequent analysis.

---

## 4.3 Analysis Workspace

This module also creates the analysis directories for each cancer type, allowing subsequent programs to use the same analysis architecture within each cancer-specific directory.

Therefore, all three cancers use the same analysis modules while their source data have already been separated by cancer type.

---

# 5. Module 02 — Data Transformation

## `02_Transformation.py`

### Function

This module is the primary data transformation stage of the analysis workflow.

Input:

```text
01_T02_mini_cohort.csv
```

Output:

```text
02_transformed_dataset.csv
```

It is primarily responsible for three tasks:

1. Creating Treatment Delay
2. Creating the Stage Framework
3. Creating COVID Before / After time groups

---

## 5.1 Date Transformation

The program converts:

```text
CR_0205 → diagnosis_date
CR_0401 → treatment_date
```

and then calculates:

```text
delay = treatment_date - diagnosis_date
```

The unit is days.

---

## 5.2 Stage Transformation

The program separately processes:

```text
CR_0307 = Clinical Stage
CR_0313 = Pathological Stage
```

Both generate:

```text
_stage_fine
_stage_4
_stage_5
```

A combined Stage is then created.

The current implementation uses:

> **Pathological Stage first; if Pathological Stage cannot be determined, Clinical Stage is used.**

The final fields are:

* `Stage_fine`
* `Stage_4`
* `Stage_5`

---

# 6. COVID Period Definition

The Treatment Delay analysis uses two time frameworks.

## 6.1 Global Period

The program defines:

```text
2020-03-11
```

and classifies patients according to Diagnosis Date as:

* Before
* After

---

## 6.2 Taiwan Period

The program defines:

```text
2022-04-01
```

and similarly classifies patients according to Diagnosis Date as:

* Before
* After

Therefore, the same patient record can simultaneously have:

```text
Global_period
TW_period
```

representing the two different time classifications.

---

# 7. Module 03 — Descriptive Analysis

## `03_Analysis.py`

### Function

This module calculates descriptive statistics for Treatment Delay.

Input:

```text
02_transformed_dataset.csv
```

The analysis is stratified according to:

* Stage framework
* TW / Global period
* Before / After

---

## 7.1 Statistics

Each group is summarized using:

* N
* Mean
* Median
* SD
* IQR

Two output formats are used.

### F2A

```text
Mean (Median)
```

Example:

```text
28.4 (23.0)
```

### F2B

```text
Mean ± SD
Median (IQR)
```

Example:

```text
28.4 ± 26.7
23.0 (12.0–39.0)
```

---

# 8. Module 03 — P-value Analysis

## `03_Analysis_pvalue.py`

This module compares Treatment Delay between the Before and After groups.

Two statistical tests are used.

### Mann–Whitney U Test

Used to compare the distribution of Treatment Delay between the Before and After groups.

### Welch's t-test

The program uses:

```text
equal_var=False
```

and therefore performs a t-test without assuming equal variances between the two groups.

Outputs:

```text
MWU_p
TTEST_p
```

together with:

```text
N_Before
N_After
```

for each group.

---

# 9. Module 0304 — Stage Matrix

## `0304_AP_StageMatrix.py`

This module is separate from the primary Treatment Delay analysis.

Its main purpose is to examine:

> **The correspondence between Clinical Stage and Pathological Stage.**

It creates a:

> **Clinical Stage × Pathological Stage**

cross-tabulation matrix.

---

## 9.1 Raw Stage Matrix

The program first uses the original:

```text
CR_0307
CR_0313
```

to construct a detailed Stage matrix.

---

## 9.2 Grouped Stage Matrix

It then applies the Stage grouping defined in `config.py` to create:

```text
Stage_4
Stage_5
```

matrices.

Each matrix ultimately includes:

```text
Total
```

to provide row and column totals.

---

# 10. Module 04 — Presentation F2A

## `04_Presentation_F2A.py`

This module does not recalculate the statistics. Instead, it recombines the summary and p-value results generated by previous modules.

Inputs:

```text
TW summary
TW p-value

Global summary
Global p-value
```

Output:

```text
Final_table_[Stage]_F2A.csv
```

F2A uses a compact format:

```text
N
Mean (Median)
MWU p
T-test p
```

Taiwan and Global results are placed into the same table.

---

# 11. Module 04 — Presentation F2B

## `04_Presentation_F2B.py`

The primary difference between F2B and F2A is the amount of information presented.

F2B additionally retains:

```text
Mean ± SD
Median (IQR)
```

Therefore, the complete format is:

```text
N
Mean ± SD
Median (IQR)
MWU p
T-test p
```

It similarly integrates:

* TW
* Global

results.

---

# 12. Module 05 — Excel Figure 2

## `05_ExcelCombine_Fig2.py`

This module combines the previously generated:

```text
*_F2A.csv
*_F2B.csv
```

files into a single Excel workbook.

Output filenames are generated according to the cancer directory:

```text
LungCancer_Fig2.xlsx
ColorectalCancer_Fig2.xlsx
LiverCancer_Fig2.xlsx
```

Each CSV becomes a separate worksheet within the Excel workbook.

This module is primarily responsible for **result packaging and delivery-format organization** and does not perform statistical analysis again.

---

# 13. Module 05 — Excel Stage Matrix

## `05_ExcelCombine_StageMatrix.py`

This module combines:

```text
*_matrix.csv
```

files into:

```text
[ Cancer Name ]_StageMatrix.xlsx
```

and places the matrices into the Excel workbook.

This module likewise belongs to the Presentation / Output layer and does not recalculate the Stage Matrix.

---

# 14. Master Controller

## `00_RUN.py`

`00_RUN.py` is the main execution entry point for the entire pipeline.

It executes the modules in sequence:

```text
02_Transformation.py
        ↓
03_Analysis.py
        ↓
03_Analysis_pvalue.py
        ↓
0304_AP_StageMatrix.py
        ↓
04_Presentation_F2A.py
        ↓
04_Presentation_F2B.py
        ↓
05_ExcelCombine_Fig2.py
        ↓
05_ExcelCombine_StageMatrix.py
```

Therefore, users do not need to manually execute each analysis program in sequence.

`00_RUN.py` calls the modules according to the predefined execution order.

---

# 15. Configuration

Some analysis specifications are centralized in:

```text
config.py
```

These primarily include:

* Stage analysis configuration
* Stage ordering
* TW period
* Global period
* Stage Matrix configuration

Therefore, if the analysis grouping or presentation order needs to be modified in the future, the configuration can generally be adjusted first without modifying every analysis module.

---

# 16. Complete Data Flow

The entire system can be simplified as:

```text
T02 Cancer Registry
        │
        ▼
01 Extraction
        │
        ├── Lung
        ├── Colorectal
        └── Liver
        │
        ▼
01_T02_mini_cohort
        │
        ▼
02 Transformation
        │
        ├── Diagnosis Date
        ├── Treatment Date
        ├── Treatment Delay
        ├── Stage
        ├── TW Period
        └── Global Period
        │
        ├───────────────────┐
        ▼                   ▼
03 Analysis          0304 Stage Matrix
        │
        ▼
03 Analysis p-value
        │
        ├───────────────┐
        ▼               ▼
       F2A             F2B
        │               │
        └───────┬───────┘
                ▼
        Excel Figure 2

Stage Matrix
      │
      ▼
StageMatrix Excel
```

---

# 17. Module Classification

From a system architecture perspective, the programs can be divided into four layers:

| Layer              | Modules              | Primary Function                           |
| ------------------ | -------------------- | ------------------------------------------ |
| **Extraction**     | 01                   | Establish research data and cancer cohorts |
| **Transformation** | 02                   | Create analytical variables and groupings  |
| **Analysis**       | 03, 03 p-value, 0304 | Statistical analysis and Stage Matrix      |
| **Presentation**   | 04, 05               | Table integration and Excel output         |

Therefore, the primary responsibilities of the modules are separated:

> **Extraction retrieves the data; Transformation defines the analysis dataset; Analysis calculates the results; Presentation organizes the results into usable formats.**

---

# 18. Execution Concept

Under normal circumstances, users do not need to understand or execute each Python program individually.

The overall workflow is designed as:

```text
Prepare Raw Data
      ↓
Run Extraction
      ↓
Create Cancer Directories
      ↓
Run 00_RUN.py
      ↓
Automatically complete:
      ↓
Transformation
      ↓
Analysis
      ↓
P-value Analysis
      ↓
Stage Matrix
      ↓
F2A / F2B
      ↓
Excel
```

Therefore, `00_RUN.py` can be regarded as the primary entry point for the entire analysis workflow.

---

# 19. Final Outputs

After execution within each cancer directory, the main results fall into two categories.

## Treatment Delay Analysis

```text
Final_table_[Stage]_F2A.csv
Final_table_[Stage]_F2B.csv
```

and the integrated:

```text
[ Cancer ]_Fig2.xlsx
```

---

## Stage Analysis

```text
Stage_matrix.csv
Stage_4_matrix.csv
Stage_5_matrix.csv
```

and:

```text
[ Cancer ]_StageMatrix.xlsx
```

---

# 20. Core System Logic

This program is not a single analysis script. Instead, it divides a complete analytical workflow into multiple stages:

```text
Raw Data
   ↓
Cohort
   ↓
Derived Variables
   ↓
Stratification
   ↓
Statistical Analysis
   ↓
Presentation
   ↓
Excel Deliverable
```

The most important data transformation is:

```text
Cancer Registry
      ↓
Treatment Delay
      +
Stage
      +
COVID Period
      ↓
Before vs After
      ↓
Statistical Comparison
```

---

# 21. Summary

The primary purpose of this system is to establish a reproducible cancer data analysis workflow for comparing Treatment Delay before and after COVID-19 time periods.

The analysis consists of three primary dimensions:

### Cancer

* Lung
* Colorectal
* Liver

### Stage

* Stage Framework
* Clinical / Pathological Stage

### Period

* TW
* Global
* Before
* After

Descriptive statistics and two statistical tests are ultimately used to compare Treatment Delay across different cancers, stages, and COVID-19 time frameworks.

The overall analytical structure can be condensed into:

> **Cancer × Stage × COVID Period → Treatment Delay → Statistical Comparison → F2A/F2B → Excel**

The software architecture uses a modular:

> **Extraction → Transformation → Analysis → Presentation**

design, with `00_RUN.py` centrally controlling the execution order.

---

# Appendix: Program Module Overview

| No. | Program                          | Type           | Function                                 |
| --- | -------------------------------- | -------------- | ---------------------------------------- |
| 01  | `01_Extraction_T02_Three.py`     | Extraction     | T02 stratification and cohort creation   |
| 02  | `02_Transformation.py`           | Transformation | Treatment Delay, Stage, and COVID period |
| 03  | `03_Analysis.py`                 | Analysis       | Descriptive statistics                   |
| 03  | `03_Analysis_pvalue.py`          | Analysis       | MWU / Welch's t-test                     |
| 03  | `0304_AP_StageMatrix.py`         | Analysis       | Clinical × Pathological Stage Matrix     |
| 04  | `04_Presentation_F2A.py`         | Presentation   | F2A summary table                        |
| 04  | `04_Presentation_F2B.py`         | Presentation   | F2B detailed table                       |
| 05  | `05_ExcelCombine_Fig2.py`        | Presentation   | F2 results Excel workbook                |
| 05  | `05_ExcelCombine_StageMatrix.py` | Presentation   | Stage Matrix Excel workbook              |
| 00  | `00_RUN.py`                      | Controller     | Full pipeline automation                 |

### One-Sentence Version

> **This is a modular analysis pipeline that stratifies T02 cancer registry data by cancer type, transforms the data into Treatment Delay and Stage analysis datasets, performs statistical comparisons across COVID-19 Before/After periods and TW/Global time frameworks, and automatically generates F2A, F2B, and Stage Matrix Excel outputs.**
