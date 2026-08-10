⭐ 軍師

可以。這篇我會把它改成**真正符合你現在 repo 定位的英文文章**：不是單純「How to install Whisper」，而是把重點放在 **AI + Whisper + 文件／逐字稿處理 workflow**。

核心會保留你原本真正有價值的東西：

> **影片／音訊 → Whisper → 分段 → 低資源環境穩定化 → 合併 → 講義校正 → AI 整理 → 可重複使用的知識抽取流程**

而不是把它寫成 Whisper 使用教學而已。

---

# Building a High-Accuracy Knowledge Extraction Workflow Using AI and Whisper

## From Long-Form Training Videos to Searchable, Reusable Knowledge

This workflow was developed to convert long-form training videos and audio recordings into structured, reusable knowledge.

The basic idea is simple:

> **Video / Audio → Speech-to-Text → Transcript Cleaning → Source-Based Correction → AI-Assisted Knowledge Extraction**

However, the difficult part is not simply generating a transcript.

The real challenge is preserving enough information throughout the process that the resulting knowledge remains reliable and useful without requiring the user to watch the entire recording again.

This workflow therefore separates the process into several stages:

```text
Video / Audio
      ↓
Whisper Transcription
      ↓
Transcript Segmentation
      ↓
AI-Assisted Transcript Processing
      ↓
Source / Lecture Material Correction
      ↓
Structured Summary
      ↓
Human-Readable Transcript
      ↓
Reusable Knowledge
```

The workflow was designed around a practical constraint:

> **AI is very good at processing large amounts of text, but it should not be allowed to silently discard information simply because that information appears repetitive or unimportant.**

---

# 1. The Problem

Long training videos contain several different types of information at the same time.

For example:

* Formal lecture material
* Explanations
* Demonstrations
* Software operations
* Questions and answers
* Configuration details
* Exceptions
* Time-dependent conditions
* Troubleshooting information

A normal summary can easily remove information that later turns out to be important.

For example, a lecturer may casually mention:

> "This setting usually stays unchanged."

That sentence might look unimportant during summarization.

But if it describes a configuration parameter that determines whether a system works, removing it can make the resulting documentation incomplete.

Therefore, the objective is not simply:

> **Make the transcript shorter.**

The objective is:

> **Convert noisy speech into reusable information while preserving operationally important details.**

---

# 2. Local Whisper Installation

The original workflow uses OpenAI Whisper locally.

The advantage of local processing is that the audio and video do not need to be uploaded to an external transcription service.

This can be useful for research, education, and institutional environments where the source material may contain sensitive information. 

---

## System Requirements

The original environment was:

* Windows 10 / 11
* 64-bit system
* CPU supported
* NVIDIA GPU optional
* Administrator privileges required during installation

The installation workflow is:

```text
Miniconda
    ↓
Chocolatey
    ↓
FFmpeg
    ↓
OpenAI Whisper
    ↓
PyTorch
    ↓
Test transcription
```

---

# 3. Install Miniconda

Install the Windows 64-bit version of Miniconda using the graphical installer.

Recommended installation options:

* Create shortcuts
* Register Miniconda3 as the default Python
* Clear package cache upon completion

Do **not** add Miniconda3 directly to PATH during installation.

After installation, open:

```text
Anaconda PowerShell Prompt
```

with administrator privileges. 

---

# 4. Install Chocolatey and FFmpeg

Chocolatey is used as the package manager for FFmpeg.

Install Chocolatey from the Anaconda PowerShell Prompt, then reopen the administrator shell and verify:

```bash
choco -v
```

Install FFmpeg:

```bash
choco install ffmpeg -y
```

Verify:

```bash
ffmpeg -version
```

FFmpeg allows Whisper to process video files directly. 

---

# 5. Install Whisper

Install Whisper:

```bash
pip install -U openai-whisper
```

For a CPU-based installation, PyTorch can also be installed with:

```bash
pip install torch torchvision torchaudio
```

A basic transcription command is:

```bash
whisper input.mp4 --language Chinese --model medium --output_format txt
```

The important parameters are:

```text
--language Chinese
--model medium
--output_format txt
```

The `medium` model provides a practical balance between transcription quality and processing speed.

Other model sizes can be selected depending on available hardware:

```text
small   → faster
medium  → general-purpose choice
large   → higher accuracy, higher resource requirements
```



---

# 6. The First Important Discovery: Model Size Is Not Always the Solution

During actual testing, the available computer was not powerful enough to reliably process long recordings using the `medium` or `large` models.

The practical solution was therefore to use:

```text
small + CPU
```

However, this introduced another problem.

With long recordings, Whisper could sometimes become unstable.

Observed behaviors included:

* The model becoming influenced by preceding context
* Repeating a "safe" output
* Producing repeated phrases
* Errors becoming more noticeable after language or speaker changes

Importantly, the observed correlation between speaker changes and transcription errors does not necessarily mean that speaker changes themselves caused the problem.

The practical response was to change the workflow rather than simply changing the model.

---

# 7. Segment the Video Before Transcription

Instead of processing a long video as one continuous input:

> **Split the video into shorter segments first.**

In the tested environment, approximately five-minute segments produced a more reliable result than ten-minute segments.

For example:

```bash
ffmpeg -ss 00:00:00 -t 00:05:00 -i textBC.mp4 -c copy part01.mp4
ffmpeg -ss 00:05:00 -t 00:05:00 -i textBC.mp4 -c copy part02.mp4
ffmpeg -ss 00:10:00 -t 00:05:00 -i textBC.mp4 -c copy part03.mp4
```

Continue until the entire recording has been segmented.

The important idea is not that five minutes is universally optimal.

It was the experimentally observed stable segment size for the hardware and workflow being used.

---

# 8. Use Stable Whisper Parameters

For the tested environment, the following parameters were retained:

```bash
--model small
--condition_on_previous_text False
--verbose False
```

For example:

```bash
whisper part01.mp4 \
--language Chinese \
--model small \
--output_format txt \
--condition_on_previous_text False \
--verbose False
```

The same process can then be applied to every segment.

This produces:

```text
part01.txt
part02.txt
part03.txt
...
part17.txt
```

The key design decision here is:

> **Do not rely on a larger model alone to solve a long-context failure mode.**

Instead, change the structure of the input.

---

# 9. Merge the Transcripts

After transcription, the individual text files need to be combined.

A simple PowerShell concatenation approach was initially considered, but formatting problems made a small Python merging script more convenient.

The resulting workflow becomes:

```text
part01.txt
part02.txt
part03.txt
...
part17.txt
       ↓
Python merge script
       ↓
Complete transcript
```

This is a small example of an important general principle:

> **When a manual operation is repeated often enough, turn it into a script.**

---

# 10. Do Not Treat the Transcript as the Final Knowledge Product

At this point, we have a transcript.

But a raw transcript is not necessarily useful documentation.

The next stage is to combine:

```text
Lecture materials
+
Complete transcript
```

The lecture materials provide the authoritative structural reference.

The transcript provides information that may not appear in the formal materials, including:

* Spoken explanations
* Questions and answers
* Demonstrations
* Operational details
* Practical warnings
* Informal clarifications

The two sources therefore have different roles.

---

# 11. Source-Based Transcript Correction

The correction rules are intentionally conservative.

### If lecture materials are available

Use the official lecture materials as the primary source for terminology and structure.

The transcript should be treated as supporting information rather than automatically correct information.

Only information that can be reasonably confirmed should be incorporated into the structured material.

If a section involves:

* Demonstrations
* Screen contents
* Commands
* Visual operations

and the transcript alone cannot establish what happened, mark it:

```text
[Review video]
```

Do not invent missing information merely to make the documentation complete. 

---

# 12. Separate Summary from Transcript Preservation

The workflow produces two different products.

## A. Structured Summary

The summary is designed for rapid review.

For example:

```text
1. What is this lecture about?

2. Core systems and relationships

3. Important concepts

4. Operational constraints

5. Important exceptions

6. Items requiring video review
```

The summary can be generated from all transcript segments at once when appropriate, but it should be treated as a **quick browsing layer**, not as the authoritative record.

---

## B. Human-Readable Transcript

The detailed transcript serves a different purpose.

Its job is to allow the user to read the recording without having to watch the entire video again.

The rules are therefore much stricter.

---

# 13. Low-Noise Transcript Correction

The detailed transcript should preserve the original sentence structure as much as possible.

The allowed transformations are intentionally limited to:

* Removing obvious filler words
* Removing pure repetitions
* Correcting obvious transcription errors
* Improving punctuation
* Marking visual operations as requiring video review

For example:

> "嗯、啊、那個、就是說"

can be removed when they contain no information.

But operational conditions, exceptions, timing constraints, and caveats should remain.

The goal is:

> **Not a rewritten document.**

Instead:

> **A low-noise corrected version of the machine-generated transcript.**

The target is approximately **95% or higher information retention**. 

---

# 14. Why Aggressive Summarization Is Dangerous

Large language models naturally tend to optimize for readability.

This can create a problem.

Suppose the original transcript contains:

```text
A
B
C
D
E
```

A normal summarization workflow may produce:

```text
A + B + C → Summary
D + E → Summary
```

That may look better.

But if `D` contains an exception condition and `E` contains a configuration requirement, the resulting document is no longer operationally equivalent to the original.

Therefore, transcript correction and summarization should be treated as **different tasks**.

### Transcript correction

```text
Original
 ↓
Remove noise
 ↓
Preserve information
```

### Summary

```text
Original
 ↓
Extract important concepts
 ↓
Compress information
```

They should not use the same processing rules.

---

# 15. Long Context Is a Practical Constraint

The problem is not necessarily that an AI system cannot understand the entire recording.

A sufficiently long transcript can simply exceed the practical amount of text that can be processed reliably in one operation.

As the input becomes larger, information compression becomes increasingly likely.

Therefore:

> **Split first, process second.**

For example, the tested workflow recommends processing approximately five minutes of transcript at a time when detailed transcript correction is required. 

This gives the system smaller, more manageable units while preserving the original information.

---

# 16. A Strict Correction Prompt

A reusable instruction can be structured around the following rules:

> Compare the output sentence by sentence against the original transcript.
>
> Convert each original sentence into one cleaned sentence.
>
> Do not merge sentences.
>
> Do not reorder sentences.
>
> Do not summarize.
>
> Do not abstract.
>
> Do not remove content because it appears unimportant.
>
> Remove only obvious speech disfluencies and pure repetition.
>
> Correct obvious transcription errors.
>
> Preserve all conditions, timing information, exceptions, and operational details.
>
> If an operation depends on the visual screen content, mark it as:
>
> `[Review video]`

This forces the AI into a **denoising mode** rather than a summarization mode.

---

# 17. Use Screenshots as an Auxiliary Source

Some information cannot be reliably recovered from audio alone.

For example:

* Button locations
* Interface settings
* Software configuration
* Commands displayed on screen
* Visual relationships
* Demonstration results

Therefore, screenshots from the video can be supplied together with the transcript.

The recommended workflow is:

```text
Original Video
      ↓
Whisper Transcript
      ↓
Screenshots
      ↓
AI Processing
      ↓
Detailed Transcript
      ↓
Review Unclear Sections
```

However, the transcript remains the primary source for the detailed textual record.

Images and PDFs serve as supporting evidence.

---

# 18. Example of Structured Knowledge Extraction

A training session might contain a large amount of conversational material.

Instead of preserving everything in the summary, the system can extract the actual conceptual structure.

For example:

### 1. What is the lecture about?

How each organization can transform its own data into OMOP CDM and use it within a TRE environment without physically transferring the original data.

### 2. Core architecture

The lecture emphasizes decentralized data access:

* Data remains with the Data Partner / Data Custodian.
* The platform sends queries.
* Only aggregated results are returned.
* Raw data is not centralized or downloaded.

### 3. System roles

The transcript and lecture materials can then be reconciled to distinguish:

**RQUEST**

* Cohort discovery
* Aggregated queries
* Researcher-facing functions

**INSIGHT**

* Collection management
* Data ingestion
* Metadata configuration
* Data-manager-facing functions

If the lecture materials establish that Link has been integrated into INSIGHT, the transcript can be corrected accordingly. 

This is where AI becomes useful:

> **The AI is not inventing the knowledge.**

It is organizing information from multiple source representations into a form that humans can review.

---

# 19. The Complete Workflow

The complete architecture can therefore be represented as:

```text
                Training Video
                      │
                      ▼
              ┌──────────────┐
              │    FFmpeg    │
              │  Segmentation│
              └──────────────┘
                      │
                      ▼
              Short Video Segments
                      │
                      ▼
              ┌──────────────┐
              │   Whisper    │
              │ Transcription│
              └──────────────┘
                      │
                      ▼
              Raw Transcript
                      │
                      ▼
              Python Merge
                      │
                      ▼
           Complete Raw Transcript
                 /           \
                /             \
               ▼               ▼
       Lecture Materials     Screenshots
               │               │
               └───────┬───────┘
                       ▼
               AI-Assisted Processing
                    /          \
                   /            \
                  ▼              ▼
          Structured Summary   Detailed
                              Transcript
                                  │
                                  ▼
                           Human Review
                                  │
                                  ▼
                         Reusable Knowledge
```

---

# 20. The Core Design Principle

The most important part of this workflow is not Whisper itself.

Whisper is simply the first transformation layer:

> **Audio → Text**

The larger system is:

> **Unstructured media → structured information**

The workflow deliberately assigns different tasks to different components:

| Component         | Role                                                    |
| ----------------- | ------------------------------------------------------- |
| FFmpeg            | Segment long recordings                                 |
| Whisper           | Speech-to-text                                          |
| Python            | File processing and workflow automation                 |
| Lecture materials | Primary terminology / structural reference              |
| Screenshots       | Visual verification                                     |
| AI                | Extraction, organization, correction, and summarization |
| Human             | Final review and interpretation                         |

This division prevents one component from being responsible for the entire information pipeline.

---

# 21. Why This Workflow Works Better Than Simply Asking AI to Summarize a Video

A naïve workflow is:

```text
Video
 ↓
AI
 ↓
Summary
```

The proposed workflow is:

```text
Video
 ↓
Controlled transcription
 ↓
Segmentation
 ↓
Source correction
 ↓
Information-preserving transcript
 ↓
Structured extraction
 ↓
Human review
```

The difference is important.

The first approach asks AI to perform transcription, interpretation, compression, and documentation simultaneously.

The second approach creates intermediate representations that can be inspected at each stage.

This makes it much easier to determine **where information was lost** if the final result is incorrect.

---

# 22. Final Architecture

The workflow can ultimately be summarized as:

> **Capture → Transcribe → Segment → Preserve → Correct → Extract → Structure → Review**

The goal is not to create the shortest possible summary.

The goal is to create a knowledge artifact that is:

* Searchable
* Reusable
* Reviewable
* Source-grounded
* Less dependent on repeatedly watching the original recording

The most important design principle is therefore:

> **Use AI to reduce the amount of human attention required, not to eliminate human control over information.**

This turns a long training video from a one-time viewing experience into a reusable knowledge base.

