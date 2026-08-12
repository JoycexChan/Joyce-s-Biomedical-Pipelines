## Designing an ETAP Pipeline Based on Real-World Medical Data Structures

Real-world medical data generally has relatively stable data structures and source hierarchies. However, based on an investigation of the historical 130 R scripts, the research population used during analysis can easily change due to differences between individual programs or manual processing.

Therefore, when conducting medical data analysis, I believe the **research cohort should first be fixed**, and the specified data tables should then be extracted based on this cohort to create the Extraction outputs.

After the cohort has been fixed, the workflow proceeds through:

**Extraction → Transformation → Analysis → Presentation**

In other words:

> **First determine “who the study population is,” then determine “which data should be retrieved,” followed by defining the analytical rules and calculating the results, and finally organizing the outputs in a standardized manner.**

This can reduce confusion caused by different analytical modules independently redefining their data populations, while allowing the same data-processing modules to be reused across different analytical tasks.

---

# 1. What Is ETAP?

**ETAP Pipeline** is the modular data-processing architecture used for data analysis in this project. It separates data-processing tasks that might otherwise be mixed together within a single program into four stages:

**Extraction → Transformation → Analysis → Presentation**

The four stages are responsible for:

| Stage              | Function                                          | Core Question                                       |
| ------------------ | ------------------------------------------------- | --------------------------------------------------- |
| **Extraction**     | Extract the data required for analysis            | What data do I need to use?                         |
| **Transformation** | Create new fields and rules required for analysis | How do I transform the raw data into analysis data? |
| **Analysis**       | Perform statistical calculations                  | What do I need to calculate?                        |
| **Presentation**   | Integrate and organize analytical results         | How should the results be presented?                |

---

# 2. Extraction — Data Extraction

Extraction is the first stage of the data pipeline.

Its primary purpose is to extract the data and fields required for the current task from the raw data and create a **mini dataset**.

The key principle is:

> **Reduce the data scope without changing the structure or meaning of the original data.**

For example, the original dataset may contain a large number of fields, while the current analysis only requires a subset of them. A mini dataset can therefore be created first:

```text id="qk4d5n"
Raw Data
   ↓
Extraction
   ↓
Mini Dataset
```

If the study requires a specific patient population, a **Cohort** can also be established at this stage.

For example:

```text id="7ujj1p"
Raw Medical Data
       ↓
Extraction
       ↓
Target Cohort
       ↓
COVID Cohort
```

Therefore, Extraction also establishes the data entry point for subsequent analysis.

---

# 3. Transformation — Data Transformation

Transformation converts **raw fields** into **variables required for analysis**.

This stage does not directly perform the final statistical analysis. Instead, it explicitly encodes the rules required by subsequent analysis into the dataset.

For example:

```text id="1c7z2b"
Diagnosis Date
+
Treatment Date
       ↓
Treatment Delay
```

or:

```text id="kn9q8s"
Original Stage
       ↓
Stage Mapping
       ↓
Stage_4 / Stage_5
```

An important principle of Transformation is:

> **New fields required by subsequent analyses should, whenever possible, be created consistently at this layer.**

This prevents later Analysis modules from independently recalculating the same rules.

For example, the current ETAP implementation creates analytical variables such as:

* Treatment Delay
* Stage groups
* TW / Global periods

during the Transformation stage.

The resulting data flow is therefore:

```text id="8sv0mq"
Mini Dataset
     ↓
Transformation
     ↓
Analysis Dataset
```

---

# 4. Analysis — Statistical Analysis

The Analysis stage performs calculations on the analysis dataset prepared by Transformation.

For example:

* N
* Mean
* Median
* SD
* IQR
* p-value
* Statistical Test

Simple statistical calculations can be centralized within the same analysis module, while different types of statistical tests can be separated into different modules.

For example, the current project separates descriptive statistics from p-value calculations:

```text id="0sx2dj"
Analysis Dataset
       │
       ├── Descriptive Statistics
       │
       └── Statistical Tests
```

The core principle of this stage is:

> **Analysis determines “what to calculate,” rather than redefining how the data is defined.**

Data definitions and the creation of new analytical variables should, whenever possible, be completed during the Transformation stage.

---

# 5. Presentation — Results Organization

Presentation is the final layer.

It should not perform the analysis again. Instead, it organizes the results already generated by Analysis into the format required by the user.

For example:

```text id="x7apn2"
Analysis Results
      ↓
Presentation
      ↓
Target Table
      ↓
Excel
```

The F2A / F2B modules in the current project belong to this layer.

For example:

### F2A

* N
* Mean (Median)
* p-value

### F2B

* N
* Mean ± SD
* Median (IQR)
* p-value

Therefore, the primary purpose of Presentation is:

> **Organize already-calculated results into a format that is easy to read and deliver.**

At the same time, it minimizes the amount of manual formatting that users need to perform in Excel.

---

# 6. Complete ETAP Data Flow

The overall workflow can be represented as:

```text
                    Raw Data
                       │
                       ▼
              ┌─────────────────┐
              │   Extraction    │
              │   Data Extraction│
              └────────┬────────┘
                       │
                       ▼
                 Mini Dataset
                       │
                       ▼
              ┌─────────────────┐
              │ Transformation  │
              │ Data Transformation
              └────────┬────────┘
                       │
                       ▼
                Analysis Dataset
                       │
                       ▼
              ┌─────────────────┐
              │    Analysis     │
              │ Statistical Analysis
              └────────┬────────┘
                       │
                       ▼
                 Analysis Results
                       │
                       ▼
              ┌─────────────────┐
              │  Presentation   │
              │ Results Organization
              └────────┬────────┘
                       │
                       ▼
                  Final Output
```

# Core Principle

> **Prepare the data first, encode the rules into the data, perform statistical analysis only after the data definitions are complete, and organize the final output at the end.**

This is the core principle behind the ETAP Pipeline used in the current project.
