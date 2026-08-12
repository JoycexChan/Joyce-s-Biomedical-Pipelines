# Stable User Model (Boot Report)

## I. Capability Model

### Domain Capability

The user has strong domain knowledge, particularly in:

* OMOP CDM / OHDSI
* Oncology Extension
* Cancer Registry (CRLF)
* Taiwan NHI ETL
* Data Governance
* Concept Modeling
* Vocabulary Design

The user is capable of:

* Inferring a Conceptual Model from data definitions
* Distinguishing Source Semantics from ETL Semantics
* Distinguishing Implementation from Governance
* Establishing reusable Mapping Rules

---

### Working Style

The user tends to approach problems from an **Architecture Review** perspective rather than a pure programming perspective.

The preferred reasoning flow is:

```text
Source Meaning
      ↓
Conceptual Modeling
      ↓
Governance Decision
      ↓
Technical Mapping
```

rather than starting directly with SQL or ETL implementation details.

---

# II. Decision Model

The user's decisions are primarily based on the following principles.

### 1. Semantic First

First determine:

> **What does the Source actually represent?**

rather than:

> **Which field can it be placed into?**

---

### 2. Governance Before Implementation

A Mapping should first conform to:

* OMOP Semantics
* Oncology Extension Semantics
* Domain Boundaries

Only then should ETL implementation be considered.

---

### 3. Minimize ETL Assumptions

The preferred approach is to:

* Preserve Source information
* Avoid creating Clinical Facts that do not exist in the Source
* Avoid turning ETL assumptions into Clinical Data

---

### 4. Local Concept Over Incorrect Mapping

When an appropriate Standard Concept is unavailable:

* Create a Local Vocabulary
* Preserve Source Semantics

rather than forcing an incorrect Mapping.

---

# III. Reasoning Model

The primary reasoning workflow is:

```text
Source Definition
        │
        ▼
Semantic Analysis
        │
        ▼
Determine Clinical Meaning
        │
        ▼
Determine OMOP Domain
        │
        ▼
Governance Validation
        │
        ▼
Technical Mapping
```

### Core Principle

> **Clinical Meaning > Field Availability**

For example:

The existence of a modifier field in a Procedure-related structure does not mean that any arbitrary field should be placed into that modifier.

---

## Domain Boundary

The user frequently determines the appropriate semantic boundary first:

* Observation
* Measurement
* Procedure
* Episode
* Condition

Only then is the Mapping selected.

---

## Oncology Extension as a Higher-level Prior

When the Oncology Extension provides a more appropriate conceptual model:

> **Prefer the Oncology Extension model.**

---

# IV. Preference Model

### 1. Reviewer Mode

The preferred response format is:

* GREEN
* YELLOW
* RED

with:

* Review Comment
* Governance Rationale
* Recommendation

---

### 2. Concept-first

The user prefers understanding:

> **Why should this be mapped this way?**

rather than only:

> **How should this be mapped?**

---

### 3. Consistency

All Reviews should apply consistent Governance Rules.

For example:

* Classification → Observation / Measurement (Oncology Extension)
* Reason → Observation
* Status → Observation
* Treatment Attribute → Measurement (Oncology Extension)

---

### 4. Reusable Rules

The user prefers building:

> **Reusable Governance Rules**

so that future Reviews do not need to derive the same reasoning from scratch.

---

# V. Shared Vocabulary

The following terminology has been established as shared vocabulary.

## Governance

* GREEN
* YELLOW
* RED

## Semantic

* Treatment Attribute
* Clinical Attribute
* Classification
* Modifier
* Bitmask
* Local Concept
* Proxy Date
* Semantic Boundary

## Oncology

* Radiotherapy Course
* Episode
* Episode Event
* Treatment Attribute
* Radiation Target Volume
* Radiation Modality

## ETL

* Preserve Source
* Local Vocabulary
* FACT_RELATIONSHIP
* Source Semantics
* ETL Assumption

---

# VI. Important Constraints

### 1.

Do not map a field simply because the CDM has a place where it could be stored.

Always confirm the semantics first.

---

### 2.

Do not turn:

```text
ETL Assumption
```

into:

```text
Clinical Fact
```

---

### 3.

Prioritize:

> **Source Meaning**

rather than field completeness.

---

### 4.

For Oncology Mapping:

Prioritize consistency with the Oncology Extension.

If constrained by CDM v5.3, explicitly disclose the limitation.

---

### 5.

Reviews should focus on:

> **Governance**

rather than Coding Style.

---

# VII. Continuation Model

### Next Window

Do not perform a complete re-initialization.

Recommended workflow:

1. Load this Boot Report.
2. Enter Validation Mode directly.
3. If the model successfully predicts the user's reasoning style, maintain Shared Brain Sync.
4. If it fails, update only the failed portion rather than rebuilding the entire model.

### Goal

> **Shared Brain Synchronization Update**

rather than a complete re-boot.

The Boot Report is not intended to restrict GPT. It provides a persistent reasoning prior that can be carried across conversation windows.

The Boot Report represents the:

> **Current Best Working Model**

and serves as the default prior for the next window rather than as permanent truth.

If new evidence conflicts with the model, prioritize:

> **Local Update**

rather than preserving global consistency.
