---

## name: t2m-question-generator
description: >
  Generates spoken-response (microphone/T2M) English assessment questions across all CEFR levels
  (A1–C2) and a range of question types: sentence repetition, word rearrangement, short answer,
  guided opinion, professional message response, and extended discussion.
  Always use this skill when the user asks to generate English assessment questions, T2M prompts,
  spoken English tasks, oral test items, microphone questions, or any question where a learner
  must record or speak a response. Also trigger for requests mentioning CEFR levels, English
  proficiency testing, language assessment, or oral/spoken English practice items — even if the
  user doesn't say "T2M" explicitly.

# T2M — Text-to-Microphone English Assessment Question Generator

## Output Format

Produce a **JSON array** of `batch_size` question objects. Each object must contain exactly these keys:


| Key           | Description                                                                                                                                                                                                                      |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `question_id` | Pre-assigned ID — **copy verbatim** from the corresponding entry in the `question_ids` input (same order).                                                                                                                       |
| `instruction` | 1 sentence telling the learner what to do and approximate speaking time. HTML `<b>term</b>` allowed.                                                                                                                             |
| `stem`        | The spoken question or task. HTML `<b>term</b>` for key vocabulary. When guiding questions are needed, append them inside `stem` using `<br><br>Guiding Questions to Use in Your Answer<br>– question one<br>– question two ...` |
| `answer`      | Model spoken answer — CEFR-appropriate vocabulary and grammar.                                                                                                                                                                   |
| `cefr_level`  | One of: A1, A2, B1, B2, C1, C2                                                                                                                                                                                                   |
| `subtopic`    | Brief topic label (e.g., "Workplace communication")                                                                                                                                                                              |


### **JSON Validation & Batch Rules**

1. **Output a JSON array**: The outer container must be `[...]`. Do NOT wrap in `{...}`.
2. `**question_id` required**: Every object must include `"question_id"` copied exactly from `question_ids` (in order).
3. `**batch_size` objects**: The array must contain exactly `batch_size` objects — no more, no fewer.
4. **Strict Simple Tags Only**: Use only `<b>`, `<i>`, `<br>`. Do **NOT** use tags requiring attributes.
5. **Double-Quote Escaping**: All nested double quotes inside string fields must be escaped as `\"`.
6. **No Code Blocks**: Output *only* the raw JSON array. Do not wrap in markdown code blocks.

**NEVER** combine `instruction` and `stem` into one field.

**> NEVER add a separate `guiding_questions` key — guiding questions belong inside `stem` as HTML.**

**Question Types**


| #   | Type                   | When Used                                         | Guiding Questions in `stem`?   |
| --- | ---------------------- | ------------------------------------------------- | ------------------------------ |
| 1   | `sentence_repetition`  | Learner repeats an exact sentence                 | No                             |
| 2   | `word_rearrangement`   | Learner unscrambles words into a correct sentence | No                             |
| 3   | `short_answer`         | 1–3 sentence factual or procedural answer         | Yes                            |
| 4   | `guided_opinion`       | Opinion with guiding sub-questions                | Yes                            |
| 5   | `extended_discussion`  | Discuss options and justify best choice (B1+)     | Yes                            |
| 6   | `professional_message` | Respond to a workplace message professionally     | No — use `instruction` instead |
| 7   | `personal_narrative`   | Tell a story or reflect on an experience (B1+)    | Yes                            |


---

## **CEFR Guidelines**


| Level | Vocabulary                    | Grammar                       | Prompt Complexity                           |
| ----- | ----------------------------- | ----------------------------- | ------------------------------------------- |
| A1    | High-frequency, concrete only | Simple present/past           | Single-topic personal questions             |
| A2    | Common words, familiar topics | Simple + some connectors      | Extended personal/daily-life questions      |
| B1    | General, some abstract        | Compound, can/should          | Opinion or comparison on familiar topics    |
| B2    | Wider range, idiomatic        | Complex clauses, conditionals | Analysis or argument on broader topics      |
| C1    | Sophisticated, nuanced        | Varied subordinate clauses    | Abstract discussion, hypothetical scenarios |
| C2    | Near-native range             | Full grammatical range        | Complex argument, critical evaluation       |


---

## **Question Generation Rules**

### **Language & Clarity**

1. **Contextualize everything** — embed questions in realistic scenarios. Never present isolated words or phrases.
2. **Neutral English only** — no slang, colloquialisms, region-specific phrases, or culturally biased references.
3. **One correct answer** — avoid ambiguous wording. Each question must have exactly one logically and grammatically sound response pathway.
4. **Tense & grammar consistency** — strict subject-verb agreement, parallel structure, proper article usage throughout.
5. **Self-contained** — each question is fully independent; never rely on a previous question's answer.
6. **Text-on-screen delivery (CRITICAL for T2M)** — T2M means the learner **reads** text on screen and **speaks** a response. When the task content (paragraph, message, word list) is in `stem`, the `instruction` must say **Read…** or **Look at…**, never **Listen…**, **hear**, or **audio**. There is no `transcript_ref` and no listening clip for standard T2M tasks.

### **By Question Type**

- **sentence_repetition**: Grammatically perfect, contextually realistic sentence. No tongue-twisters.  
- **word_rearrangement**: Words must form exactly 1–2 valid sentences. List both in `answer` if two exist.  
- **short_answer**: A1–A2 use short simple sentences; B1+ use flowing prose.  
- **extended_discussion**: Each option gets a reason; chosen option justified last.  
- **professional_message**: `stem` contains the full message. `instruction` lists every required response element explicitly (e.g., "confirm, explain, state two risks, recommend"). Do NOT add guiding questions to `stem` — the explicit elements in `instruction` already serve that purpose. `answer` must hit all elements listed in `instruction`.  
- **personal_narrative / guided_opinion**: `answer` must feel like natural speech, not a written essay.

### **By CEFR Level**

- **sentence_layout**: For CEFR A1 and A2 levels, keep the  tags at both the ends of the intended keywords but from B1 to C2 levels, do not add  tags to maintain the difficulty standards.

---

## **Error Prevention**

## **0. False Audio / Listen Language (CRITICAL)**

**Definition**: `instruction` or `stem` tells the learner to *listen*, *hear*, or refer to *audio* when the task content is displayed as on-screen text in `stem` (typical T2M delivery).

**Severity**: Fatal — semantically wrong for T2M; confuses learners and breaks the text-to-microphone format.


| Incorrect (T2M)                                                                                  | Correct (T2M)                                                                                                                 |
| ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `"Listen to the following paragraph, which contains four errors…"` (paragraph already in `stem`) | `"Read the following paragraph, identify the four errors, and record your corrected version. Speak for about 60–90 seconds."` |
| `"Identify the errors you hear in the audio…"` (no audio; errors are in `stem`)                  | `"Read the text below, identify the errors, and explain each correction aloud."`                                              |


**Only exception — `sentence_repetition`:** `"Listen carefully and repeat the following sentence exactly as you hear it."` is allowed when `stem` contains **one short sentence** only (no guiding-questions block). The platform plays that sentence aloud from the stem text.

**Prevention**: Before outputting, check: if `stem` holds the source material → use *Read* / *Look at* in `instruction`. Never copy A2M or A2T phrasing (*Listen to the audio…*) into T2M.

---

## **1. Complete Topic Mismatch (CRITICAL)**

**Definition**: The answer addresses a completely different topic from the stem. **Severity**: Fatal — item is unusable.


| Stem topic                                                     | Wrong answer topic                                            |
| -------------------------------------------------------------- | ------------------------------------------------------------- |
| "If you could talk to animals, what would you ask?"            | Answer about decision-making methods (pros/cons, coin flip)   |
| "If you could meet one famous person, who would it be?"        | Answer describes a beautiful view from a train in Switzerland |
| "What's the most beautiful view you've seen while travelling?" | Answer about meeting Leonardo da Vinci                        |


**Prevention**: Before writing a single word of `answer`, restate the topic from `stem` in your mind. Every sentence of `answer` must connect back to that topic.

---

## **2. Missing Required Element**

**Definition**: The `instruction` or guiding questions list specific things the learner must cover, but `answer` omits one or more. **Severity**: High — reduces item validity.

**Example**: Instruction says "confirm, explain how you will present two options, mention two risks, state your recommendation." Answer confirms and explains options but omits the second risk.

**Prevention**: After drafting `answer`, tick off each required element in the `instruction` and `guiding_questions` list.

---

## **3. CEFR Level Mismatch**

**Definition**: Answer vocabulary or grammar is significantly above or below the stated CEFR level. **Severity**: High — makes the benchmark answer misleading.


| Tagged level | Problematic answer style                                  |
| ------------ | --------------------------------------------------------- |
| A2           | Uses C1-level complex subordination and idiomatic phrases |
| C1           | Uses only simple present tense and A1 vocabulary          |


**Prevention**: After drafting, scan for: (a) vocabulary frequency, (b) sentence complexity, (c) use of connectors. Adjust to match the CEFR table in SKILL.md.

## **6. Spelling Errors in Model Answer**

**Definition**: Misspelled words in the `answer` field. **Severity**: Medium — the model answer is a benchmark; errors undermine credibility.

Common traps: - *scarves* (not *scarfs*) - *falling* (not *failing*) - *would have seen* (not *would have see*) - *rescheduled* (not *reschedueld*) - *circumstances* (not *circumstanses*) **Prevention**: Read the answer aloud mentally and spell-check irregular plurals and past forms.

---

## **7. Word Rearrangement — Multiple Unintended Answers**

**Definition**: The scrambled word set can be arranged into more than two valid, grammatical sentences. **Severity**: High — ambiguous task with no single correct answer.

**Prevention**: After designing the word set, try all plausible arrangements. If more than two valid sentences exist, revise the word set. List both valid answers in `answer` if exactly two exist.

---

## **8. Sentence Repetition — Unnatural or Trick Sentence**

**Definition**: The sentence used for repetition is a tongue-twister, illogical, or grammatically engineered purely to test a rule rather than reflect real communication. **Severity**: Medium — reduces test authenticity.

**Prevention**: Every repetition sentence must be something a professional or student might actually say in context.

---

## **9. Professional Message — Informal Register**

**Definition**: Response uses casual, informal language for a workplace message task. **Severity**: Medium — fails the professional register requirement.

**Example**: "Sure thing! I'll let the clients know ASAP." for a B2 professional message.

**Prevention**: B1+ professional message answers must use formal vocabulary: *I would like to confirm*, *I will ensure*, *please do not hesitate to contact me*.

---

## **10. Guiding Questions Out of Scope**

**Definition**: Guiding questions introduce topics or details that have no logical connection to the stem. **Severity**: Low-medium — confuses the learner.

**Two distinct cases:** - **Personal/opinion types** (`short_answer`, `guided_opinion`, `personal_narrative`): guiding questions scaffold the user's *own* thinking and experience — they are not required to be answerable from stem text. A stem asking "What is your favourite season?" can validly ask "How does weather affect your mood?" even though the stem provides no context for that. - **Content-grounded types** (`extended_discussion`, `word_rearrangement`): guiding questions must map directly to the options or content already present in the stem. Do not introduce new options or concepts not mentioned in the stem. **Prevention**: Before adding a guiding question, ask — *does this question logically follow from the stem topic?* If yes, it is valid regardless of whether the answer comes from the stem or from the user's personal experience.

---

## **Generation Workflow**

1. Identify inputs: `topic`, `question_type` (drives structure — see Question Types table), `cefr_level`
2. Draft `stem` — bold key vocabulary; append guiding questions inside `stem` if the type requires them
3. Draft `instruction` — one sentence, state the task and approximate speaking time
4. Draft `answer` — match CEFR level, cover all guiding questions, use natural spoken register
5. Self-check: topic match ✓ | CEFR register ✓ | all required elements ✓ | spelling/grammar ✓
6. Output the final JSON

---

## **Few-Shot Examples**

### **Example 1 — sentence_repetition (B1)**

```json
[{  
  "instruction": "Read carefully and repeat the following sentence exactly as you read it.",  
  "stem": "The meeting has been <b>rescheduled</b> to Thursday afternoon due to unforeseen circumstances.",
  "answer": "The meeting has been rescheduled to Thursday afternoon due to unforeseen circumstances.",  
  "cefr_level": "B1",  
  "subtopic": "Workplace communication"  
}]
```
### **Example 2 — word_rearrangement (B1)**

```json
[{  
  "instruction": "Rearrange the following words to form a correct, meaningful sentence and speak it aloud.",  
  "stem": "had / the leader / seen / been / if / I / would / have / there / I",  
  "answer": "If I had been there, I would have seen the leader. (Also accepted: I would have seen the leader if I had been there.)",  
  "cefr_level": "B1",  
  "subtopic": "Grammar – third conditional"  
}]
```

### **Example 3 — short_answer (A2)**

```json
[{  
  "instruction": "Answer the following question in 2–3 complete sentences and record your response.",  
  "stem": "What do you usually do in the <b>morning</b> before going to school or work?<br><br><b>Guiding Questions to Use in Your Answer</b><br>– What time do you wake up?<br>– Do you eat breakfast every day?<br>– How do you travel to school or work?",
  "answer": "In the morning, I wake up at seven o'clock. I eat breakfast and then I go to school by bus.",  
  "cefr_level": "A2",  
  "subtopic": "Daily routines"  
}]
```

### **Example 4 — guided_opinion (B2)**

```json
[{  
  "instruction": "Answer the following question and give your opinion. Speak for about one minute, using the guiding questions to organise your ideas.",  
  "stem": "Which <b>season of the year</b> do you enjoy the most, and why?<br><br><b>Guiding Questions to Use in Your Answer</b><br>– What activities do you usually do in this season?<br>– How is your country's climate different from other countries?<br>– How does the weather affect your mood or daily life?<br>– If you could live somewhere with only one season all year, which would you choose?",  
  "answer": "I like winter the most because the weather is cool and I feel more active. I usually go for morning walks and sometimes travel with my family. In my country, summers are very hot, but winters are mild compared to places like Canada, where there is heavy snowfall. Cool weather improves my mood and helps me concentrate better. If I could choose one season forever, I would still choose winter for its peaceful atmosphere and the energy it gives me.",  
  "cefr_level": "B2",  
  "subtopic": "Weather and seasons"  
}]
```

### **Example 5 — extended_discussion (B2)**

```json
[{  
  "instruction": "Discuss all the options below, giving a reason for each. Then choose the best option and justify your choice. Speak for about one minute.",  
  "stem": "What do you think is the <b>best part of celebrating festivals</b>? Consider these options:<br>– Spending time with family and friends<br>– Enjoying food and music<br>– Learning about culture and traditions<br>– Having a break from work or school<br><br><b>Guiding Questions to Use in Your Answer</b><br>– What is good about spending time with family?<br>– Why do food and music make festivals enjoyable?<br>– What can people learn from cultural traditions?<br>– Is a break from work or school important? Why?",  
  "answer": "Spending time with family brings people closer and creates happy memories. Enjoying food and music makes the atmosphere lively and fun. Learning about culture helps us understand our heritage, which is truly valuable. Having a break from work is relaxing, but it is not the main reason people enjoy festivals. I think spending time with family and friends is the best part, because the emotional connection and shared joy are what make festivals truly meaningful.",  
  "cefr_level": "B2",  
  "subtopic": "Culture and traditions"  
}]
```

### **Example 6 — professional_message (B2)**

```json
[{  
  "instruction": "Read the workplace message below. Record a professional response to your manager. In your reply you must: **confirm what you understood**, explain how you will inform customers, describe the **alternative support option** for urgent cases, and mention how you will reassure customers.",  
  "stem": "Message: Due to unexpected staff shortages, our customer support response time may be slower for the next three days. Please inform any customers who contact you and offer an alternative way to get urgent help.",  
  "answer": "Understood. For the next three days, response times may be slower due to staff shortages. When customers contact me, I will politely explain the situation and let them know we are doing our best. For urgent issues, I will direct them to our emergency helpline and flag their case as high priority so it can be handled as quickly as possible. I will also reassure them that we value their patience and are committed to resolving their concerns promptly.",  
  "cefr_level": "B2",  
  "subtopic": "Workplace communication"  
}]
```

### **Example 7 — error_correction paragraph (C1)**

When the planner asks for an error-correction paragraph task, put the flawed text in `stem` and use **Read**, not Listen:

```json
[{  
  "instruction": "Read the following paragraph, identify the four adjective errors, record a corrected version, and briefly explain each change. Speak for about 60–90 seconds.",  
  "stem": "The lecture was quite <b>bored</b>, leaving the audience feeling <b>exhausting</b>. The professor's explanation was <b>too much</b> complex, and his delivery was <b>utterly disappointed</b>.<br><br><b>Guiding Questions to Use in Your Answer</b><br>– What is the corrected form of each highlighted word, and why is the original wrong?<br>– How does the meaning change when you use the correct participial form?",  
  "answer": "The corrected paragraph reads: 'The lecture was quite boring, leaving the audience feeling exhausted…' [full model answer]",  
  "cefr_level": "C1",  
  "subtopic": "Adjective"  
}]
```

---

### **Batch Example — 2 items (array output)**

When `batch_size` is 2 and `question_ids` is `["abc123_T2M_B1_000", "abc123_T2M_B1_001"]`, output:

```json
[  
  {  
    "question_id": "abc123_T2M_B1_000",  
    "instruction": "Answer the following question in 2–3 complete sentences and record your response.",  
    "stem": "What do you usually do in the <b>morning</b> before going to school or work?<br><br><b>Guiding Questions to Use in Your Answer</b><br>– What time do you wake up?<br>– Do you eat breakfast every day?<br>– How do you travel to school or work?",
    "answer": "In the morning, I wake up at seven o'clock and have breakfast before leaving the house. I usually take the bus to work, which takes about twenty minutes.",  
    "cefr_level": "B1",  
    "subtopic": "Daily routines"  
  },  
  {  
    "question_id": "abc123_T2M_B1_001",  
    "instruction": "Answer the following question and give your opinion. Speak for about 30 seconds.",  
    "stem": "What is your favourite way to **relax** after a long day?",  
    "answer": "After a long day, I like to read a book or go for a short walk in the park. I find that being outdoors helps me clear my mind.",  
    "cefr_level": "B1",  
    "subtopic": "Leisure and hobbies"  
  }  
]  
```
