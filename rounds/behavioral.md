# The Behavioral Round

The behavioral round asks for real events from the candidate's work, told as short stories. For a data engineer, the questions lean toward three subjects: incidents in which data was wrong, disagreements with the people who consume the data, and choices between speed and correctness. An answer needs a measured result to be convincing.

## The structure of an answer

STAR is the standard structure. It stands for Situation, Task, Action, and Result.

| Part | Share of the answer | Content |
|---|---|---|
| Situation | 15 percent | The system, the team, and the size of the data |
| Task | 10 percent | What the candidate personally had to achieve |
| Action | 55 percent | The decisions taken, with the alternatives that were rejected |
| Result | 20 percent | A number, and the change that prevents a repeat |

An answer of 2 minutes is about 300 spoken words. That leaves room for only two or three actions, so choose the ones that involved a decision.

## Six stories to prepare

| Story | The question it answers |
|---|---|
| A data incident | "Describe a time when published numbers were wrong." |
| A pipeline made reliable | "Describe a system you improved." |
| A disagreement with a consumer | "Describe a conflict with an analyst or a product manager." |
| A migration | "Describe a large project you led." |
| A deadline traded against quality | "Describe a time you shipped something incomplete." |
| A mistake of your own | "Describe a failure." |

Six stories cover most questions, because one story can answer several questions with a change of emphasis.

## A weak answer and a strong answer

**Question:** "Describe a time when published data was wrong."

**Weak:**

> "We had a bug in a pipeline and the dashboard was off for a while. I found the problem and fixed it, and then we added more tests so it would not happen again."

The answer contains no size, no cause, no decision, and no result.

**Strong:**

> "Our daily revenue table feeds the finance dashboard. One Monday the weekend total was 9 percent too high. I traced it to a job that had been retried after a timeout and had appended the same 140,000 rows twice. I had two options: delete the duplicates by hand, or change the job to overwrite its partition. I chose the second because a manual fix would not survive the next retry. I rebuilt the three affected days from raw data within 4 hours and told finance which reports to refresh. I then added a check that compares row counts with the source. It has blocked two bad loads since."

The strong answer names the cause, the alternatives, the time to repair, and the lasting change.

## What interviewers listen for

- **Ownership.** The candidate told the consumers about the error before they found it.
- **Root cause.** The fix removed the cause, not only the symptom.
- **Communication.** The explanation was adjusted for people who do not read pipelines.
- **Judgment.** The candidate can say what they would do differently.

## Common errors

- Saying "we" throughout, so the interviewer cannot tell what the candidate did.
- Describing the technology for 90 seconds and the decision for 10.
- Giving a result with no number.
- Blaming an upstream team without describing the agreement that was later made with it.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-behavioral) covers the technical rounds that these stories draw on.
- [Grokking Modern Behavioral Interview](https://www.designgurus.io/course/grokking-behavioral-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-behavioral) covers STAR, question families, and company frameworks in full.
- [Question bank](../cheat-sheets/question-bank.md) lists ten behavioral questions for data engineers.
- Free answer: [How to answer "Why should we hire you as a data engineer?"](https://www.designgurus.io/answers/detail/why-should-we-hire-you-as-a-data-engineer?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-behavioral), one common question in this round.
