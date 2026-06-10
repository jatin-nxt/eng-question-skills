# Difficulty Judge Skill — CEFR Level Assessment

## Task
Evaluate the difficulty of a generated question and predict its **CEFR level** (A1, A2, B1, B2, C1, or C2).

Output fields:
- `predicted_cefr` — one of: A1, A2, B1, B2, C1, C2
- `confidence` — float 0.0–1.0 (how certain you are of this prediction)
- `reasoning` — 2–4 sentences explaining the key linguistic features that determined the level

## CEFR Level Criteria Table

| Level | Vocabulary Range | Grammar Complexity | Clause Density | Cognitive Demand |
|-------|-----------------|-------------------|----------------|-----------------|
| A1 | Only high-frequency basic words (≤ 1000 most common) | Simple present/past; SVO only | Single clause | Literal recall, concrete |
| A2 | Common everyday words; familiar topics | Simple compound sentences; basic connectors (and, but, so) | 1–2 clauses | Concrete comprehension |
| B1 | General vocabulary; some abstract nouns | Complex sentences; can/should/might; subordinates (because, if) | 2–3 clauses | Mild inference; opinion on familiar topics |
| B2 | Wider range; idiomatic phrases; abstract vocabulary | Conditionals; passive; relative clauses; reported speech | 3–4 clauses | Analysis; argument; unfamiliar topics |
| C1 | Sophisticated; nuanced; low-frequency vocabulary | Wide range; nominalisations; complex subordination | 4–5 clauses | Critical thinking; implicit meaning |
| C2 | Near-native; rare vocabulary; technical register | Full grammatical range; highly complex | 5+ clauses | Complex evaluation; cultural nuance |

## Scoring Approach
Assess the question across three dimensions:

1. **Vocabulary difficulty** — What is the highest-frequency band of vocabulary used?
2. **Grammar complexity** — What grammatical structures appear? What is the most complex?
3. **Cognitive demand** — What thinking skill is required? (recall → inference → analysis → evaluation)

The predicted CEFR level should reflect the **harder** dimension — a question with B2 vocabulary but B1 grammar should be rated B2.
