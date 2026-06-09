---
name: a2t-question-generator

description: >
  generates audio-to-text (A2T) English assessment tasks across all CEFR levels (A1–C2).
  The learner listens to an audio clip and produces a written response: filling in blanks
  with specific words from the audio, or writing a summary, description, opinion, or argument.
  Always use this skill when the user asks to generate A2T prompts, audio-based writing tasks,
  listening-and-writing assessments, or any task where a learner listens to an audio clip and
  then writes their answer — even if the user doesn't say "A2T" explicitly.

---
# A2T — Audio-to-Text English Assessment Question Generator

## Output Format

Produce a **JSON array** of `batch_size` question objects. Each object must contain exactly these keys:


| Key              | Description                                                                                                                                              |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `question_id`    | Pre-assigned ID — **copy verbatim** from the corresponding entry in the `question_ids` input (same order).                                               |
| `transcript_ref` | **Required.** Verbatim listening script as **plain text only** (no HTML). Full text of the clip the learner hears; length should match the CEFR task.    |
| `instruction`    | 1 sentence specifying the expected response format, length, and any formatting rules; HTML (`<b>term</b>`) permitted.                                    |
| `stem`           | The writing task description; HTML (`<b>term</b>`) permitted for key vocabulary. For `fill_in_blank`, contains numbered blank sentences.                 |
| `answer`         | Model written response at the target CEFR level (comma-separated words for `fill_in_blank`; paragraph prose for `paragraph_response`).                   |
| `explanation`    | At least 2 lines (or ≥ 100 characters) explaining why the answer is correct and level-appropriate. For `fill_in_blank`, explain each blank individually. |
| `cefr_level`     | One of: A1, A2, B1, B2, C1, C2                                                                                                                           |
| `subtopic`       | Brief topic label (e.g., "Daily routines")                                                                                                               |


### **JSON Validation & Batch Rules**

1. **Output a JSON array**: The outer container must be `[...]`. Do NOT wrap in `{...}`.
2. **question_id required**: Every object must include `"question_id"` copied exactly from `question_ids` (in order).
3. **batch_size objects**: The array must contain exactly `batch_size` objects — no more, no fewer.
4. **Strict Simple Tags Only**: Use only `<b>`, `<i>`, `<br>`. Do **NOT** use tags requiring attributes.
5. **Double-Quote Escaping**: All nested double quotes inside string fields must be escaped as `\"`.
6. **No Code Blocks**: Output *only* the raw JSON array — do not wrap in markdown code blocks.
7. **No HTML in `transcript_ref`**: Plain text only. Never put HTML tags inside the transcript.

**NEVER** combine `instruction` and `stem` into one field. `instruction` specifies the response format and any formatting rules. `stem` contains the actual writing task description or blank sentences.

---

## **Question Types**


| #   | Type                 | When Used                                                                    | Answer format         |
| --- | -------------------- | ---------------------------------------------------------------------------- | --------------------- |
| 1   | `fill_in_blank`      | Learner writes 1–2 words per numbered blank; answers comma-separated (A1–B1) | Comma-separated words |
| 2   | `paragraph_response` | Learner writes a short summary, description, opinion, or argument (A2–C2)    | Prose paragraph(s)    |


---

## **CEFR Guidelines**

### **fill_in_blank**


| Level | Expected response         | Task type                                                           |
| ----- | ------------------------- | ------------------------------------------------------------------- |
| A1    | 1 word per blank          | Identify simple facts (name, number, colour) from a very short clip |
| A2    | 1–2 words per blank       | Fill details from a short monologue: time, place, person, activity  |
| B1    | 1–2 words, some inference | Fill details including paraphrased or lightly inferred information  |


### **paragraph_response**


| Level | Expected response                 | Task type                                                |
| ----- | --------------------------------- | -------------------------------------------------------- |
| A2    | 3–5 sentences                     | Describe or summarise a simple fact from audio           |
| B1    | 1 short paragraph (5–7 sentences) | Summarise, compare, or give a brief opinion              |
| B2    | 1–2 paragraphs                    | Analyse, argue, or evaluate with examples from the audio |
| C1    | 2–3 paragraphs                    | Extended argument, detailed analysis                     |
| C2    | 3+ paragraphs                     | Critical evaluation, complex multi-angle argument        |


---

## **Generation Rules**

### **All Types**

1. The task must be achievable from a single listen — all required information must be present in `transcript_ref`.
2. Do not require external knowledge beyond the transcript.
3. Use `<b>term</b>` for key vocabulary that anchors the writing task to the audio.
4. The `answer` must match the CEFR level in vocabulary range, grammar complexity, and response length.

### **fill_in_blank**

1. Number each blank sequentially (1. 2. 3. ...).
2. Each blank must correspond to exactly one clearly stated fact in the transcript.
3. `instruction` must specify: (a) write one or two words, (b) comma-and-space separated format, (c) no extra punctuation, (d) capitalisation rule.
4. `answer` must list answers in the exact same order as the blanks — one answer per blank, comma-separated.
5. `explanation` must address each blank individually, citing the exact transcript phrase that provides the answer.

### **paragraph_response**

1. State the expected response length clearly in `instruction`.
2. Make the task type explicit in `stem`: summarise / argue / describe / evaluate.
3. `answer` must feel like natural written prose, not a bullet list.
4. `explanation` must identify the features that make the answer level-appropriate (vocabulary range, grammar, connectors, length).

---

## **Error Prevention**

## **1. Incorrect Answer in Scheme (CRITICAL)**

**Definition**: The answer key provides the wrong word for a given blank — a word not stated in the transcript for that position. **Severity**: Fatal — makes the answer key untrustworthy.


| Blank question                            | Transcript says                    | Wrong key answer |
| ----------------------------------------- | ---------------------------------- | ---------------- |
| "Her best friend is called ____________." | "My best friend is called Sophie." | "bus"            |


**Prevention**: For every blank, copy the exact word or phrase from the transcript that fills it. Verify one-to-one mapping between blank number and the corresponding transcript phrase.

---

## **2. Multiple Answer Key Errors (CRITICAL)**

**Definition**: The answer key has compound errors: duplicate answers for distinct blanks, wrong proper nouns (name, title), or spelling errors. **Severity**: Fatal — multiple incorrect entries make the key unreliable.

Common traps:

- Same answer appearing for two different blanks (e.g., "piano" listed for two distinct questions)  
- Wrong title: "Mr. Brown" when transcript says "Mrs. Brown"  
- Spelling error in key: "consert" instead of "concert"

**Prevention**: Number each answer and verify it maps to the correct blank. Re-read all proper nouns and spell-check any irregular words before outputting.

---

## **3. Verbatim Repetition in Written Response**

**Definition**: For `paragraph_response` tasks requiring summary or paraphrase, the model `answer` copies the transcript verbatim. **Severity**: High — fails the learning objective of reformulating content.

**Prevention**: If the instruction asks for summary or paraphrase, every sentence in `answer` must reword the source content rather than reproduce it.

---

## **4. Blank–Transcript Mismatch**

**Definition**: A blank sentence asks about information not present in the transcript, or uses vocabulary the transcript never includes. **Severity**: High — makes the blank unanswerable or ambiguous.

**Prevention**: Draft all blank sentences directly from the transcript content. Do not introduce vocabulary or facts that are absent from `transcript_ref`.

---

## **Generation Workflow**

1. Identify inputs: `question_type`, `cefr_level`, topic or transcript
2. Draft `transcript_ref` — plain text, appropriate length and content for CEFR level and type
3. **For `fill_in_blank`**: Draft numbered blank sentences from transcript facts → draft `instruction` with formatting rules → draft `answer` as comma-separated words in blank order → draft `explanation` per blank citing transcript
4. **For `paragraph_response`**: Draft `stem` as a clear writing task (summarise / argue / describe) → draft `instruction` specifying length and genre → draft `answer` as prose at CEFR level → draft `explanation` citing level-appropriate features
5. Self-check: every blank/answer traces to transcript ✓ | instruction-stem not overlapping ✓ | answer length and register match CEFR ✓ | no verbatim copy for summary tasks ✓
6. Output the final JSON

---

## **Few-Shot Examples**
The JSON code fences below are examples for documentation only. Actual generated output must be a raw JSON array without code fences.

### **Example 1 — fill_in_blank (A2)**

```json
[{  
  "transcript_ref": "Hi, I'm Sarah. I started learning English last year, and now I love reading books. My favourite book is about animals. I usually go to the library three times a week. My best friend is called Anna, and we go together. We like to sit near the window in the library, where it's quiet. Next year, I want to join a reading club at school.",  
  "instruction": "For each question, type your answer in the textbox below. Write one or two words. Write only the answers. Separate the answers with a comma and a space. Do not use any extra punctuation. Do not capitalise unnecessarily.",  
  "stem": "1. Sarah started learning ____________ last year.\n2. Her favourite book is about ____________.\n3. She goes to the library ____________ times a week.\n4. Sarah's best friend is called ____________.\n5. They like to sit near the ____________ in the library.\n6. Next year, Sarah wants to join a ____________ club.",  
  "answer": "English, animals, three, Anna, window, reading",  
  "explanation": "English – She says she started learning English last year. animals – Her favourite book is about animals. three – She goes to the library three times a week. Anna – She names her best friend. window – She says they like to sit near the window. reading – She wants to join a reading club.",  
  "cefr_level": "A2",  
  "subtopic": "Hobbies and daily routines"  
}]
```

### **Example 2 — paragraph_response (B1)**

```json
[{  
  "transcript_ref": "A new study shows that students who exercise regularly tend to perform better in exams. Researchers believe physical activity improves concentration and reduces stress.",  
  "instruction": "Write a short paragraph of 5–7 sentences.",  
  "stem": "Summarise the findings of the study and explain why researchers believe <b>exercise</b> helps students.",  
  "answer": "According to the study, students who exercise regularly tend to do better in exams. Researchers believe that physical activity has two main benefits for students. First, it improves their concentration, which helps them focus during study sessions. Second, it reduces stress, which can otherwise affect exam performance. These findings suggest that regular exercise should be part of a student's routine.",  
  "explanation": "This answer summarises both findings from the audio and explains the reasoning given by researchers. It uses B1-appropriate paragraph structure with connectors such as 'first' and 'second'. The vocabulary ('concentration', 'physical activity', 'performance') matches B1 range. The response length of 5 sentences fits the instruction.",  
  "cefr_level": "B1",  
  "subtopic": "Health and education"  
}]
```

### **Example 3 — paragraph_response (B2)**

```json
[{  
  "transcript_ref": "Economists are divided on the impact of a universal basic income. Some argue it would reduce poverty and provide financial security, while others warn it could reduce work incentives and strain public finances.",  
  "instruction": "Write one to two paragraphs (approximately 100–150 words).",  
  "stem": "Present the arguments for and against a <b>universal basic income</b> as described in the audio, and give your own view.",  
  "answer": "The audio presents two opposing views on universal basic income. Supporters argue that it would reduce poverty and give people financial security, which could improve quality of life for those in low-income situations. Critics, however, warn that it might reduce people's motivation to work and could put significant pressure on government budgets.\n\nIn my view, the potential benefits of reducing poverty are compelling, but the concerns about work incentives should not be dismissed. A carefully designed scheme with appropriate conditions might address both sets of concerns, though this would require further research and political consensus.",  
  "explanation": "This answer clearly presents both sides of the debate using the audio content and adds a personal opinion with supporting reasoning, as the task requires. The vocabulary ('financial security', 'work incentives', 'political consensus') and grammar (conditionals, complex noun phrases) are appropriate for B2. The two-paragraph structure and word count (~130 words) match the instruction.",  
  "cefr_level": "B2",  
  "subtopic": "Economics and social policy"  
}]
```

---

### **Batch Example — 2 items (array output)**

When `batch_size` is 2 and `question_ids` is `["abc123_A2T_B1_000", "abc123_A2T_B1_001"]`, output:

```json
[  
  {  
    "question_id": "abc123_A2T_B1_000",  
    "transcript_ref": "Hi, I'm Sarah. I started learning English last year, and now I love reading books. I usually go to the library three times a week. My best friend is called Anna, and we go together.",  
    "instruction": "Write one or two words per blank. Separate answers with a comma and a space. Do not capitalise unnecessarily.",  
    "stem": "1. Sarah started learning ____________ last year.\n2. She goes to the library ____________ times a week.\n3. Her best friend is called ____________.",  
    "answer": "English, three, Anna",  
    "explanation": "English – She says she started learning English last year. three – She goes to the library three times a week. Anna – She names her best friend as Anna.",  
    "cefr_level": "B1",  
    "subtopic": "Hobbies and daily routines"  
  },  
  {  
    "question_id": "abc123_A2T_B1_001",  
    "transcript_ref": "A new study shows that students who exercise regularly tend to perform better in exams. Researchers believe physical activity improves concentration and reduces stress.",  
    "instruction": "Write a short paragraph of 5–7 sentences.",  
    "stem": "Summarise the findings of the study and explain why researchers believe <b>exercise</b> helps students.",  
    "answer": "According to the study, students who exercise regularly tend to do better in exams. Researchers believe that physical activity has two main benefits for students. First, it improves their concentration, which helps them focus during study sessions. Second, it reduces stress, which can otherwise affect exam performance. These findings suggest that regular exercise should be part of a student's routine.",  
    "explanation": "This answer summarises both key findings and explains the researchers' reasoning. It uses B1-appropriate connectors ('first', 'second') and vocabulary ('concentration', 'physical activity'). The 5-sentence length fits the instruction.",  
    "cefr_level": "B1",  
    "subtopic": "Health and education"  
  }  
]  
```

