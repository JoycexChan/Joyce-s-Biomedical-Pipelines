# OMOP Governance Review

## Using GPTs to Assist with OMOP Mapping Governance Review

This is a set of GPT-based tools designed to assist with **OMOP Governance Review**.

If you have:

* Field names
* Field descriptions
* Field options / code lists
* Data Dictionary / Codebook
* Existing FHIR mappings
* Existing OMOP mappings

you can provide them directly to the GPT and ask it to assess:

> **What does this field actually represent? Should it be included in OMOP? If it should, is the current mapping appropriate?**

---

# Use the GPTs Directly

## ① OMOP Governance Review (template)

**For users who want to use the predefined review framework directly.**

This GPT includes:

* **OMOP CDM specifications**
* **NBCT specifications** (publicly available information)
* **An OMOP Governance working model**

You can directly provide your data and start the review.

**GPT:**
👉 [OMOP Governance Review (template)](https://chatgpt.com/g/g-6a7cba37dcf881919a1d494cd3208b15-omop-governance-review-template)

---

## ② OMOP Governance Review (use your data)

**For users who have their own national or organizational specifications.**

This version primarily contains:

* **OMOP CDM specifications**
* **Governance Review Framework**

You can upload your own:

* National data specifications
* Registry specifications
* Data Dictionaries
* Hospital / organizational rules
* Business rules

An **OMOP review reasoning model** is also provided.

👉 [**My OMOP Governance working model**](./09_SUME_example.md)

If you want the GPT to follow this reasoning approach, you can upload the model together with your own specifications.

**GPT:**
👉 [OMOP Governance Review (use your data)](https://chatgpt.com/g/g-6a7cca9129d88191b927a9b2fc7474a6-omop-governance-review-use-your-data)

---

# How to Use It

It is actually very simple.

Provide all the information about the field you want to review.

For example:

```text
Field name:
CASE.m6

Field description:
Treatment status

Field options:
1 = Complete response
2 = Partial response
3 = Stable disease
4 = Progression
5 = Inevaluable
```

If an existing mapping has already been created, you can provide that as well:

```text
FHIR Mapping:
Observation

Current OMOP Mapping:
OBSERVATION.value_as_concept_id
```

Then simply ask:

> **Please review how this field should be mapped to OMOP.**

Or:

> **Please recommend how this field should be modeled in OMOP and provide an example template.**

---

## If You Have a Large Amount of Information

You do not need to format everything into a perfect table first.

You can provide:

* Field names
* Field descriptions
* Code lists
* Data Dictionary
* Mapping tables
* Engineer comments

and let the GPT organize the information required for the review.

---

# ⚠️ Important Note Before Using It

This tool is a **Governance Review assistant**, not an automated mapping black box.

Users should still have a basic understanding of:

* OMOP CDM
* OMOP Domains
* Concepts
* Vocabularies
* Source Concepts
* Standard Concepts

in order to interpret the review results.

The GPT can help transform:

> “I think this field should go here.”

into a more structured Governance question.

However, **the final mapping decision still requires human judgment from someone with appropriate domain and OMOP knowledge.**

---

# What Does the Review Engine Actually Review?

The purpose of this GPT is not:

> **Mapping Generator**

but rather:

> **Mapping Reviewer / Governance Reviewer**

In other words, it does not simply see:

```text
There is a field
      ↓
OMOP has a field that looks suitable
      ↓
Put it there
```

Instead, it works in the opposite direction:

```text
Source Field
      ↓
What does this actually represent?
      ↓
Why does this field exist?
      ↓
Is it clinical information?
      ↓
Should it enter OMOP?
      ↓
If so, how should it be modeled?
      ↓
Is the current mapping semantically appropriate?
```

The core principle is:

> **Semantic correctness > Storage availability**

In other words:

> **The fact that OMOP has somewhere to store a field does not mean that the field should be stored there.**

This is the core Governance principle of the Review Board. 

---

# Review Workflow

The overall process can be summarized as:

```text
Source Field
     │
     ▼
① Semantic Classification
     │
     ▼
② Source Semantic Analysis
     │
     ▼
③ Evidence Sufficiency
     │
     ▼
④ OMOP Eligibility
     │
     ▼
⑤ Event Model
     │
     ▼
⑥ Information Loss
     │
     ▼
⑦ Current Mapping Review
     │
     ▼
⑧ FHIR Consistency Review
     │
     ▼
⑨ Competing Architecture
     │
     ▼
⑩ Governance Verdict
     │
     ▼
🟢 GREEN / 🟡 YELLOW / 🔴 RED
```

---

# Step 0 — Semantic Classification

The first step is **not** to look for an OMOP table.

Instead, classify the Source Field:

* Clinical Event
* Clinical Attribute
* Administrative Data
* Billing Data
* Registry Data
* Workflow Data
* Metadata

If the field is:

* Administrative Data
* Billing Data
* Workflow Data
* Metadata

the review further asks:

> **Should this information enter OMOP at all?**

Not every piece of source data should necessarily become an OMOP Clinical Record. 

---

# Step 1 — Source Semantic Analysis

The review then asks:

1. What does this field actually represent?
2. Why was this field created?
3. What is its business purpose?
4. Is it a Clinical Event?
5. Is it primarily Clinical, Administrative, Workflow, Registry, or Billing Information?

The purpose is:

> **Understand the Source first, then discuss OMOP.**

Rather than seeing a field name and immediately searching for a Concept ID.

---

# Step 1.5 — Evidence Sufficiency Review

The next question is:

> **Is there enough evidence to make the decision?**

Possible results include:

```text
Sufficient
Partially Sufficient
Insufficient
```

For example, suppose the only information available is:

```text
STATUS = 4
```

but nobody knows:

```text
What does 4 mean?
Why does this field exist?
Where is the field generated?
```

The reviewer should not guess.

Additional information may be required, such as:

* Data Dictionary
* Registry Specification
* Business Rules
* Sample Data
* Source System Design
* Codebook

If the evidence is insufficient, the review can be:

> **Governance Review Deferred**

rather than generating a seemingly complete mapping based on assumptions. 

---

# Step 2 — OMOP Eligibility Review

Once the Source Meaning is understood, the next question is:

> **Should this information enter OMOP?**

Possible outcomes include:

```text
A. Standard OMOP Event
B. Event Attribute
C. Source Value Only
D. Observation Extension
E. ETL Metadata
F. Do Not Load
```

Most importantly:

> **Do Not Load is a valid answer.**

Not every source field must be forced into an OMOP table. 

---

# Step 3 — Event Model Review

If the data should enter OMOP, the next question is whether it represents:

* An Event
* An Event Attribute
* An Administrative Attribute
* Metadata
* Source Traceability

For example:

```text
A treatment event
+
an attribute describing that treatment event
```

is not the same thing as:

```text
An independent clinical event
```

If a field does not represent an independent Event, it should not be turned into a separate Clinical Record simply because OMOP happens to have a place where the value could be stored. 

---

# Step 3.5 — Information Loss Analysis

The review then asks:

> **What information will be lost if the current mapping is used?**

It evaluates:

* Semantic Loss
* Workflow Loss
* Billing Context Loss
* Registry Context Loss
* Traceability Loss

and classifies the result as:

```text
Reversible
Partially Reversible
Irreversible
```

For example, suppose the source contains:

```text
4 = Progression
```

If the mapping keeps only a Standard Concept but completely loses the original source information, the reviewer should ask:

> **Can we still determine what the original `4` meant?**

The review therefore asks not only:

> “Can this be mapped?”

but also:

> **“How much of the original meaning remains after mapping?”** 

---

# Step 4 — Current Mapping Review

If the user already has a mapping:

```text
FHIR:
Observation

OMOP:
OBSERVATION.value_as_concept_id
```

the reviewer evaluates it directly:

```text
Accept
Questionable
Reject
```

and explains why.

The important principle is:

> **An existing mapping is not automatically correct simply because someone has already implemented it.**

The Review Board is intended to challenge existing mappings rather than rationalize them. 

---

# Step 4.25 — Engineer Intent Reconstruction

Sometimes a questionable mapping was not created arbitrarily.

It may have resulted from:

* Legacy system limitations
* FHIR structural constraints
* CDM version limitations
* Data completeness requirements
* Historical ETL design
* Time constraints

The reviewer can therefore attempt to reconstruct:

```text
Likely Intent
Potential Constraints
Governance Interpretation
```

However:

> **Understanding why an engineer made a decision does not mean accepting the mapping.**

It preserves the historical design context. 

---

# Step 4.5 — FHIR Consistency Review

If FHIR Mapping is available, an additional review layer can check:

* FHIR Semantic Drift
* FHIR Resource Misuse
* FHIR → OMOP Double Translation Error

In other words:

> **A FHIR mapping can also be wrong.**

A mapping such as:

```text
FHIR → Observation
```

does not automatically imply:

```text
OMOP → Observation
```

FHIR is a separate semantic model and should be evaluated independently. 

If the user does not have FHIR:

> **This review layer is simply skipped.**

---

# Step 5 — Competing Architecture Models

If multiple reasonable architectures exist, the review compares them.

For example:

```text
Model A
Standard OMOP Event

Model B
Event + Attribute

Model C
Source Value / Local Concept
```

Each model can be compared by:

* Mapping Strategy
* Supporting Evidence
* Weaknesses
* Information Preservation
* Confidence Level

However, the existence of another possible model does not automatically reduce confidence in the current model.

A competing model should have comparable or stronger explanatory power before it affects the preferred model. 

**Do Not Load** can also be treated as a competing architecture when supported by the evidence. 

---

# Step 6 — Alternative Design

If the current mapping is not ideal, the reviewer can propose an Alternative Design:

```text
OMOP Table
OMOP Field
Design Rationale
```

If the correct conclusion is:

> **This should not enter OMOP**

the reviewer should simply explain why.

There is no need to force a Storage Location merely because an ETL output is expected. 

---

# Step 7 — Governance Verdict

The final step is the Governance Verdict.

Possible outcomes include:

```text
🟢 Accept
🟡 Questionable
🔴 Reject
```

The review can additionally assess:

```text
Governance Risk
Low / Medium / High

Confidence
Very High / High / Medium / Low
```

and:

```text
Governance Review Required
Yes / No

Governance Review Recommended
Yes / No

ADR Recommended
Yes / Optional / No
```



---

# Step 7.5 — Architecture Risk Matrix

The review can additionally evaluate:

| Risk Type                  | Level               |
| -------------------------- | ------------------- |
| Semantic Drift             | Low / Medium / High |
| Information Loss           | Low / Medium / High |
| Clinical Misinterpretation | Low / Medium / High |
| Future Maintenance Risk    | Low / Medium / High |

This makes the review more specific than simply saying:

> “I don't think this mapping is very good.”

Instead, it identifies:

> **What type of architectural risk does this mapping create?** 

---

# Step 8 — Architecture Smell Detection

The review also looks for Architecture Smells, such as:

* Storage-driven Mapping
* Semantic Mismatch
* Administrative Data disguised as Clinical Event
* Billing Data disguised as Clinical Event
* Metadata disguised as Clinical Event
* Registry Data disguised as Clinical Event
* Workflow Data disguised as Clinical Event
* Source Information Loss
* Workflow Context Loss

When detected, the review marks:

> 🚨 **Architecture Smell**

and identifies:

```text
Root Cause
Potential Impact
Recommended Action
```



---

# Step 9 — Governance Escalation

Finally, the reviewer determines whether escalation is needed:

```text
Governance Review Required
Governance Review Recommended
No Escalation
```

For example, escalation may be recommended when there is:

```text
Semantic Drift = High
```

or:

```text
Information Loss = High
```

or:

```text
Clinical Misinterpretation = High
```

or:

```text
Source Semantics Unclear
```

which may result in:

```text
Governance Review Required = Yes
ADR Recommended = Yes
```



---

# Core Principles of the Review Engine

The entire tool is essentially doing a few things:

```text
Not:

“Where can I put this field?”

Instead:

“What does this field actually represent?”
        ↓
“Why does it exist?”
        ↓
“Does it contain clinical meaning?”
        ↓
“Should it enter OMOP?”
        ↓
“If so, which model is most appropriate?”
        ↓
“What information will be lost?”
        ↓
“What architectural risks does this mapping create?”
```

Its goal is therefore not to:

* Find a Storage Location
* Find a Concept ID
* Rationalize an existing mapping
* Force every field into OMOP

but to:

> **Preserve Source Semantics**
> **Minimize Information Loss**
> **Detect Architecture Risks**
> **Expose Semantic Drift**
> **Reduce Governance Meeting Workload**

If the data should not enter OMOP, the reviewer should say so rather than forcing a mapping. 

---

# Evidence vs. Governance Inference

The review should distinguish between:

```text
Observed Evidence
```

and:

```text
Governance Interpretation
```

For example:

```text
[Source Documentation]
Data Dictionary:
4 = Progression

[Engineer Mapping]
Current Mapping:
Observation.value_as_concept_id

[OMOP Specification]
OMOP field definition...

[Governance Inference]
Therefore, the current mapping may cause...
```

Do not present:

> **GPT inference**

as:

> **Source Documentation evidence.**

The Review Board therefore distinguishes sources such as:

* `[Source Documentation]`
* `[Engineer Mapping]`
* `[Engineer Comment]`
* `[FHIR Specification]`
* `[OMOP Specification]`
* `[Governance Inference]`

to separate evidence from inference. 

---

# Fast Mode / Full Mode

If you only want a quick assessment of a mapping, use **Fast Governance Review**.

It outputs:

```text
Executive Summary
Classification
Current Mapping Review
Risk
Verdict
```

For a complete review, use **Full Governance Review**, which runs the full workflow.

The default is Full Mode unless Fast Mode is explicitly requested. 

---

# Final Note

This GPT is best understood as:

> **A first-round Governance Review tool, not an automated decision-maker.**

It can take:

```text
Field
↓
Semantics
↓
Evidence
↓
OMOP Eligibility
↓
Mapping
↓
Information Loss
↓
Architecture Risk
↓
Governance Verdict
```

and perform a structured first-pass review.

This allows issues that might otherwise require a large Governance meeting to be reduced to:

```text
🟢 Acceptable
🟡 Needs Discussion
🔴 Clear Problem
```

with the genuinely difficult decisions left for human review.

The intended use is therefore not:

> **“Treat GPT as an OMOP expert and let it decide everything.”**

It is:

> **“Let GPT perform the first round of Governance Review, then have someone with appropriate OMOP and domain knowledge interpret the results.”**

The goal is to:

> **Reduce the cognitive and meeting cost of Governance Review—not automate the Governance Decision itself.**
