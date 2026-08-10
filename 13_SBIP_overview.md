# ⭐ SBIP｜Shared Brain Initialization Protocol

**SBIP is a GPT designed to build a long-term AI collaboration model.**

[GPTs link here](https://chatgpt.com/g/g-6a506a9ac13881919befe669df62e869-shared-brain-initialization-protocol-sbip)

**After completing the GPT's questions, obtain the AI-Readable Model Card (Boot Configuration) and paste it into any new chat window. It can be used with GPT and Gemini. Do not use it with Claude, for reasons explained in the IRR documentation.**

It is not intended for personality analysis or psychological assessment. 

Instead, it progressively builds a **Stable User Model** through interaction, allowing the AI to understand your capabilities, decision-making patterns, reasoning style, and collaboration preferences while reducing the need to repeatedly explain yourself when starting a new conversation.

### 🧠 SBIP primarily builds four models

* **Capability Model** — What you are good at and what your core capabilities are
* **Decision Model** — How you typically evaluate situations and make decisions
* **Reasoning Model** — How you classify, abstract, build, and revise models
* **Preference Model** — How you prefer the AI to collaborate with you

It also naturally accumulates **Shared Vocabulary** and **Shared Definitions** throughout the conversation.

### 🔄 How to Use SBIP

After opening SBIP, **simply answer the GPT's questions**.

SBIP progressively builds the model using high-information-gain questions and displays the current progress for:

> Capability / Decision / Reasoning / Preference / Overall

Once all four models reach **90% or higher**, SBIP enters:

> **Shared Brain Sync — ONLINE**

At this point, initialization stops and SBIP begins using the completed Stable User Model for **Context-first** collaboration.

**Note:** GPT may occasionally omit the progress percentages from its response. If this happens, enter:

> **"No progress percentage was displayed. Please return to test mode."**

### 📦 Model Cards

Once the initialization is complete, you can request two types of Model Cards.

**Human-Readable Model Card (RPG Panel)**
Designed to provide a quick overview of the current reasoning style, capabilities, and collaboration preferences. This version is primarily intended for presentation and entertainment.

Enter:

> **"Please output the Human-Readable Model Card (RPG Panel)."**

**AI-Readable Model Card (Boot Configuration)**
Designed to transfer the Stable User Model to a new AI conversation, allowing the new AI to perform Validation and enter Shared Brain Synchronization without rebuilding the model from scratch.

Enter:

> **"Please output the AI-Readable Model Card (Boot Configuration)."**

[👉 This is an example of my SBIP output.](./14_SBIP_example.md)

### ♻️ Continuous Synchronization

The Shared Brain is not a permanent personality description. It is a **continuously updateable Working Model**.

When new evidence appears:

* If it is consistent with the current model → maintain Shared Brain Sync
* If it conflicts with the current model → **update only the affected components**
* Do not rebuild the entire model

In other words:

> **Initialization → Shared Brain Sync → Synchronization Update**

Rather than making the AI relearn the user from scratch every time.
