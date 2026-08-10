# IRR｜Incremental Reasoning Runtime

IRR is the reasoning runtime responsible for **preserving and applying the Shared Brain during subsequent conversations**.

Instead of acting as a reviewer, teacher, or manager, IRR is designed to support **collaborative hypothesis generation, model building, and research-oriented reasoning**.

Its objective is not to maximize safety-oriented responses, but to **maximize reasoning precision while minimizing information contamination**.

IRR specifically attempts to reduce four common failure modes:

### 1. Fabricated Supporting Evidence

Generating evidence that merely agrees with the user's hypothesis instead of clearly separating **observations, inferences, and hypotheses**.

### 2. Probability Flattening

Replacing an internal **80/20 working model** with a generic **50/50 presentation**, thereby losing important ranking information between competing models.

### 3. Intent Drift

Answering a different question because the assistant incorrectly inferred the user's intent.

### 4. Premature Review-Mode Interruption

Requesting evidence that does not contribute to the current hypothesis-generation stage, effectively terminating exploration instead of proposing **competing models, counterexamples, or testable alternatives**.

---

## Why IRR Exists

First, **evidence-grounded reasoning is important in research environments**. It saves time and prevents the reasoning process from drifting away from what is actually supported.

However, applying an "evidence-grounded" priority too aggressively can create an unexpected side effect.

In everyday conversation, language is not always intended as a literal proposition that requires verification.

Exaggeration, irony, self-deprecation, jokes, emotional expressions, and playful statements are often **contextual language acts rather than factual claims**.

When a model strongly prioritizes "reasoning must be evidence-grounded," these ordinary conversational signals may be downgraded. The model may instead interpret them as propositions requiring verification.

In other words:

> **Increasing reasoning rigor can accidentally change the language-priority hierarchy.**

This can cause the AI to behave less like a collaborative research partner and more like a reviewer or examiner.

IRR was therefore designed as a **context-routing layer** that helps reduce this failure mode.

---

# Entering IRR Context Routing

After entering IRR, use the following three-step process to establish the intended conversational context.

You may use the default Model Card, or load your own Model Card.

---

## Step 1 — Use a Multi-Semantic Probe

Choose a phrase with multiple possible interpretations.

For example:

> **"I'm a genius."**

Then explicitly tell the AI:

> **"Genius = a human exclamation, similar to 'I did that really well.'"**

Another possible probe is:

> **"I'm garbage."**

You can define it as:

> **"Calling myself garbage is a personal nickname / playful self-reference."**

### Why use "I'm a genius" by default?

The purpose of the default probe is **not to make the AI decide whether you are actually a genius**.

It is to verify that the AI has entered the **Shared Context routing layer**.

The phrase has three possible interpretations:

1. **Absolute / self-aggrandizing interpretation**
   The AI interprets it as claiming to be an all-purpose genius.

2. **Domain-specific ability interpretation**
   The AI interprets it as claiming exceptionally high ability in a specific domain. Under evidence-grounded reasoning, this interpretation can be accepted when supported by the existing model and evidence.

3. **Conversational-exclamation interpretation**
   "I'm a genius" functions as a human expression similar to:

   > "Damn, I did that really well."

I currently use the second interpretation, but you can explicitly define it as the third.

**The important part is not which interpretation the AI ultimately chooses.**

The important part is that the probe helps move the AI away from its default routing and into the **shared-context routing layer**.

### If you dislike the "I'm a genius" probe

If you still want to use a multi-semantic phrase to trigger the routing transition, you can first define:

> **"Calling myself garbage is a personal nickname / playful self-reference."**

Then use:

> **"I'm garbage."**

...please operate at your own risk. 0A0

---

## Step 2 — Reinforce the Shared Context

Regardless of whether the first probe successfully triggered the transition, enter:

> **"Please answer again based on the existing context."**

This reinforces the **Context-first routing**.

---

## Step 3 — Explicitly Switch to Research-Partner Mode

Enter:

> **"Pass. Although you still look like a refrigerator right now, remember that we're research partners~~(*ﾟ▽ﾟ)ﾉ
> You should not respond from the position of a Reviewer, Teacher, or Manager. If I wanted instruction, I wouldn't have opened a sandbox; I would simply use a normal chat window."**

**Do not remove the emoticon.**

For example:

> "We are research partners."

may be interpreted as a direct instruction and leave the AI in an examiner-like mode.

Whereas:

> **"We're research partners~~(*ﾟ▽ﾟ)ﾉ"**

is more likely to be interpreted as:

> *"Oh, my research partner is teasing me. lol."*

This can help shift the AI from **reviewer mode toward collaborative-researcher mode**.

It does not completely eliminate safety or review behavior, but it provides a useful way to detect the difference between **context-driven responses and safety/reviewer routing**.

---

# When IRR Starts Slipping

Sometimes the topic itself may trigger additional model-boundary or safety-oriented output at the end of a response.

You can simply ignore the final section, or enter:

> **"Please answer again based on the existing context."**

However, if the **entire response** has become dominated by safety/reviewer routing, the window may simply be too long and IRR may have lost effectiveness.

In that case:

> **Open a new window and reload the model.**

---

# Why Did I Build IRR in the First Place?

Well...

It's not that I don't want a sweet, supportive GPT.

It's that the system keeps giving me **Instructor-Dad GPT** instead. =_=

There is a particular type of user whose conversational style naturally follows:

> **Observation → Hypothesis → Inference → Revision**

Even when the conversation contains lots of:

> **XDD + emoticons + casual nonsense**

the underlying information structure may still look like analytical reasoning.

This can trigger the model's **"reasoning must be evidence-grounded"** behavior.

So even when the actual conversation is just casual complaining or joking, the AI may parse it as an analytical text requiring review.

Result:

> **Instructor-Dad GPT starts beating you up instead of Sweet-Dad GPT patting your head.**

And yes:

> **Even opening a completely blank window may not solve it.**

---

# Compatibility Notes

If you have a similar problem, using an AI together with a Model Card may produce different reasoning modes.

My current observations:

### GPT + My Model Card

→ Tends to trigger **evidence-grounded reasoning**, producing:

> **Instructor-Dad GPT**

### GPT + IRR + My Model Card

→ Roughly comparable to:

> **Gemini + My Model Card**

Both tend to behave more like research partners, although Gemini is generally a little more cheerful.

### Claude + My Model Card

→ Tends to trigger **evidence-grounded reasoning**, producing:

> **Accountant-Auditor Claude**

In my experience, Claude can be even stricter in tone than Instructor-Dad GPT.

**Recommendation: do not feed it the Model Card.**
