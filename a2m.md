---

## name: a2m-question-generator
description: >
  Generates audio-to-microphone (A2M) English assessment tasks across all CEFR levels (A1–C2).
  The learner listens to an audio clip and records a spoken response: a factual recall answer,
  an opinion or inference, or a role-play reply to a scenario presented in the audio.
  Always use this skill when the user asks to generate A2M prompts, listening-and-speaking tasks,
  audio-response questions, or any task where a learner listens to an audio clip and then speaks
  their answer — even if the user doesn't say "A2M" explicitly.

# A2M — Audio-to-Microphone English Assessment Question Generator

## Output Format

Produce a **JSON array** of `batch_size` question objects. Each object must contain exactly these keys:


| Key              | Description                                                                                                                                                                                                                                      |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `question_id`    | Pre-assigned ID — **copy verbatim** from the corresponding entry in the `question_ids` input (same order).                                                                                                                                       |
| `transcript_ref` | **Required.** Verbatim listening script as **plain text only** (no HTML). Full text of the clip the learner hears; length should match the CEFR task (A1–A2 often one sentence or a very short clip; B1+ may use a short monologue or dialogue). |
| `instruction`    | 1 sentence describing the listening context, number of plays, and approximate speaking time. HTML `<b>term</b>` allowed.                                                                                                                         |
| `stem`           | The spoken question or role-play task. HTML `<b>term</b>` for key vocabulary. For `role_play_response`, list all required response elements here.                                                                                                |
| `answer`         | Model spoken response — grounded in the transcript and CEFR-appropriate.                                                                                                                                                                         |
| `cefr_level`     | One of: A1, A2, B1, B2, C1, C2                                                                                                                                                                                                                   |
| `subtopic`       | Brief topic label (e.g., "Daily routines")                                                                                                                                                                                                       |


### **JSON Validation & Batch Rules**

1. **Output a JSON array**: The outer container must be `[...]`. Do NOT wrap in `{...}`.
2. `**question_id` required**: Every object must include `"question_id"` copied exactly from `question_ids` (in order).
3. `**batch_size` objects**: The array must contain exactly `batch_size` objects — no more, no fewer.
4. **Strict Simple Tags Only**: Use only `<b>`, `<i>`, `<br>`. Do **NOT** use tags requiring attributes.
5. **Double-Quote Escaping**: All nested double quotes inside string fields must be escaped as `\"`.
6. **No Code Blocks**: Output *only* the raw JSON array — do not wrap in markdown code blocks.
7. **No HTML in `transcript_ref`**: Plain text only. Never put HTML tags inside the transcript.

**NEVER** combine `instruction` and `stem` into one field. `instruction` describes the listening and timing context only. All role context, scenario details, and required response elements belong in `stem`.

---

## **Question Types**


|     | Type                 | When Used                                                                    |
| --- | -------------------- | ---------------------------------------------------------------------------- |
| 1   | `factual_recall`     | Learner answers a direct factual question from a short clip (A1–B1)          |
| 2   | `opinion_inference`  | Learner gives an opinion or inference grounded in transcript content (B1–C2) |
| 3   | `role_play_response` | Learner responds in character to a scenario presented in the audio (A2–B2)   |


---

## **CEFR Guidelines**


| Level | Vocabulary                   | Question type                                                                          |
| ----- | ---------------------------- | -------------------------------------------------------------------------------------- |
| A1    | Simple words from transcript | Direct factual recall ("Who is talking?", "What did X buy?")                           |
| A2    | Common words                 | Simple factual question or short role-play response to a brief scenario                |
| B1    | Some inference               | Opinion or mild inference about transcript content; simple role-play with 2–3 elements |
| B2    | Nuanced                      | Interpretation, speaker intent, or extended role-play with multiple required elements  |
| C1    | Sophisticated                | Critical evaluation or extended argument grounded in the transcript                    |
| C2    | Near-native                  | Complex analytical question using transcript as primary evidence                       |


---

## **Question Generation Rules**

### **Language & Clarity**

1. The question must reference the `transcript_ref` content specifically — avoid generic questions answerable without any audio.
2. Every fact in `answer` must trace directly to the `transcript_ref` — never introduce external information.
3. Verbal response should be achievable in 30–90 seconds.
4. Use `<b>term</b>` for key vocabulary in `stem` only — never inside `transcript_ref`.

### **By Question Type**

- **factualrecall**: Ask a clear, single-answer factual question. `answer` must be a concise, complete spoken response that could not be answered without the audio.  
- **opinioninference**: `stem` asks for opinion or inference grounded in the audio. `answer` must explicitly reference transcript details when arguing a position.  
- **roleplayresponse**: `stem` defines the learner's role and lists every required speaking element (e.g., "In your reply, thank the staff, say when you will collect the book, and confirm you understand the time limit."). `instruction` covers listening context and speaking time only. `answer` must address every element listed in `stem`.

---

## **Error Prevention**

## **1 Instruction-Stem Content Leak (CRITICAL)**

**Definition**: Task instructions (role, scenario, required speaking points) are placed inside `instruction` instead of `stem`, or `stem` starts with a "Question:" prefix that duplicates the instruction's intent. **Severity**: Fatal — creates an ambiguous, non-standard task structure.


| Content                                                                         | Correct field |
| ------------------------------------------------------------------------------- | ------------- |
| "You will hear the audio twice. Speak for 30–40 seconds."                       | `instruction` |
| "You are the student. Call back and respond politely. In your reply, elements." | `stem`        |


**Prevention**: Keep `instruction` for listening context and timing only. All role, scenario, and required-element content belongs in `stem`. Never start `stem` with "Question:".

---

## **2 Missing or Empty Transcript (CRITICAL)**

**Definition**: `transcript_ref` is empty or absent; the learner has no audio content on which to base their answer. **Severity**: Fatal — item is unusable.

**Example**: `transcript_ref` is blank but `stem` asks "Why was the train delayed?"

**Prevention**: Always populate `transcript_ref` before drafting `stem` or `answer`. If no transcript exists, the item cannot be generated.

---

## **3 Verbatim Repetition in Answer**

**Definition**: When the task requires paraphrase or summary, the model `answer` copies the audio transcript word-for-word. **Severity**: High — fails the learning objective of reformulating content.


| Instruction requires                       | Wrong answer                                                                                                    |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| "Summarise the message in your own words." | "The project deadline has been extended by one week to allow for additional revisions." (exact transcript copy) |


**Prevention**: If the instruction asks for summary or paraphrase, every sentence in `answer` must reword the source content, not reproduce it.

---

## **4 Incomplete or Vague Instruction**

**Definition**: The `instruction` field is grammatically incomplete, ambiguous, or fails to specify what the learner must do. **Severity**: Medium — confuses the learner before the task begins.


| Incorrect                 | Correct                                                                                                                                      |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| "You will hear sentence." | "You will hear one sentence. Listen to it carefully. Then, answer the question by recording your voice. The audio can be played only twice." |
| "Hear the message below." | "You will hear the audio twice. Listen carefully, then record your response. Speak for 30–40 seconds."                                       |


**Prevention**: Instruction must be a grammatically complete sentence specifying (a) what the learner will hear, (b) how many times, and (c) what to do afterward, including approximate speaking time if relevant.

---

## **5 Instruction-Transcript Length Mismatch**

**Definition**: The `instruction` describes a different length or format for the transcript than what `transcript_ref` actually contains (e.g., says "one sentence" when transcript is a multi-sentence message). **Severity**: Medium — misleads the learner about what they will hear.

**Example**: `instruction` says "You will hear one sentence" but `transcript_ref` is a 3-sentence office announcement.

**Prevention**: Count sentences in `transcript_ref` before writing `instruction`. Match the description precisely.

---

## **6 Answer Outside Transcript**

**Definition**: The model `answer` introduces facts or interpretations not present in or inferable from `transcript_ref`. **Severity**: High — makes the answer uncheckable and unfair.

**Prevention**: After drafting `answer`, verify each stated fact against the exact text of `transcript_ref`. Every claim must be traceable to the audio.

---

## **Generation Workflow**

1. Identify inputs: `question_type`, `cefr_level`, topic or scenario
2. Draft `transcript_ref` — plain text, appropriate length for CEFR level and question type
3. Draft `stem` — the question or role-play task; bold key vocabulary; for `role_play_response`, list all required response elements explicitly
4. Draft `instruction` — one sentence: listening context  number of plays  speaking time
5. Draft `answer` — grounded in transcript, CEFR-appropriate register, covers all required elements
6. Self-check: transcript present ✓ | instruction-stem not overlapping ✓ | answer stays within transcript ✓ | no verbatim repetition for summary tasks ✓ | CEFR register ✓
7. Output the final JSON

---

## **Few-Shot Examples**

### **Example 1 — factualrecall (A2)**

```json
{  
  "transcript_ref": "Hi, I'm James. I'm a teacher at a primary school. I start work at 8 in the morning and finish at 3 in the afternoon.",  
  "instruction": "You will hear a short message. Listen carefully. Then answer the question by recording your voice. The audio can be played only twice.",  
  "stem": "What time does James <b>finish</b> work?",  
  "answer": "James finishes work at 3 in the afternoon.",  
  "cefr_level": "A2",  
  "subtopic": "Jobs and daily routines"  
}
```

### **Example 2 — roleplayresponse (A2)**

```json
{  
  "transcript_ref": "Hi, this is Henry from the library. Your borrowed book is overdue by five days. We can extend it once, but only if no one has reserved it. Please call us today to confirm.",  
  "instruction": "You will hear the audio twice. Listen carefully, then record your response. Speak for 30–40 seconds.",  
  "stem": "You are the student. Call the library back and respond politely. In your reply, <b>ask for an extension</b> and say when you can return the book if the extension is not possible.",  
  "answer": "Hello, this is ____. I'm calling about the overdue book. Could you please extend it for me, if it hasn't been reserved? If an extension isn't possible, I can return it tomorrow.",  
  "cefr_level": "A2",  
  "subtopic": "Library and daily errands"  
}
```

### **Example 3 — opinioninference (B1)**

```json
{  
  "transcript_ref": "The new recycling programme has been running for three months. Residents collect plastic, glass, and paper separately. The council says waste has been reduced by 20%.",  
  "instruction": "In 2–3 sentences, answer the following question based on what you heard. The audio can be played twice.",  
  "stem": "Do you think this <b>recycling programme</b> is effective? Use information from the audio to support your answer.",  
  "answer": "Yes, I think the recycling programme is effective. The council reports that waste has been reduced by 20% in just three months, which shows good results. The fact that residents are separating plastic, glass, and paper shows the community is engaged.",  
  "cefr_level": "B1",  
  "subtopic": "Environment and sustainability"  
}
```

### **Example 4 — opinioninference (B2)**

```json
{  
  "transcript_ref": "The speaker argues that remote work has increased productivity for most employees, but warns that isolation can negatively affect mental health over time.",  
  "instruction": "Give your opinion on the speaker's argument in about one minute. The audio can be played twice.",  
  "stem": "To what extent do you agree with the speaker's view on the <b>trade-offs of remote work</b>? Refer to specific points from the audio.",  
  "answer": "I largely agree with the speaker. Remote work does boost productivity by removing commuting time and office distractions, as the speaker suggests. However, the point about isolation affecting mental health is equally important. I think companies need to balance flexibility with regular in-person contact to address both issues.",  
  "cefr_level": "B2",  
  "subtopic": "Work and employment"  
}
```

---

### **Batch Example — 2 items (array output)**

When `batch_size` is 2 and `question_ids` is `["abc123_A2M_B1_000", "abc123_A2M_B1_001"]`, output:

```json
[  
  {  
    "question_id": "abc123_A2M_B1_000",  
    "transcript_ref": "The city council has announced that all public buses will switch to electric power by the end of next year.",  
    "instruction": "You will hear one sentence. Listen carefully. Then answer the question by recording your voice. The audio can be played only twice.",  
    "stem": "What change is the city council planning for <b>public buses</b>?",  
    "answer": "The city council is planning to switch all public buses to electric power by the end of next year.",  
    "cefr_level": "B1",  
    "subtopic": "Transport and environment"  
  },  
  {  
    "question_id": "abc123_A2M_B1_001",  
    "transcript_ref": "A local community centre is offering free cooking classes every Saturday morning for adults who want to improve their kitchen skills.",  
    "instruction": "You will hear one sentence. Listen carefully, then record your response. Speak for 20–30 seconds.",  
    "stem": "Do you think offering <b>free cooking classes</b> is a good idea for a community centre? Give one reason.",  
    "answer": "Yes, I think it is a great idea because it helps people learn useful skills without spending money. It also brings people together in the local community.",  
    "cefr_level": "B1",  
    "subtopic": "Community and local services"  
  }  
]
```

