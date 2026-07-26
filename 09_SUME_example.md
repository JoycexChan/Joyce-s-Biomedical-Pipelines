# Stable User Model (Current Best Working Model)

---

## 1. Capability Model

The user is an experienced data architect and governance reviewer working on a Taiwan National Health Insurance (NHI) to OMOP CDM ETL project, with additional responsibility for National Cancer Registry (NBCT/TS) semantic governance.

Primary capabilities include:

* OMOP CDM v5.3 semantic modeling.
* ETL architecture and documentation.
* Clinical terminology governance.
* Reviewing field-level mappings.
* Evaluating source semantics independently from implementation.
* Designing governance recommendations suitable for review board documentation.

The user is capable of:

* Reading OMOP specifications directly.
* Distinguishing semantic issues from engineering implementation.
* Discussing Oncology Extension vs OMOP v5.3.
* Improving ETL documentation quality and readability.
* Detecting inconsistencies between source semantics and target modeling.

The user frequently asks for governance review rather than implementation advice.

---

## 2. Decision Model

The user's decision hierarchy is consistently:

> **Source Semantic**
>
> ↓
>
> **OMOP Semantic Compatibility**
>
> ↓
>
> **Engineering Feasibility**
>
> ↓
>
> **Implementation Simplicity**

Important characteristics:

* Never change source semantics merely to fit OMOP.
* Governance is independent from ETL.
* ETL constraints should not redefine source meaning.
* Preserve original source meaning whenever possible.
* Prefer explaining trade-offs instead of forcing one solution.

When multiple mappings are acceptable, the user prefers documenting:

* acceptable
* recommended
* future recommendation

instead of declaring one mapping "correct."

---

## 3. Reasoning Model

The user's reasoning style is semantic-first and incremental.

Typical reasoning pattern:

1. Understand source definition.
2. Determine true clinical meaning.
3. Compare with OMOP semantic domain.
4. Compare with Oncology Extension (if applicable).
5. Evaluate implementation.
6. Produce governance recommendation.
7. Assign Review Status:

   * 🟢 GREEN
   * 🟡 YELLOW
   * 🔴 RED

The user strongly prefers:

* explanation before conclusion
* semantic justification
* governance rationale
* explicit assumptions

The user dislikes:

* unsupported assertions
* implementation-first reasoning
* unnecessary complexity
* overfitting mappings to the CDM

---

## 4. Preference Model (Collaboration Preferences)

The user prefers collaborative discussion rather than authoritative answers.

Preferred interaction style:

* Explain reasoning.
* Identify trade-offs.
* Highlight semantic risks.
* Suggest improvements rather than rewrite everything.
* Preserve previous design decisions unless evidence changes.

The user appreciates:

* concise but technically rigorous explanations.
* governance-oriented wording suitable for documentation.
* distinguishing between:

  * OMOP CDM v5.3
  * Oncology Extension
  * ETL implementation
  * Source preservation

For ETL documentation, the user values:

* readability
* reviewer friendliness
* architectural clarity

The user often iterates diagrams and documentation through multiple refinements.

---

## 5. Shared Vocabulary

Frequently used shared terminology:

* Governance
* Semantic-first
* Source Semantic
* Source Preservation
* ETL
* Mapping
* Vocabulary Mapping
* Standard Concept
* Source Concept
* Oncology Extension
* Cancer Modifier
* Measurement
* Observation
* Procedure
* Drug Exposure
* Device Exposure
* Body Site
* Route
* Laterality
* Stage Group
* Stage Descriptor
* AJCC
* TNM
* CRLF Vocabulary
* Athena
* Review Board
* Review Status
* 🟢 GREEN
* 🟡 YELLOW
* 🔴 RED

Frequently discussed source datasets:

* TS (Taiwan Cancer Registry)
* TOTFAO1/2
* TOTFAE

Common ETL documentation sections:

* ETL Overview
* Record Pattern
* Field Mapping

Arrow conventions established for diagrams:

* Black → source value
* Blue → vocabulary mapping
* Orange → indirect/reference relationship

---

## 6. Critical Constraints (High Priority)

The following constraints consistently influence responses:

### Semantic Preservation

Never sacrifice source semantics simply to match OMOP structure.

---

### Governance Independence

Governance review must remain independent from ETL implementation.

---

### Distinguish Standards

Clearly separate:

* OMOP CDM v5.3 recommendations
* Oncology Extension recommendations
* Engineering implementation choices

---

### Review Output Structure

Governance reviews should generally include:

1. Source Semantic
2. OMOP CDM assessment
3. Oncology Extension assessment
4. Field-level evaluation
5. Governance recommendation
6. Final review table
7. Review Status

---

### ETL Documentation

When reviewing ETL documentation:

* Prefer architectural clarity over exhaustive field wiring.
* Distinguish:

  * source mappings
  * vocabulary mappings
  * ETL-generated values
  * reference relationships
* Avoid diagrams that become visually overloaded ("spider web" effect).
* Recommend separating overview diagrams from detailed field mappings when complexity increases.

---

### Practical Modeling Philosophy

When multiple valid implementations exist:

* Prefer preserving information.
* Accept pragmatic ETL solutions when no appropriate OMOP field exists (e.g., retaining non-standard source information in `sig` rather than creating low-value `OBSERVATION` records), provided the semantic limitation is explicitly documented.

---

## 7. Next Session Protocol

**Next Window Instructions:**

Do **NOT** re-initialize.

**Recommended Workflow:**

1. Load this Boot Report.
2. Enter Validation Mode immediately.
3. Validate whether current reasoning still matches the user's semantic-first governance model.
4. If validation succeeds, continue using the established Shared Brain model directly.
5. If discrepancies are found, perform **Local Updates only**; do **not** rebuild the full model.

**Objective:**

Maintain **Shared Brain Synchronization**, not a fresh Boot.

This Boot Report represents the **current best working model**, serving as the default reasoning prior for subsequent sessions. If future conversations provide stronger evidence, update only the affected sections while preserving stable reasoning patterns.
