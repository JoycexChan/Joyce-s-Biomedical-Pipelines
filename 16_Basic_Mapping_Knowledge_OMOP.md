# Basic Mapping Knowledge Required for AI-Assisted OMOP Mapping Review

> **Purpose**
> This document summarizes the basic Mapping knowledge required when using an **AI-assisted review tool** to review OMOP Mapping.
>
> The purpose is not to require Mapping personnel to memorize every OMOP CDM table and field. Instead, it establishes a common reasoning framework so that both Mapping personnel and the AI-assisted review tool can evaluate Mapping from the perspectives of **clinical meaning, Clinical Events, CDM Tables, Standard Concepts, and source traceability**.

---

## 1. Understand the Architecture Map

The architecture map does not need to be memorized.

It should be **kept as a reference and checked continuously during Mapping**. With experience, the structure will become familiar.

The basic structure is:

> **One Patient → Visit → Clinical Event**

A cancer patient may simultaneously have two pathways:

```text
Cancer Patient
      │
      └── Visit
             │
             └── General Clinical Event
```

and:

```text
Cancer Patient
      │
      └── Visit
             │
             └── Cancer Clinical Event
                    │
                    └── EPISODE / EPISODE_EVENT
```

Therefore, when reviewing a source field, do not immediately ask:

> **“Which OMOP field should this source field be mapped to?”**

Instead, first ask:

> **“What clinical information or Clinical Event does this source data represent?”**

---

### 1.1 General Clinical Events

When source information has clinical meaning but cannot reasonably serve as the primary representation of another Clinical Event, it may be stored according to its semantics as an:

`OBSERVATION`

When necessary, `FACT_RELATIONSHIP` may be used to establish a relationship between the Observation and another Clinical Event.

For example:

```text
PROCEDURE_OCCURRENCE
        │
        │ FACT_RELATIONSHIP
        ▼
   OBSERVATION
```

The key principle is:

> **An Observation can describe clinically meaningful information about another Clinical Event without becoming the primary Clinical Event itself.**

---

### 1.2 Cancer Clinical Events

When the **Oncology Extension** is adopted, cancer-related:

* Observations
* Findings
* Attributes

may be treated as **Cancer Condition Modifiers** and stored in:

`MEASUREMENT`

They can be linked to the relevant Clinical Event using:

* `modifier_of_event_id`
* `modifier_of_field_concept_id`

For example:

```text
Cancer Condition
      │
      ▼
   EPISODE
      │
      ▼
 EPISODE_EVENT
      │
      ├── PROCEDURE_OCCURRENCE (Radiotherapy)
      ├── DRUG_EXPOSURE (Chemotherapy)
      ├── PROCEDURE_OCCURRENCE (Surgery)
      └── CONDITION_OCCURRENCE
                   ▲
                   │
              MEASUREMENT
           Cancer Modifier
```

Therefore, **general Observations and Oncology Extension Cancer Modifiers represent different semantic pathways**.

The destination table should not be determined merely by the name of the source field.

---

# 2. Event-Driven Mapping

Mapping should be centered on the **Clinical Event**.

A date is a temporal attribute of an event. It should not independently determine the Mapping.

Therefore, Mapping should not be expected to work like:

```text
Source Field A
      ↓
OMOP Field A
```

Instead, consider the source data as a collection of fields that together describe an event:

```text
Source Data
   │
   ├── Field A
   ├── Field B
   ├── Field C
   ├── Field D
   └── Field E
          │
          ▼
    One Clinical Event
          │
          ▼
      One OMOP Record
```

In other words:

> **Multiple source fields may collectively constitute a single OMOP Clinical Event.**

Conversely, a single source field may describe only an **attribute or modifier of a Clinical Event**, rather than an independent event.

Therefore, the primary Mapping question should be:

> **“What event does this data describe?”**

rather than:

> **“Which OMOP field looks most similar to this source field?”**

---

# 3. Concept IDs and Source Traceability

OMOP Mapping should distinguish three different concepts:

```text
*_concept_id
*_source_concept_id
*_source_value
```

Each has a different purpose.

---

## 3.1 `*_concept_id`

`*_concept_id` represents the **OMOP Standard Concept** and its standardized semantic meaning.

When selecting a Standard Concept, the following must be considered:

> **The Domain and field semantics must be appropriate.**

A Concept should not be selected simply because Athena returns a Concept that appears similar to the source value.

The reasoning should be:

```text
Source Meaning
      ↓
Appropriate Domain
      ↓
Appropriate OMOP Field
      ↓
Appropriate Standard Concept
```

When **Oncology Conventions** are adopted, cancer-related:

> Observation / Finding / Attribute

may be represented as **Cancer Condition Modifiers** and stored in `MEASUREMENT`.

---

## 3.2 `*_source_concept_id`

`*_source_concept_id` is responsible for **source concept traceability**.

Its purpose is to preserve the relationship between the OMOP record and the concept used by the source vocabulary.

The relationship is:

```text
*_source_concept_id
        │
        ▼
CONCEPT.concept_id
        │
        ▼
CONCEPT.vocabulary_id
        │
        ▼
VOCABULARY
```

Therefore:

> **Any source Concept referenced by `*_source_concept_id` should be traceable through `CONCEPT.vocabulary_id` to a clearly identifiable source Vocabulary.**

---

### 3.2.1 Existing OMOP Vocabularies

If OMOP already provides an appropriate Vocabulary, such as an existing ICD Vocabulary, the existing Vocabulary should be used directly.

There is no need to create another local Vocabulary simply to represent the same established Vocabulary.

---

### 3.2.2 Local Source Data

For source systems such as:

* NBCT
* CRLF
* NHI claim codes
* Other local or Taiwan-specific source data

if there is no appropriate existing Vocabulary that can be directly used, a corresponding **Local / Source Vocabulary** should be established.

For example:

```text
NBCT
  ↓
TW_NBCT Vocabulary
```

```text
NHI Claim Codes
  ↓
TW_NHI_CLAIM Vocabulary
```

```text
CRLF
  ↓
TW_CRLF Vocabulary
```

This prevents a source Concept from becoming an orphaned ID whose dictionary context is unknown.

For example, the following is not sufficient by itself:

```text
source_concept_id = 123456
```

The Concept should be traceable to:

```text
CONCEPT
    ↓
vocabulary_id
    ↓
VOCABULARY
```

so that the source dictionary can be identified.

---

### 3.2.3 Why Not Simply Store the Original Option Code?

Suppose the source data contains:

```text
ebrt = 4
```

If the final OMOP record only contains:

```text
source_value = "4"
```

the meaning may become ambiguous.

For example:

```text
ebrt = 4
srs  = 4
other = 4
```

may all be valid source values.

Therefore, the value `4` alone cannot reliably identify its original field or source context.

Instead:

```text
source_concept_id
        ↓
CONCEPT
        ↓
vocabulary_id
        ↓
Source Vocabulary
```

provides the necessary source traceability.

---

# 4. `*_source_value`

As a general principle:

> **The original value is the cleanest value to preserve in `*_source_value`.**

For example:

```text
source_value = "4"
```

The source Vocabulary provides the semantic interpretation:

```text
source_concept_id
        ↓
TW_CRLF
        ↓
Code = 4
        ↓
EBRT Technique ...
```

Thus, the three components have different responsibilities:

```text
*_concept_id
    → OMOP Standard Concept / standardized meaning

*_source_concept_id
    → Source Concept and Vocabulary traceability

*_source_value
    → Original source value
```

---

## 4.1 When the Mapping Is Not Yet Complete

In practice, the appropriate source Vocabulary or Source Concept may not yet have been established.

In this situation, the source meaning should not be discarded simply to keep the final format clean.

A temporary representation such as:

> **Original Value: Field / Option Description**

may be used.

For example:

```text
4: EBRT Technique / IMRT, IMPT
```

Once the source Vocabulary has been established or confirmed:

1. Create the corresponding `*_source_concept_id`.
2. Restore `*_source_value` to the original source value.

For example:

```text
source_concept_id
        ↓
TW_CRLF / EBRT / 4

source_value
        ↓
"4"
```

This prevents information from being lost during the Mapping process.

> **Principle: Temporary redundancy is preferable to losing source semantics during an incomplete Mapping process.**

---

# 5. Mapping Decision Flow

The Mapping process can follow the sequence below:

```text
                     Source
                       │
                       ▼
                Identify Clinical Meaning
                       │
                       ▼
                Identify Clinical Event
                       │
                       ▼
             Determine Appropriate CDM Table
                       │
                       ▼
             Search Standard Vocabulary
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Appropriate           None
          Standard               │
             │                   ▼
             │            Check Existing Source
             │            Vocabulary / Local
             │                   │
             │          ┌────────┴────────┐
             │          ▼                 ▼
             │         Yes                No
             │          │                 │
             │          │                 ▼
             │          │          Establish Local Vocabulary
             │          │                 │
             └──────────┴─────────────────┘
                        ▼
                Create source_concept_id
                        │
                        ▼
                Create concept_id
          (if an appropriate Standard
           Concept is available)
                        │
                        ▼
                Preserve source_value
                        │
                        ▼
                     Complete
```

The important point is that **Vocabulary Mapping occurs after the clinical meaning and appropriate CDM representation have been determined**.

The process should not begin by searching for a Concept based solely on the source field name or source code.

---

# 6. Minimum Review Principles for the AI-Assisted Review Tool

When reviewing a Mapping proposal, the AI-assisted review tool should be able to evaluate at least the following questions.

### ① What does the source information describe?

> What is the clinical meaning of the source field or source value?

### ② What kind of information is it?

> Is it a Clinical Event, an Event Attribute, an Observation, or a Cancer Condition Modifier?

### ③ Why is it stored in this CDM Table?

> Can the Mapping explain why the selected CDM Table represents the source semantics?

### ④ Is `*_concept_id` appropriate?

> Does the Standard Concept have the correct Domain and semantic meaning?

### ⑤ Is `*_source_concept_id` traceable?

> Can the Source Concept be traced through `CONCEPT` to the correct `vocabulary_id` and source Vocabulary?

### ⑥ Is `*_source_value` traceable to the source?

> Can the final OMOP record still be traced back to the original source value?

---

# Core Principle

> **OMOP Mapping is not field-to-field translation.**

The overall reasoning path should be:

```text
Source
  ↓
Clinical Meaning
  ↓
Clinical Event
  ↓
CDM Table
  ↓
Standard Concept
  ↓
Source Concept / Vocabulary
  ↓
Source Value
```

The purpose of this framework is not to require users to memorize every OMOP CDM table and field.

Instead, it establishes a consistent Mapping process:

> **Understand the source data → identify the Clinical Event → determine the appropriate CDM representation → identify the Standard Concept → preserve source traceability → preserve the original value.**

With repeated Mapping, the architecture will become increasingly familiar. During review, however, the architecture map should remain the primary reference rather than relying on field-name similarity or intuition alone.

The role of the **AI-assisted review tool** is to help check whether a proposed Mapping follows this reasoning framework. It should **assist the Mapping reviewer, not replace the reviewer's understanding of the source data and its clinical meaning**.
