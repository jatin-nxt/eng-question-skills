---

## name: t2t-question-generator

description: >
  Generates text-to-text (T2T) English assessment tasks across all CEFR levels (A1–C2).
  The learner reads a written prompt and produces a written response: rearranging words,
  converting sentences (passive voice, reported speech), filling in gaps, writing a reply
  to a situation, expanding notes into a paragraph, or producing a guided piece of writing.
  Always use this skill when the user asks to generate T2T prompts, written transformation
  tasks, grammar conversion exercises, cloze tasks, situational writing, guided writing,
  or any task where a learner reads text and writes their answer — even if the user doesn't
  say "T2T" explicitly. This skill is for written-response tasks only; for spoken response
  use the T2M skill, and for audio-input tasks use A2T.

# T2T — Text-to-Text English Assessment Question Generator

## Output Format

Produce a **JSON object** with exactly these keys:


| Key           | Description                                                                                                                                                                                                                                                          |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `instruction` | 1–3 sentences stating the task type, transformation rules (e.g., keep or omit the agent), format constraints (word count, sentence count), and output instructions (e.g., "Write your answer in the text box. Do not add punctuation."). HTML `<b>term</b>` allowed. |
| `stem`        | The source text the learner works with: a sentence to transform, a cloze text with gaps, a situation and message, bullet notes, or a writing prompt. HTML `<b>term</b>` and `<br>` permitted for formatting.                                                         |
| `answer`      | Model written response — correct transformation, gap-fill answers (comma-separated for multi-gap), or a model piece of writing at the target CEFR level.                                                                                                             |
| `cefr_level`  | One of: A1, A2, B1, B2, C1, C2                                                                                                                                                                                                                                       |
| `subtopic`    | Brief topic label (e.g., "Passive voice", "Guided writing")                                                                                                                                                                                                          |


> ### JSON Validation & HTML Escaping Rules
>
> 1. **Strict Simple Tags Only**: Use only `<b>`, `<i>`, `<br>`. Do **NOT** use tags requiring attributes.
> 2. **Double-Quote Escaping**: All nested double quotes inside string fields must be escaped as `\"`.
> 3. **No Code Blocks**: Output *only* the raw JSON object — do not wrap in markdown code blocks.

> **NEVER** combine `instruction` and `stem` into one field.
> `instruction` states what the learner must do and any constraints. `stem` contains the actual source text, sentences, cloze passage, notes, or writing prompt — nothing else.

---

## Question Types


| #   | Type                           | What the learner does                                                                               | CEFR range |
| --- | ------------------------------ | --------------------------------------------------------------------------------------------------- | ---------- |
| 1   | `word_rearrangement`           | Rearranges jumbled words into one correct sentence                                                  | A1–C2      |
| 2   | `passive_voice_conversion`     | Rewrites one active sentence in passive voice; agent kept or omitted as instructed                  | B1–C1      |
| 3   | `passive_voice_identification` | Given two sentences, identifies which can be converted to passive and rewrites only that one        | B1–C1      |
| 4   | `reported_speech_conversion`   | Converts direct speech to reported speech, or rearranges jumbled words into correct reported speech | B1–B2      |
| 5   | `fill_in_gaps`                 | Writes one word per blank in a cloze text                                                           | A2–B2      |
| 6   | `situational_writing`          | Reads a situation and a message, writes a polite reply within a stated word or sentence limit       | B1–B2      |
| 7   | `notes_to_paragraph`           | Expands structured bullet notes into a coherent paragraph within a stated word count                | A2–B2      |
| 8   | `guided_writing`               | Writes a diary entry, message, blog post, or short article following a prompt and word limit        | B1–C1      |


---

## CEFR Guidelines


| Level | Expected writing style                                                       | Typical task types                                                           |
| ----- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| A1    | Short, simple sentences; high-frequency vocabulary                           | Simple word rearrangement; single-word gap-fill                              |
| A2    | Short paragraphs; basic connectors (and, but, because)                       | Word rearrangement; gap-fill; very short notes-to-paragraph                  |
| B1    | Clear paragraphs; range of connectors; simple passive and reported speech    | Passive conversion; reported speech; situational writing; notes-to-paragraph |
| B2    | Varied sentence structures; wider vocabulary; nuanced transformations        | Complex passive; reported speech; guided writing with specific constraints   |
| C1    | Sophisticated vocabulary; complex clause structures; precise transformations | Advanced grammar conversion; extended guided writing                         |
| C2    | Near-native range; idiomatic control; critical and stylistic awareness       | Complex guided writing; multi-step transformations                           |


---

## Generation Rules

### Language & Clarity

1. Every `answer` must be traceable directly to the `stem` — never introduce facts, vocabulary, or sentences that are not in the source material.
2. The `instruction` must state all constraints explicitly before the learner reads the `stem` — no constraint should appear for the first time inside `stem`.
3. Use neutral, formal English. Avoid slang, region-specific idioms, or culturally biased content.
4. Each task is fully self-contained; the learner must not need any prior context.
5. Word or sentence count stated in `instruction` must be achievable by the model `answer` — verify the word count of `answer` before outputting.

### By Question Type

- **word_rearrangement**: The jumbled set must form exactly one valid, grammatical sentence. If the instruction forbids contractions, `answer` must contain no contractions. Separate words with `/` in `stem`.
- **passive_voice_conversion**: State explicitly in `instruction` whether the agent must be kept or omitted. The `answer` must honour this exactly — "do not omit the agent" means the "by [agent]" phrase is required; "omit the agent" means it must be absent. Only convert sentences that have a clear direct object; if the sentence cannot form a natural passive, state this in the explanation.
- **passive_voice_identification**: Give exactly two sentences in `stem`. The correct one must have a direct object that allows natural passive conversion. The other must be genuinely non-convertible (intransitive verb, stative verb, etc.). `answer` rewrites only the convertible sentence. If neither sentence can be naturally converted, `answer` must state "Neither sentence can be converted to a natural passive voice."
- **reported_speech_conversion**: `answer` must apply all required changes: pronoun shift, tense backshift, time/place expression change, and removal of quotation marks. For jumbled-words variant, all words in the `stem` set must appear in `answer`.
- **fill_in_gaps**: Each blank must have exactly one clearly correct answer that fits grammatically and semantically. `answer` lists the answers comma-separated in gap order. The `instruction` must say "Do not add punctuation" and specify how many words per gap (typically one).
- **situational_writing**: `stem` must contain both the situation description and the message/prompt the learner is replying to, clearly labelled. The `answer` must address the situation politely, stay within the word or sentence limit, and use appropriate register.
- **notes_to_paragraph**: `stem` presents the notes in a structured list. `answer` must use complete sentences, cover all note points, and stay within the stated word count. Count the words in `answer` before outputting.
- **guided_writing**: `stem` provides the writing prompt (topic, opening sentence if any, or bullet points). `answer` must follow the stated format, word count, and any required content points. Count the words in `answer` before outputting.

---

## Error Prevention

## 1. Agent Inclusion/Omission Error (CRITICAL)

**Definition**: The `answer` includes the agent ("by [person]") when `instruction` says to omit it, or omits it when `instruction` says to keep it.
**Severity**: Fatal — the answer directly contradicts the stated task rule.


| Instruction says             | Wrong answer                                           | Correct answer                                          |
| ---------------------------- | ------------------------------------------------------ | ------------------------------------------------------- |
| "omit the agent/doer"        | "The proposal has been approved **by the committee**." | "The proposal has been approved."                       |
| "do not omit the agent/doer" | "The broken chair needs to be repaired."               | "The broken chair needs to be repaired **by someone**." |


**Prevention**: Immediately before writing `answer`, re-read the exact instruction phrase about the agent. Apply it literally.

---

## 2. Missing Key Word in Transformation (CRITICAL)

**Definition**: The `answer` omits a word from the source sentence that is required for correct meaning or grammatical completeness.
**Severity**: Fatal — changes the meaning or makes the answer factually wrong.

**Example**: Source contains "sudden infant death syndrome"; answer writes "infant death syndrome" — omitting "sudden" changes the medical term entirely.

**Prevention**: After drafting `answer`, read it against the `stem` word by word. Every content word in the source sentence must appear in the transformed answer unless the task explicitly instructs removal (e.g., omit the agent).

---

## 3. Fabricated or Mismatched Answer (CRITICAL)

**Definition**: The `answer` introduces a sentence, word, or content that does not exist in `stem` — commonly occurring in `passive_voice_identification` when neither source sentence is actually convertible.
**Severity**: Fatal — the answer is uncheckable against the source.

**Example**: `stem` contains "The baby slept peacefully. / He laughed at the joke." — neither has a direct object — but `answer` gives "The toy was broken by the baby," which is not in `stem` at all.

**Prevention**: For `passive_voice_identification`, before converting, confirm that the chosen sentence has a direct object. If neither sentence qualifies, write: "Neither sentence can be converted to a natural passive voice." Never invent a new sentence.

---

## 4. Gap-Fill Wrong Word

**Definition**: One or more words in the `answer` are grammatically incorrect, logically wrong, or do not fit the specific gap in the cloze text.
**Severity**: High — incorrect gap answers mislead learners.


| Gap context                                     | Wrong answer word | Correct word      |
| ----------------------------------------------- | ----------------- | ----------------- |
| "I didn't know ____ to read music"              | "why"             | "how"             |
| "improving ____ week"                           | "every"           | "each"            |
| "____ I finish my homework" (sequential action) | "after"           | "when" / "before" |


**Prevention**: For each blank, substitute the candidate word back into the full sentence and read it aloud mentally. Verify it is grammatically correct, semantically fitting, and the only natural choice for that position.

---

## 5. Word Count Violation

**Definition**: The word count of `answer` is significantly below or above the range stated in `instruction`.
**Severity**: High — the task has an explicit length requirement; exceeding or falling short of it makes the answer an invalid model.


| Instruction states                      | Wrong answer length | Correct range |
| --------------------------------------- | ------------------- | ------------- |
| "Write a short paragraph (35–45 words)" | ~22 words           | 35–45 words   |
| "Write a message (about 10 words)"      | ~100 words          | ~10 words     |
| "Write an 80–100 word diary entry"      | ~40 words           | 80–100 words  |


**Prevention**: Count the words in `answer` before outputting. If the count falls outside the stated range, expand or trim until it falls within bounds.

---

## 6. Incorrect Grammar in Transformation

**Definition**: The `answer` uses the wrong conjunction, preposition, tense, or pronoun in a transformation task.
**Severity**: Medium — produces a grammatically incorrect model answer.

Common traps:

- Gap requires "since" (duration from a past point to now) but answer gives "because" (reason)
- Gap requires "to" (direction) but answer gives "from" ("been to London" not "been from London")
- Reported speech requires tense backshift ("will" → "would") but answer keeps original tense
- Reported speech requires pronoun shift ("I" → "she/he") but answer keeps first person

**Prevention**: For transformation tasks, apply the grammar rule step by step before outputting. For reported speech: shift tense ✓ | shift pronouns ✓ | shift time/place expressions ✓.

---

## Generation Workflow

1. Identify inputs: `question_type`, `cefr_level`, topic or source sentence
2. Draft `stem` — the source sentence(s), cloze text, situation + message, notes, or writing prompt; format with `<br>` and labels as needed
3. Draft `instruction` — state the task type, all transformation rules (agent keep/omit, contraction ban), format constraints (word count), and output instructions; use imperative form
4. Draft `answer` — apply the transformation correctly, or write a model response within the stated word count; for multi-gap fill, list answers comma-separated in gap order
5. Self-check:
  - Transformation tasks: every source word present ✓ | agent rule followed exactly ✓ | grammar correct ✓
  - Gap-fill: each answer word fits its specific gap grammatically and semantically ✓
  - Writing tasks: word count within stated range ✓ | all note/prompt points covered ✓ | CEFR register ✓
6. Output the final JSON

---

## Few-Shot Examples

### Example 1 — word_rearrangement (B1)

```json
{
  "instruction": "Rearrange the given words to form a correct sentence. Do not use any contractions.",
  "stem": "to treatment / never known / doctors / respond well / have / why some / cancers / while others / do not",
  "answer": "Doctors have never known why some cancers respond well to treatment while others do not.",
  "cefr_level": "B1",
  "subtopic": "Word rearrangement"
}
```

### Example 2 — passive_voice_conversion, agent kept (A2)

```json
{
  "instruction": "Rewrite the given sentence in the passive voice. Do not omit the agent/doer.",
  "stem": "A student answered all the questions.",
  "answer": "All the questions were answered by a student.",
  "cefr_level": "A2",
  "subtopic": "Passive voice — agent kept"
}
```

### Example 3 — passive_voice_conversion, agent omitted (B1)

```json
{
  "instruction": "Rewrite the given sentence in the passive voice. Omit the agent/doer.",
  "stem": "They are going to display the notice on the board.",
  "answer": "The notice is going to be displayed on the board.",
  "cefr_level": "B1",
  "subtopic": "Passive voice — agent omitted"
}
```

### Example 4 — passive_voice_identification (B1)

```json
{
  "instruction": "Two sentences are given below. Identify the sentence that can be converted to a natural passive voice and rewrite only that sentence. Do not omit the agent/doer.",
  "stem": "She resembles her mother.\nShe built the house.",
  "answer": "The house was built by her.",
  "cefr_level": "B1",
  "subtopic": "Passive voice — identification and conversion"
}
```

### Example 5 — reported_speech_conversion (B1)

```json
{
  "instruction": "Convert the sentence into reported speech. Write your answer in the text box.",
  "stem": "\"Don't open the window while the baby is sleeping,\" mother said to my sister.",
  "answer": "Mother warned my sister not to open the window while the baby was sleeping.",
  "cefr_level": "B1",
  "subtopic": "Reported speech — imperative"
}
```

### Example 6 — fill_in_gaps (A2)

```json
{
  "instruction": "Read the text and write one word for each gap. Write your answers in the text box. Do not add punctuation.",
  "stem": "<b>TEXT</b>:<br>My parents took me to the science museum last Saturday. I was excited because I had never been there ______. We saw a huge exhibition about space and learned ______ the planets. I really liked the part ______ you could try driving a small robot.",
  "answer": "before, about, where",
  "cefr_level": "A2",
  "subtopic": "Gap-fill — prepositions and connectors"
}
```

### Example 7 — situational_writing (B1)

```json
{
  "instruction": "Read the situation. Write the reply in one sentence.",
  "stem": "Situation: You cannot submit your assignment today because your internet connection is not working. You want to inform your teacher politely.\n\nMessage: \"Please submit your assignment by 5 p.m. today.\"",
  "answer": "I'm sorry, I cannot submit my assignment by 5 p.m. today because my internet connection is not working.",
  "cefr_level": "B1",
  "subtopic": "Situational writing — polite reply"
}
```

### Example 8 — notes_to_paragraph (A2)

```json
{
  "instruction": "Write a short paragraph (35–45 words) using the notes. Use complete sentences.",
  "stem": "Notes:\nlast Saturday: school clean-up drive\ncleaned classrooms and playground\nplanted small trees\nfelt happy and proud",
  "answer": "Last Saturday, I took part in a school clean-up drive. We cleaned the classrooms and the playground. We also planted some small trees. I felt happy because I helped my school, and I was proud of our teamwork.",
  "cefr_level": "A2",
  "subtopic": "Notes to paragraph"
}
```

### Example 9 — guided_writing (B1)

```json
{
  "instruction": "Write an 80–100 word diary entry. Your story must begin with the sentence provided.",
  "stem": "Your story must begin with: \"It was raining very hard when I left the house …\"",
  "answer": "It was raining very hard when I left the house. I forgot my umbrella and got completely wet. On the way, I met my friend, and she kindly shared her umbrella with me. We walked together to school, laughing about how unprepared we both were. By the time we arrived, my shoes were soaked, but I did not mind because I had such good company. It was a bad start but a very good finish.",
  "cefr_level": "B1",
  "subtopic": "Guided writing — diary entry"
}
```
### Example 10 — reported_speech_conversion, jumbled-words variant  (B1)

```json
{
  "instruction": "Rearrange the jumbled words to form correct reported speech. Write your answer in the text box.",
  "stem": "the guide / told / us / that / we / should / not / litter",
  "answer": "The guide told us that we should not litter.",
  "cefr_level": "B1",
  "subtopic": "Reported speech — jumbled words"
}
```
