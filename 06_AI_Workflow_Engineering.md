# How I Solved GPT's "Start Over" Problem
________________________________________
## "Every new GPT chat meant starting over.
## Weeks of accumulated reasoning disappeared the moment I opened a new conversation.
## So I spent a year building three tools to fix it."
________________________________________
## AI Reasoning Workflow Engineering
Building reproducible reasoning environments for GPT
Why did this project start?

This project originated as a personal interest rather than a commercial AI project.

In 2025, while building an AI-assisted work journal, I compressed approximately twenty days of GPT conversations into structured summaries to continue working across new chat sessions.

During this process, I observed an interesting phenomenon:

GPT responses become significantly more precise after a stable reasoning context has been established.

Rather than simply remembering previous conversations, GPT gradually builds an internal working model of the user and predicts which reasoning paths are most likely to match the user's intent.

I refer to this state as a Shared Brain.

The remainder of this project explores how to reproduce, initialize, and preserve that reasoning environment across independent chat sessions.
________________________________________
## A software engineering analogy

Rather than viewing GPT as a standalone assistant, this project treats it as a computing platform.

GPT
↓
Foundation Model
(Comparable to computer hardware)

↓

IRR
Reasoning Runtime
(Comparable to an operating system)

↓

User Model Card
(Comparable to system configuration)

↓

Research Workflow

GPT provides the reasoning capability.

IRR determines how reasoning is performed.

The User Model Card defines the reasoning environment.

Together they allow long-running research conversations to become reproducible instead of restarting from scratch.
________________________________________
## Project Components
### Stable User Model Extractor (SUME)

Extracts the reasoning model that has gradually emerged during long GPT conversations.

Instead of losing months of accumulated context when opening a new chat, the extracted User Model Card allows the reasoning environment to be reconstructed.

Purpose

Continue previous work without rebuilding context
Iteratively improve the User Model Card over time
Preserve long-term reasoning consistency

### Shared Brain Initialization Protocol (SBIP)

Rather than waiting for GPT to gradually infer the user's reasoning style, SBIP actively constructs the User Model through structured interaction.

Purpose

Generate a complete User Model Card from scratch
Reduce initialization time
Produce reproducible reasoning environments

### Incremental Reasoning Runtime (IRR)

IRR is the reasoning runtime responsible for preserving the Shared Brain during subsequent conversations.

Instead of acting as a reviewer, IRR is designed to support collaborative hypothesis generation and research-oriented reasoning.

Its objective is not to maximize safety-oriented responses, but to maximize reasoning precision while minimizing information contamination.

IRR specifically attempts to reduce four common failure modes:

* Fabricated supporting evidence

Generating evidence that merely agrees with the user instead of explicitly separating observations from hypotheses.

* Probability flattening

Replacing an internal 80/20 working model with a "50/50" presentation, thereby losing ranking information.

* Intent drift

Answering a different question because the assistant incorrectly inferred the user's intent.

* Premature review-mode interruption

Requesting evidence that does not contribute to the current hypothesis-generation stage, effectively terminating exploration instead of proposing competing models or counterexamples.
________________________________________
## Real-world application

Although these projects originated from personal AI research, the underlying concepts have been directly applied to biomedical research workflows.

Current applications include:

Knowledge extraction
Clinical ETL workflow design
AI-assisted OMOP mapping review
Research workflow automation

Rather than replacing researchers, these workflows aim to reduce repetitive reasoning overhead while preserving reproducibility.
