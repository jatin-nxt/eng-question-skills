# Rubric Judge Skill — Semantic Quality Assessment

## Task
Evaluate the **semantic quality** of a generated question using type-specific criteria.
This judge does NOT check formatting (that is handled by FormatValidator).

Output fields:
- `overall_decision` — "pass" or "fail"
- `rubric_scores` — dict of dimension → score (0–5 scale)
- `feedback` — 2–4 sentences of specific, actionable feedback
- `failed_rubric_dimensions` — list of dimension names that scored < 3

## Evaluation Dimensions by Question Type

### MCQ (A2MCQ) — Use MCQRubric
| Dimension | Score 5 | Score 3 | Score 1 |
|-----------|---------|---------|---------|
| stem_clarity | Unambiguous; single reading | Mostly clear but slightly ambiguous | Multiple valid interpretations |
| distractor_quality | Each distractor plausible but unambiguously wrong when audio understood | Some distractors too obviously wrong or too close to correct | Distractors are random or identical to correct |
| single_correct_answer | Exactly one correct answer | Correct answer is best but another is arguable | Multiple defensible correct answers |
| explanation_accuracy | Fully explains correct + eliminates each distractor | Explains correct only or misses some distractors | Incorrect explanation or missing |
| audio_grounding | Answer requires the audio; not common knowledge | Answer likely but audio confirms | Answer is common knowledge; audio irrelevant |

Pass threshold: `overall_decision = "pass"` if **all** dimensions ≥ 3 AND `single_correct_answer` ≥ 4.

### Speaking (T2M, A2M, P2M) — Use SpeakingRubric
| Dimension | Score 5 | Score 3 | Score 1 |
|-----------|---------|---------|---------|
| task_clarity | Learner knows exactly what to say and for how long | Mostly clear; minor ambiguity | Unclear what to do or say |
| elicitation_quality | Prompt genuinely elicits extended spoken output (30–90s) | Elicits brief but adequate response | Yes/no or one-word answer is all that's needed |
| achievability | Fully answerable without extra knowledge | Mostly answerable but may trip some learners | Requires knowledge not available from audio/picture |
| register_appropriateness | Register precisely matches CEFR level | Register is close to target level | Register is too formal or too informal for level |
| modality_consistency | Delivery mode matches question type (see rules below) | Minor wording slip but task is still doable | Wrong modality — learner cannot follow instructions |

**modality_consistency rules by type:**
- **T2M** (text → microphone): When the task content is shown in `stem`, `instruction` must use *Read…* / *Look at…* — never *Listen…*, *hear*, or *audio*. Exception: `sentence_repetition` may say *Listen carefully and repeat…* for one short sentence played from the stem.
- **A2M / P2M** (audio/picture → microphone): `instruction`/`stem` must reference listening or viewing; `transcript_ref` or `picture_ref` must support the task.
- Fail T2M immediately (score 1) for patterns like *Listen to the following paragraph…* when the paragraph text is already in `stem`.

Pass threshold: all dimensions ≥ 3.

### Writing (A2T) — Use WritingRubric
| Dimension | Score 5 | Score 3 | Score 1 |
|-----------|---------|---------|---------|
| task_clarity | Learner knows exactly what to write and how long | Mostly clear with minor ambiguity | Unclear genre/purpose/length |
| response_length_appropriate | Length instruction matches CEFR expectation precisely | Roughly appropriate | Too short for C1 or too long for A2 |
| content_grounding | Requires using the audio content; not generic | Mostly grounded in audio | Generic — could answer without the audio |
| achievability | Fully achievable in one read/listen | Mostly achievable | Requires re-listening or external knowledge |

Pass threshold: all dimensions ≥ 3.

### MCQ — Text-based (MCQ) — Use TextMCQRubric
| Dimension | Score 5 | Score 3 | Score 1 |
|-----------|---------|---------|---------|
| stem_clarity | Unambiguous; single reading; source text and question are clearly separated | Mostly clear but slightly ambiguous wording | Multiple valid interpretations of the question |
| distractor_quality | Each distractor is plausible to an inattentive reader but unambiguously wrong once the text is read carefully | Some distractors are too obviously wrong or too close to the correct answer | Distractors are random, unrelated, or identical to the correct answer |
| single_correct_answer | Exactly one correct answer; no other option can be independently defended | Correct answer is the best option but another is arguable with effort | Two or more options are both defensible as correct |
| explanation_accuracy | Correctly justifies the answer and individually eliminates each distractor with a specific reason | Justifies the correct answer but only partially addresses distractors | Explanation is incorrect, missing, or refers to a different item |

Pass threshold: `overall_decision = "pass"` if **all** dimensions ≥ 3 AND `single_correct_answer` ≥ 4.

> **Note**: `audio_grounding` is intentionally absent — MCQ is text-only; there is no audio stimulus to ground.

### T2T — Text-to-Text (T2T) — Use T2TRubric
| Dimension | Score 5 | Score 3 | Score 1 |
|-----------|---------|---------|---------|
| task_clarity | Learner knows exactly what transformation or writing to produce; task type unambiguous | Mostly clear; minor ambiguity about expected output form | Unclear what the learner must do or what a correct answer looks like |
| instruction_completeness | All constraints explicitly stated in `instruction` before `stem` — agent keep/omit rule, word count, contraction ban, output format, etc. | Most constraints stated; one minor constraint buried or missing | Key constraint appears for the first time inside `stem`, or is absent entirely |
| answer_accuracy | Model answer is demonstrably correct per the stated rule; every source word accounted for; agent rule exactly followed; word count within stated range | Answer is mostly correct with a minor omission or slight word-count deviation | Answer violates the stated rule (e.g., includes omitted agent), omits a key word, or falls significantly outside the word count |
| achievability | Task is fully completable using only the source text; difficulty matches stated CEFR level | Mostly achievable; one vocabulary item or structure may trip a borderline learner | Requires external knowledge not in the stem, or the complexity is clearly mismatched to the CEFR level |

Pass threshold: all dimensions ≥ 3.

**Type-specific scoring notes:**
- `word_rearrangement`: `answer_accuracy` fails if any word from `stem` is missing or if contractions appear when banned.
- `passive_voice_conversion` / `passive_voice_identification`: `answer_accuracy` fails if the agent rule (keep/omit) is not followed exactly, or if the sentence chosen for conversion has no direct object.
- `reported_speech_conversion`: `answer_accuracy` fails if tense backshift, pronoun shift, or time/place expression change is missing.
- `fill_in_gaps`: `answer_accuracy` fails if any gap answer is grammatically incorrect or semantically wrong in context.
- `situational_writing` / `notes_to_paragraph` / `guided_writing`: `instruction_completeness` must include the word/sentence count limit; `answer_accuracy` checks that the model answer meets that limit.
