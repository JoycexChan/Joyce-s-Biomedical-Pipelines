# How I Solved GPT's "Start Over" Problem with SUME
## Allowing long-running research conversations to become reproducible instead of restarting from scratch.
________________________________________
## Stable User Model Extractor (SUME)

Stable User Model Extractor (SUME) extracts the reasoning model that gradually emerges during long GPT conversations.

Every new GPT chat normally starts from scratch. SUME changes that.

Instead of losing months of accumulated reasoning when opening a new chat, the extracted User Model Card allows the reasoning environment to be reconstructed.
________________________________________
## Purpose
• Continue previous work without rebuilding context.
• Iteratively improve the User Model Card over time.
• Preserve long-term reasoning consistency across conversations.
________________________________________
## How to Use
### Step 1

Copy the complete SUME prompt from GitHub.

👉 Copy the SUME prompt on GitHub

https://...

### Step 2
Paste the prompt into your long-running GPT conversation.
This process does not modify your existing conversation.

SUME will generate a User Model Card describing the reasoning model that has gradually emerged in that conversation.

👉 Here is an example User Model Card extracted from one of my OMOP research conversations:
https://...

### Step 3

Copy the generated User Model Card into a new GPT conversation.

The new conversation can then reconstruct a comparable reasoning environment instead of starting from scratch.
________________________________________
## What SUME Does

The User Model Card reconstructs how GPT has learned to reason with you—not your source materials.

You still need to provide the documents, datasets, or project files required for the new conversation.

SUME reconstructs the reasoning environment.

It does not reconstruct your documents, datasets, project files, or conversation history.

### For example:

### ✅ If your conversation is about depression research, the reconstructed conversation will continue treating the discussion as a research project.

Instead of assuming:

"You are depressed."

and switching into emotional-support mode.

Similarly, if your conversation focuses on software architecture, biomedical research, or mathematics, SUME attempts to reconstruct that reasoning trajectory.

### It is not intended to preserve factual personal information such as:

favorite food
birthday
hobbies
personal preferences

Its purpose is to preserve the reasoning model, not the personal factual information.
________________________________________
## Iterative Improvement

You can generate User Model Cards from different long-running conversations.

Think of each User Model Card as a snapshot of your reasoning model at a particular point in time.

### A practical workflow is:

UserModel
2025-07-06_homethink
2025-07-23_workthink
2025-08-22_workthink
2025-09-30_othernote
...

When starting a new conversation, load them chronologically.

This allows GPT to iteratively refine your reasoning model over time.

In theory, after enough iterations, the resulting User Model Card should increasingly approximate your own long-term reasoning style.
________________________________________
## Coming Next weekend

If you would like to initialize a more complete reasoning environment from the beginning, you can instead use the next project:

Shared Brain Initialization Protocol (SBIP)

While SUME extracts an existing reasoning model from  long-running research conversations, SBIP interactively constructs one from scratch by asking the user a series of questions.
