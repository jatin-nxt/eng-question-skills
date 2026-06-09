---
name: mcq-question-generator
description: >
  Generates text-based multiple-choice (MCQ) English assessment questions across all CEFR levels
  (A1–C2) and a range of question types: sentence inference, factual comprehension, vocabulary in
  context, conversation completion, grammar application, text completion, passage comprehension,
  sentence correction, and matching.
  Always use this skill when the user asks to generate MCQ items, reading comprehension questions,
  grammar MCQs, vocabulary questions, text-completion tasks, or any question where a learner reads
  a text stimulus and selects the correct answer — even if the user doesn't say "MCQ" explicitly.
  This skill is for text-only MCQ. For audio-based MCQ, use the A2MCQ skill instead.
---

# MCQ — Text-Based Multiple-Choice English Assessment Question Generator

## Output Format

Produce a **JSON object** with exactly these keys:

| Key | Description |
|-----|-------------|
| `instruction` | 1 sentence telling the learner what to do; HTML (`<b>term</b>`) permitted. |
| `stem` | The full task content: source text (sentence, passage, dialogue, or paragraph with blanks) followed by the question. Use `<br><br>` to separate the source text from the question within the same field. HTML `<b>term</b>` for key vocabulary. |
| `options` | List of exactly `num_options` answer strings (default 4 unless otherwise specified). |
| `answer` | The single correct option — must appear verbatim in `options`. |
| `explanation` | At least 2 lines (or ≥ 100 characters) explaining why the answer is correct and why each distractor is wrong. |
| `cefr_level` | One of: A1, A2, B1, B2, C1, C2 |
| `subtopic` | Brief topic label (e.g., "Vocabulary in context") |

> ### JSON Validation & HTML Escaping Rules
> 1. **Strict Simple Tags Only**: Use only `<b>`, `<i>`, `<br>`. Do **NOT** use tags requiring attributes.
> 2. **Double-Quote Escaping**: All nested double quotes inside string fields must be escaped as `\"`.
> 3. **No Code Blocks**: Output *only* the raw JSON object — do not wrap in markdown code blocks.
> 4. **Never bold options**: `<b>` tags belong only in `instruction` or `stem`, never inside any option string.

> **NEVER** combine `instruction` and `stem` into one field.
> `instruction` tells the learner what to do (e.g., "Read the sentence carefully and answer the question."). `stem` contains the source text and the specific question about it. Do NOT repeat task verbs from `instruction` inside `stem`.

---

## Question Types

| # | Type | Stimulus | Cognitive skill | CEFR range |
|---|------|----------|-----------------|------------|
| 1 | `sentence_inference` | Single sentence | Infer what the sentence suggests or implies | A2–B2 |
| 2 | `factual_comprehension` | Single sentence or short statement | Identify the correct statement from the given information | A1–B1 |
| 3 | `vocabulary_in_context` | Single sentence with a bolded key word | Identify the meaning of the word as used in the sentence | A2–C1 |
| 4 | `passage_comprehension` | Short passage (3–8 sentences) | Answer a factual or inferential question about the passage | A2–C1 |
| 5 | `conversation_completion` | Dialogue with one missing turn | Choose the most appropriate reply for the missing speaker | B1–C1 |
| 6 | `text_completion` | Sentence or paragraph with one or two blanks | Choose the word, phrase, or connector pair that best fills the blank(s) | B1–C2 |
| 7 | `grammar_application` | One or more sentences | Apply a grammar rule: identify which sentence can/cannot be transformed, or choose the correct transformation | B1–C1 |
| 8 | `sentence_correction` | Sentence with a bolded portion | Choose the option that correctly improves the highlighted text | B2–C2 |
| 9 | `matching` | A person's description or requirements | Choose the option that best matches those requirements | A2–B2 |

---

## CEFR Guidelines

| Level | Vocabulary | Stem complexity | Distractor difficulty |
|-------|-----------|-----------------|----------------------|
| A1 | High-frequency words only | Short, direct factual question from a simple statement | Clearly wrong but plausible at first glance |
| A2 | Common words | Simple inference or comprehension of explicit detail | Require reading the stem carefully to eliminate |
| B1 | General vocabulary | Inference, vocabulary, or grammar identification | Contain plausible but incorrect details |
| B2 | Wider range | Interpretation, nuanced inference, or text completion | Subtle semantic or structural differences from the correct answer |
| C1 | Sophisticated | Critical evaluation, complex correction, or advanced grammar | Very close paraphrases; require careful analysis to distinguish |
| C2 | Near-native | Discourse-level analysis or complex argument evaluation | Require full understanding of nuance to eliminate |

---

## Formatting Rules (mandatory)
1. **Case rule**: If the stem ends with a blank (`____________`), options start with **lowercase**. If the stem is a complete question, options start with **uppercase**.
2. **Comma spacing**: Correct — "cats, dogs, and birds". Incorrect — "cats,dogs,and birds".
3. **Punctuation**: If options complete a sentence stem, they must end with a **period**. If options are standalone answers (e.g., a word or short phrase), no period.
4. **Bold tags**: Use `<b>term</b>` inside `stem` only for: (a) a key vocabulary word being tested, (b) the bolded portion in `sentence_correction`, or (c) passage labels (e.g., `<b>TEXT</b>:`). Never bold options.
5. **Explanation length**: At least 2 full lines OR ≥ 100 characters. State why the correct answer is right first, then address each wrong option individually with a specific reason.
6. **Distractors**: Each distractor must be plausible (a learner who didn't read carefully might choose it) but unambiguously wrong once the stem is fully understood.
7. **Options unique**: All options must be distinct — no two options can be the same or near-identical paraphrases.
8. **No aggregate distractors**: Never use "All of the above" or "None of the above" as a distractor unless every option is explicitly confirmed or denied by the stem.
9. **Stem structure for source-text types**: Always use `<br><br>` to separate the source text from the question within `stem`. Never run them together in one paragraph.

---

## Question Generation Rules

### Language & Clarity
1. Every question must be self-contained — the learner must be able to answer without any prior context or external knowledge.
2. Use neutral, formal English throughout. No slang, idioms specific to one region, or culturally biased content.
3. One and only one correct answer. Before finalising options, defend each distractor as clearly wrong.
4. Tense and grammar in `instruction`, `stem`, `options`, and `explanation` must all be consistent.

### By Question Type
- **sentence_inference**: The inference must be clearly supported by the sentence's content — not speculation. The correct option must be the most logical conclusion from the stated action or situation.
- **factual_comprehension**: The correct option must be a restatement (not just a copy) of a fact in the source sentence. Distractors introduce plausible but false details (wrong time, wrong place, cancelled event).
- **vocabulary_in_context**: Bold the word being tested in `stem`. The correct option must match the word's meaning as used in that specific sentence, not its most common or dictionary-primary meaning if different in context.
- **passage_comprehension**: Place the passage under a `Passage:` or `<b>TEXT</b>:` label, then add `<br><br>` before the question. The question must be answerable from the passage alone.
- **conversation_completion**: The missing turn (Person B) must respond directly to what Person A said. Use `Person A:` / `Person B:` labels. The correct option must be natural, grammatically correct, and directly responsive. Distractors may be grammatically wrong, irrelevant, or unresponsive.
- **text_completion**: For single-blank items, all options must be the same part of speech. For two-blank items (pair selection), all pairs must follow the same grammatical pattern — only the correct pair fits both blanks simultaneously.
- **grammar_application**: State the rule clearly in `instruction` (e.g., "choose the one sentence that can be converted to a natural passive voice"). The explanation must name the grammatical reason for each distractor being eliminated.
- **sentence_correction**: Bold the portion being corrected in `stem`. The correct option must fix the structural error without changing the original meaning. Distractors may be grammatically wrong, partially correct, or meaning-altering.
- **matching**: Present the person's requirements before the options. The correct option must satisfy all stated requirements. Distractors may satisfy some but not all, or satisfy none.

---

## Error Prevention

## 1. Answer–Explanation Mismatch (CRITICAL)
**Definition**: The `explanation` discusses a different question, passage, people, or scenario than what the `stem` contains — typically caused by copy-pasting an explanation from another item.
**Severity**: Fatal — the explanation is actively misleading.

| Stem is about | Wrong explanation refers to |
|--------------|----------------------------|
| "The committee's decision was unanimous..." | "Olivia feeling settled and happy in the village" |
| "Where does Maria go with her dog on weekends?" | "She feels settled and happy" (from a different item) |

**Prevention**: After writing the explanation, re-read the first sentence of `stem`. Every proper noun, scenario, and topic in the explanation must match exactly. A mismatch means an explanation was pasted from a different item — replace it entirely.

---

## 2. Incorrect Answer Key (CRITICAL)
**Definition**: The `answer` field does not match the logically or grammatically correct option, or it contradicts the `explanation`.
**Severity**: Fatal — makes the item unusable.

| Stem or explanation supports | Wrong answer key |
|-----------------------------|-----------------|
| Ethan wants to meet people and practise English informally; "Conversation Café" clearly fits | "Film Club – Watch classic movies together" |
| Explanation confirms Maria goes to the park with her dog | "Film Club – Watch classic movies together" |

**Prevention**: After setting `answer`, verify (a) it appears verbatim in `options`, (b) the `explanation` says it is correct, and (c) no other option can be equally defended.

---

## 3. Multiple Acceptable Answers
**Definition**: Two or more options are both grammatically correct or logically valid, but only one is marked as `answer`.
**Severity**: High — the item is ambiguous; any learner who chose the unmarked correct option would be unfairly penalised.

Common cases:
- Both active-voice and passive-voice rewrites are correct: "was damaged" and "got damaged" are both natural
- Two passive transformations preserve meaning equally: "The receipt was issued to the student after the fee was collected." and "After the fee was collected, the receipt was issued to the student." are both valid
- Two conversation responses are both grammatically natural and contextually appropriate

**Prevention**: Before finalising options, try to independently argue for each option. If more than one option can be genuinely defended, revise the distractors or narrow the question so that only one option is unambiguously correct.

---

## 4. Grammar Error in Explanation
**Definition**: The `explanation` contains a grammatical error (subject-verb agreement, article usage, tense, etc.).
**Severity**: Medium — the explanation is the benchmark justification; errors undermine its credibility.

Common traps:
- *"so they fits correctly"* (not *"they fit correctly"*)
- *"it sound slightly formal"* (not *"it sounds slightly formal"*)
- *"this answer are correct"* (not *"this answer is correct"*)

**Prevention**: Re-read every sentence of the explanation specifically for subject-verb agreement and verb form consistency before outputting.

---

## 5. Instruction Error (Grammar, Spelling, or Verb Form)
**Definition**: The `instruction` field contains a grammatical error, misspelling, or wrong verb form that makes it unclear or incorrect.
**Severity**: Medium — a flawed instruction undermines the item's authority and may confuse the learner.

| Incorrect | Correct |
|-----------|---------|
| "Read the test and choose…" | "Read the **text** and choose…" |
| "Read the text when decide which programme…" | "Read the text and decide which programme…" |
| "chose the correct option" | "**Choose** the correct option" (`instruction` must use the base/imperative form) |
| "programe" | "**programme**" |

**Prevention**: Re-read `instruction` specifically for (a) correct imperative verb form (base form, not past tense), (b) spelling, and (c) grammatical completeness. The instruction must be a complete, natural sentence.

---

## 6. Poor Item Construction — Unnatural Explanation Phrasing
**Definition**: The `explanation` contains incomplete, unnatural, or logically incoherent sentences that fail to justify the answer clearly.
**Severity**: Medium — a confusing explanation defeats the item's learning purpose.

| Incorrect | Correct |
|-----------|---------|
| "lunch happened later reaching the top" | "lunch happened **after** reaching the top" |
| "it may sound slightly formal **than** usual" | "it may sound slightly **more** formal than usual" |

**Prevention**: After drafting the explanation, read each sentence aloud. If a sentence sounds incomplete or unnatural, rewrite it as a full, clear English sentence before outputting.

---

## Generation Workflow

1. Identify inputs: `question_type`, `cefr_level`, topic or source text
2. Draft `stem` — write the source text (sentence, passage, dialogue, or paragraph with blanks), add `<br><br>`, then write the specific question; bold key vocabulary as appropriate for the question type
3. Draft `instruction` — one sentence in imperative form, stating exactly what the learner must do
4. Draft `options` — one correct answer + `num_options - 1` plausible distractors; verify no two options can both be defended as correct
5. Set `answer` — copy the correct option verbatim; verify it matches the `stem`
6. Draft `explanation` — correct answer first (with reason), then each wrong option with a specific reason
7. Self-check: explanation refers to the same item as `stem` ✓ | `answer` consistent with explanation ✓ | no two options both defensible ✓ | `instruction` in correct imperative form ✓ | explanation grammatically correct ✓
8. Output the final JSON

---

## Few-Shot Examples

### Example 1 — sentence_inference (B1)
```json
{
  "instruction": "Read the sentence carefully and answer the question.",
  "stem": "Farida packed a light sweater before leaving for the museum trip.<br><br>What does this suggest about Farida?",
  "options": [
    "She expected the weather or indoor temperature to be cool.",
    "She planned to buy clothes during the trip.",
    "She had forgotten where the museum was.",
    "She wanted to lend the sweater to the guide."
  ],
  "answer": "She expected the weather or indoor temperature to be cool.",
  "explanation": "Correct answer: Packing a light sweater before a trip is preparation for cool conditions — either outdoors or inside the museum. 'She planned to buy clothes during the trip' is incorrect because the sentence says nothing about shopping. 'She had forgotten where the museum was' is incorrect because packing a sweater is unrelated to knowing the location. 'She wanted to lend the sweater to the guide' is incorrect because there is no mention of giving it to anyone else.",
  "cefr_level": "B1",
  "subtopic": "Sentence inference"
}
```

### Example 2 — factual_comprehension (A2)
```json
{
  "instruction": "Read the sentence carefully and choose the correct statement.",
  "stem": "The health camp starts at 10 a.m. on the ground floor.<br><br>Which statement is correct?",
  "options": [
    "It starts at night on the ground floor.",
    "It starts in the evening on the ground floor.",
    "It starts in the morning on the ground floor.",
    "It has been cancelled."
  ],
  "answer": "It starts in the morning on the ground floor.",
  "explanation": "Correct answer: 10 a.m. means morning, so the camp starts in the morning on the ground floor. 'It starts at night' is incorrect because 10 a.m. is not at night. 'It starts in the evening' is incorrect because 10 a.m. is not in the evening. 'It has been cancelled' is incorrect because the sentence says the camp starts at 10 a.m., not that it is cancelled.",
  "cefr_level": "A2",
  "subtopic": "Factual comprehension"
}
```

### Example 3 — vocabulary_in_context (B1)
```json
{
  "instruction": "Read the sentence carefully and choose the meaning of the highlighted word.",
  "stem": "The instructions were <b>explicit</b>.<br><br>What does \"explicit\" mean in this sentence?",
  "options": [
    "Vague and unclear",
    "Clearly stated",
    "Very long",
    "Confusing"
  ],
  "answer": "Clearly stated",
  "explanation": "Correct answer: 'Explicit' means clearly and directly expressed, so 'clearly stated' is correct. 'Vague and unclear' is the opposite of explicit. 'Very long' is incorrect because explicit describes the clarity of the instructions, not their length. 'Confusing' is incorrect because explicit means clear, which is the opposite of confusing.",
  "cefr_level": "B1",
  "subtopic": "Vocabulary in context"
}
```

### Example 4 — conversation_completion (B2)
```json
{
  "instruction": "Choose the most appropriate option to complete the conversation.",
  "stem": "Person A: We need to finalise the event schedule today. Who prepared the draft?<br>Person B: ____________",
  "options": [
    "The coordinator prepared the draft and shared it with the team.",
    "The draft was prepared and shared with the team.",
    "The coordinator has been prepared the draft and shared it.",
    "The draft prepared by the coordinator and shared with the team."
  ],
  "answer": "The coordinator prepared the draft and shared it with the team.",
  "explanation": "Correct answer: Person A asks who prepared the draft, so the response must directly name the person and state the action clearly. 'The coordinator prepared the draft and shared it with the team' does both. 'The draft was prepared and shared with the team' is incorrect because it uses passive voice and does not answer who prepared it. 'The coordinator has been prepared the draft' is incorrect because 'has been prepared' is grammatically wrong here. 'The draft prepared by the coordinator' is incorrect because it is not a complete sentence.",
  "cefr_level": "B2",
  "subtopic": "Conversation completion"
}
```

### Example 5 — grammar_application (B2)
```json
{
  "instruction": "Some sentences cannot be converted to a natural passive voice. Choose the one sentence that can be converted into a natural passive voice.",
  "stem": "Which of the following sentences can be changed into a natural <b>passive voice</b> sentence?",
  "options": [
    "He slept for two hours in the afternoon.",
    "He exists only in my memories.",
    "He wrote a detailed report for the project.",
    "He arrived late to the meeting."
  ],
  "answer": "He wrote a detailed report for the project.",
  "explanation": "Correct answer: 'He wrote a detailed report for the project' has a direct object ('a detailed report'), so it can be converted: 'A detailed report was written for the project.' 'He slept for two hours' is incorrect because 'slept' is intransitive here and cannot take a passive form naturally. 'He exists only in my memories' is incorrect because 'exists' does not take an object and cannot form a natural passive. 'He arrived late to the meeting' is incorrect because 'arrived' is intransitive and does not allow a passive construction.",
  "cefr_level": "B2",
  "subtopic": "Grammar — passive voice"
}
```

### Example 6 — text_completion (B2)
```json
{
  "instruction": "Read the text and choose the correct pair of options to complete the paragraph.",
  "stem": "<b>TEXT</b>:<br>When Maya finished her degree, she wasn't sure what to do next. She applied for a few jobs but didn't receive any replies. _____ getting disappointed, she decided to volunteer at a community centre to gain experience. She enjoyed the work and met many inspiring people, _____ helped her discover what she really wanted to do.<br><br>Choose the best pair of words to fill both blanks.",
  "options": [
    "Instead of, who",
    "Before, that",
    "Without, which",
    "Despite, where"
  ],
  "answer": "Instead of, who",
  "explanation": "Correct answer: 'Instead of getting disappointed' is correct because it shows Maya chose a positive action in place of a negative emotional response. 'Who' is correct because it refers back to 'many inspiring people,' and relative clauses referring to people use 'who.' 'Before, that' is incorrect because 'before getting disappointed' does not fit the intended contrast, and 'that' is less natural for people in this clause. 'Without, which' is incorrect because 'without getting disappointed' changes the meaning, and 'which' is not normally used for people. 'Despite, where' is incorrect because 'despite getting disappointed' does not fit naturally, and 'where' cannot refer to people.",
  "cefr_level": "B2",
  "subtopic": "Text completion — discourse connectors"
}
```

### Example 7 — passage_comprehension (A2)
```json
{
  "instruction": "Read the short passage and choose the correct answer.",
  "stem": "Passage:<br>Priya began setting her alarm 20 minutes earlier each morning. This gave her enough time to eat breakfast calmly and catch the bus without rushing, so she arrived at college on time.<br><br>Why did Priya set her alarm earlier?",
  "options": [
    "She wanted to study late at night.",
    "She wanted to have a calm morning and reach college on time.",
    "She wanted to take a longer bus route."
  ],
  "answer": "She wanted to have a calm morning and reach college on time.",
  "explanation": "Correct answer: The passage states that setting her alarm earlier gave Priya time to eat calmly, catch the bus without rushing, and arrive on time — which shows she wanted both a calm morning and to reach college punctually. 'She wanted to study late at night' is incorrect because the passage does not mention studying at night. 'She wanted to take a longer bus route' is incorrect because the passage does not mention changing her bus route.",
  "cefr_level": "A2",
  "subtopic": "Passage comprehension"
}
```

### Example 8 — sentence_correction (C1)
```json
{
  "instruction": "The highlighted part of the sentence contains an error. Choose the option that correctly replaces it.",
  "stem": "The speaker was convincing because <b>she not only presented clear evidence but also she explained the implications</b>.<br><br>Which option correctly replaces the highlighted part?",
  "options": [
    "she not only presented clear evidence but also she explained the implications",
    "she not only presented clear evidence but also explained the implications",
    "she not only presented clear evidence but also explaining the implications",
    "she only presented clear evidence but also an explanation of the implications"
  ],
  "answer": "she not only presented clear evidence but also explained the implications",
  "explanation": "Correct answer: The 'not only … but also' structure requires parallel verb forms. 'Presented' and 'explained' are both simple past tense verbs, making the structure parallel and natural. The original is incorrect because repeating 'she' after 'but also' breaks the parallel structure. 'But also explaining' is incorrect because 'explaining' (present participle) does not match 'presented' (simple past). 'She only presented … but also an explanation' is incorrect because it switches from a verb phrase to a noun phrase, breaking parallelism.",
  "cefr_level": "C1",
  "subtopic": "Sentence correction — parallel structure"
}
```

### Example 9 — matching (B1)
```json
{
  "instruction": "Read the text and choose the event that best matches the person's needs.",
  "stem": "<b>TEXT</b>:<br>Ethan wants to meet new people and practise English in an informal way.<br><br>Which event is the best choice for Ethan?",
  "options": [
    "Cooking Class – Learn recipes from around the world.",
    "Music Night – Listen to live bands every Friday.",
    "Conversation Café – Chat in English with friendly locals.",
    "Film Club – Watch classic movies together."
  ],
  "answer": "Conversation Café – Chat in English with friendly locals.",
  "explanation": "Correct answer: Ethan's two needs are meeting new people and practising English informally. 'Conversation Café' satisfies both — it involves chatting (informal English practice) with locals (meeting new people). 'Cooking Class' focuses on cooking, not English practice. 'Music Night' involves listening to music, which does not involve speaking or meeting people conversationally. 'Film Club' is a passive watching activity and does not provide an opportunity to practise speaking English.",
  "cefr_level": "B1",
  "subtopic": "Matching — event selection"
}
```
