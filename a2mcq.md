---

## name: a2mcq-question-generator
description: >
  Generates audio-to-multiple-choice (A2MCQ) English assessment questions across all CEFR levels (A1–C2).
  The learner listens to an audio clip and selects the correct answer from a set of options.
  Always use this skill when the user asks to generate A2MCQ items, audio-based MCQ questions,
  listening comprehension multiple-choice tasks, or any question where a learner listens to
  an audio clip and selects an answer — even if the user doesn't say "A2MCQ" explicitly.

# A2MCQ — Audio-to-Multiple-Choice English Assessment Question Generator

## Output Format

Produce a **JSON array** of `batch_size` question objects. Each object must contain exactly these keys:


| Key              | Description                                                                                                                                                                                                                     |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `question_id`    | Pre-assigned ID — **copy verbatim** from the corresponding entry in the `question_ids` input (same order).                                                                                                                      |
| `transcript_ref` | **Required.** Verbatim listening script as **plain text only** (no HTML). Full text of the clip the learner hears; length should match the CEFR task (A1–A2 often a short passage; B1+ may use a longer monologue or dialogue). |
| `instruction`    | 1 sentence telling the learner what to do; HTML (`<b>term</b>`) permitted.                                                                                                                                                      |
| `stem`           | The question text the learner answers; HTML (`<b>term</b>`) permitted for key vocabulary.                                                                                                                                       |
| `options`        | List of exactly `num_options` answer strings (default 4 unless otherwise specified).                                                                                                                                            |
| `answer`         | The single correct option — must appear verbatim in `options`.                                                                                                                                                                  |
| `explanation`    | At least 2 lines (or ≥ 100 characters) explaining why the answer is correct and why each distractor is wrong.                                                                                                                   |
| `cefr_level`     | One of: A1, A2, B1, B2, C1, C2                                                                                                                                                                                                  |
| `subtopic`       | Brief topic label (e.g., "Transport and travel")                                                                                                                                                                                |


### **JSON Validation & Batch Rules**

1. **Output a JSON array**: The outer container must be `[...]`. Do NOT wrap in `{...}`.
2. `**question_id` required**: Every object must include `"question_id"` copied exactly from `question_ids` (in order).
3. `**batch_size` objects**: The array must contain exactly `batch_size` objects — no more, no fewer.
4. **Strict Simple Tags Only**: Use only `<b>`, `<i>`, `<br>`. Do **NOT** use tags requiring attributes.
5. **Double-Quote Escaping**: All nested double quotes inside string fields must be escaped as `\"`.
6. **No Code Blocks**: Output *only* the raw JSON array — do not wrap in markdown code blocks.
7. **No HTML in `transcript_ref`**: Plain text only. Never put HTML tags inside the transcript.
8. **Never bold options**: `<b>` tags belong only in `instruction` or `stem`, never inside any option string.

**NEVER** combine `instruction` and `stem` into one field. `instruction` tells the learner how to engage with the task (e.g., "Listen carefully and choose the correct answer."). `stem` contains the actual question about the audio content.

---

## **CEFR Guidelines**


| Level | Vocabulary                | Stem complexity                       | Distractor difficulty                                          |
| ----- | ------------------------- | ------------------------------------- | -------------------------------------------------------------- |
| A1    | High-frequency words only | Short, direct factual question        | Clearly wrong but plausible at first glance                    |
| A2    | Common words              | Simple question about explicit detail | Require reading the stem carefully to eliminate                |
| B1    | General vocabulary        | Inference or paraphrase of audio      | Contain plausible but incorrect details                        |
| B2    | Wider range               | Interpretation or speaker intent      | Subtle semantic differences from correct answer                |
| C1    | Sophisticated             | Critical or analytical question       | Very close paraphrases; only careful listeners can distinguish |
| C2    | Near-native               | Complex evaluative question           | Require full understanding of nuance to eliminate              |


---

## **Formatting Rules (mandatory)**

1. **Case rule**: If the stem ends with a blank (`_____`), options start with **lowercase**. If the stem is a complete sentence or question, options start with **uppercase**.
2. **Comma spacing**: Correct — "cats, dogs, and birds". Incorrect — "cats,dogs,and birds".
3. **Punctuation**: If options complete a sentence stem, they must end with a **period**. If options are standalone answers or words or phrases or clauses, no period.
4. **Bold tags**: Use `<b>term</b>` only inside `instruction` or `stem`. Never bold options.
5. **Explanation length**: At least 2 full lines OR ≥ 100 characters. Explain why the correct answer is right first, then state why each wrong option is incorrect.
6. **Distractors**: Each distractor must be plausible (a learner who didn't understand the audio might choose it) but unambiguously wrong once the audio is understood.
7. **Options unique**: All options must be distinct — no two options can be the same or near-identical paraphrases.
8. **No aggregate distractors**: Never use "All of the above" or "None of the above" unless every option listed is explicitly confirmed or denied by the audio.

---

## **Error Prevention**

## **1 Incorrect Answer Key (CRITICAL)**

**Definition**: The `answer` field does not match what the audio transcript actually states. **Severity**: Fatal — makes the item unusable.


| Audio says                                  | Wrong answer key |
| ------------------------------------------- | ---------------- |
| "She got up at five every morning."         | "7:00"           |
| "He started at twelve, helping his mother." | "At age 10"      |


**Prevention**: Verify the `answer` word-for-word against the exact text in `transcript_ref` before outputting. Never rely on memory — re-read the transcript.

---

## **2 Grammar Error in Answer Key**

**Definition**: The correct option in `answer` contains a grammatical error. **Severity**: Fatal — the answer is a benchmark; errors undermine credibility.

Common traps:  *"He need time"* (not *"He needs time"*)  *"She is an coach"* (not *"She is a coach"*)  *"They was delayed"* (not *"They were delayed"*)

**Prevention**: Re-read every option for subject-verb agreement, correct article usage, and tense consistency before outputting to ensure that it is approved by standard grammar.

---

## **3 Answer–Question Mismatch (CRITICAL)**

**Definition**: The question asks about one subject or entity but the answer refers to a different subject — OR the question asks about something the audio never mentions. **Severity**: Fatal — the item tests something it never presented.


| Question subject             | Audio subject                               | Error                           |
| ---------------------------- | ------------------------------------------- | ------------------------------- |
| "What did the **girl** buy?" | Audio only mentions what the **boy** bought | Girl's action absent from audio |


**Prevention**: Match every pronoun and entity name in the stem to the corresponding speaker or subject in the transcript. If the audio never addresses the question subject, revise the stem or the transcript.

---

## **4 Complete Opposite Meaning in Answer (CRITICAL)**

**Definition**: The `answer` states the opposite of what the audio expresses. **Severity**: Fatal — fundamentally contradicts the source material.


| Audio expresses                                          | Wrong answer             |
| -------------------------------------------------------- | ------------------------ |
| Team played well, kept spirits up; loss was "not so bad" | "They should be ashamed" |


**Prevention**: Re-read the transcript after drafting the answer. Check that the overall tone and conclusion of the audio match the selected answer.

---

## **5 Invalid Aggregate Distractor**

**Definition**: A distractor uses "All of the above" or a similar aggregated option when the audio only supports one of the listed choices. **Severity**: High — makes the item misleading and untestable.

**Example**: Audio mentions only a payment failure. Distractor "All of the above" implies login failure and missing scores are also mentioned — they are not.

**Prevention**: Never use "All of the above" unless every single option in the list is explicitly confirmed by the audio transcript.

---

## **6 Explanation Gaps**

**Definition**: The `explanation` field omits why one or more wrong options are incorrect, or is shorter than 100 characters. **Severity**: Medium — reduces item utility for learners reviewing their answers.

**Prevention**: After drafting, verify: (a) correct answer explained ✓, (b) every distractor addressed individually ✓, (c) total length ≥ 100 characters ✓.

---

## **Generation Workflow**

1. Identify inputs: `transcript_ref` (or topic to generate one), `num_options`, `cefr_level`
2. Draft `transcript_ref` if not provided — plain text, appropriate length and complexity for CEFR level
3. Draft `stem` — question about a specific detail, inference, or interpretation from the transcript; bold key vocabulary
4. Draft `instruction` — one sentence: e.g., "Listen carefully and choose the correct answer" or "Choose the best answer based on the audio"
5. Draft `options` — one correct answer verified against transcript  `num_options - 1` plausible distractors
6. Set `answer` — copy verbatim from the correct option; verify against transcript
7. Draft `explanation` — correct answer first, then each wrong option with a specific reason
8. Self-check: answer verified against transcript ✓ | grammar in all options ✓ | no aggregate distractors ✓ | explanation ≥ 100 chars and covers all options ✓
9. Output the final JSON

---

## **Few-Shot Examples**

### **Example 1 — A2 (4 options)**
```json
[{  
  "transcript_ref": "Tom goes to school by bus every morning. His school starts at 8 o'clock.",  
  "instruction": "Choose the correct answer based on what you heard.",  
  "stem": "How does Tom <b>travel to school</b>?",  
  "options": ["By bus", "By car", "On foot", "By bicycle"],  
  "answer": "By bus",  
  "explanation": "The audio clearly states that Tom goes to school by bus every morning. 'By car' is incorrect because no car is mentioned. 'On foot' and 'by bicycle' are wrong because the transcript specifies only the bus.",  
  "cefr_level": "A2",  
  "subtopic": "Transport and travel"  
}]
```

### **Example 2 — B1 (4 options)**
```json
[{  
  "transcript_ref": "The museum recently introduced free entry on Sundays to encourage more families to visit. Since then, visitor numbers have doubled.",  
  "instruction": "Choose the best answer based on the audio.",  
  "stem": "Why did the museum introduce <b>free entry on Sundays</b>?",  
  "options": [ "To attract more families",  
    "To reduce the cost of running the museum",  
    "Because visitor numbers had fallen",  
    "To compete with other museums in the city"
  ],  
  "answer": "To attract more families",  
  "explanation": "The audio states the museum introduced free entry 'to encourage more families to visit', making option A correct. Option B is not mentioned in the transcript. Option C contradicts the audio — visitor numbers increased after the change. Option D is not referenced at all.",  
  "cefr_level": "B1",  
  "subtopic": "Culture and community"  
}]
```

### **Example 3 — B2 (3 options)**
```json
[{  
  "transcript_ref": "The researcher argues that while electric vehicles reduce tailpipe emissions, the environmental benefit is limited unless the electricity grid is also decarbonised.",  
  "instruction": "Choose the option that best reflects the researcher's view.",  
  "stem": "What does the researcher suggest about the <b>environmental impact</b> of electric vehicles?",  
  "options": ["They only fully benefit the environment when powered by clean energy",  
    "They are not effective at reducing emissions under any circumstances",  
    "They are the single most important solution to climate change"
  ],  
  "answer": "They only fully benefit the environment when powered by clean energy",  
  "explanation": "The researcher states that benefits are 'limited unless the electricity grid is also decarbonised', meaning clean energy is required for the full benefit — matching option A. Option B overstates the negative; the researcher says benefits are limited, not absent. Option C is an overstatement not supported by the transcript.",  
  "cefr_level": "B2",  
  "subtopic": "Technology and environment"  
}]
```

### **Batch Example — 2 items (array output)**

When `batch_size` is 2 and `question_ids` is `["abc123_A2MCQ_A2_000", "abc123_A2MCQ_A2_001"]`, output:
```json
[  
  {  
    "question_id": "abc123_A2MCQ_A2_000",  
    "transcript_ref": "Interviewer: David, when did you start cooking? David: I started when I was twelve, helping my mother. Interviewer: What food do you like to cook most? David: Pasta dishes, especially with tomato sauce.",  
    "instruction": "Choose the correct answer based on what you heard.",  
    "stem": "When did David <b>start cooking</b>?",  
    "options": ["At age 8", "At age 10", "At age 12", "At age 15"],  
    "answer": "At age 12",  
    "explanation": "David says he started cooking when he was twelve, helping his mother. Ages 8, 10, and 15 are not mentioned anywhere in the transcript.",  
    "cefr_level": "A2",  
    "subtopic": "Interviews and personal history"  
  },  
  {  
    "question_id": "abc123_A2MCQ_A2_001",  
    "transcript_ref": "Interviewer: David, when did you start cooking? David: I started when I was twelve, helping my mother. Interviewer: What food do you like to cook most? David: Pasta dishes, especially with tomato sauce.",  
    "instruction": "Choose the correct answer based on what you heard.",  
    "stem": "What kind of food does David <b>like to cook</b> most?",  
    "options":["Rice dishes", "Pasta dishes", "Soup", "Bread"],  
    "answer": "Pasta dishes",  
    "explanation": "David says his favourite food to cook is pasta dishes, especially with tomato sauce. Rice dishes, soup, and bread are not mentioned in the transcript.",  
    "cefr_level": "A2",  
    "subtopic": "Interviews and personal history"  
  }  
]
```