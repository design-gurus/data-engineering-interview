# The Data Engineering Interview Loop

A data engineering loop usually has three stages: a recruiter call, one technical screen, and four or five final rounds. The final rounds cover SQL, coding, data modeling, pipeline design, and behavior. The details below are commonly reported by candidates and are not official for any company.

## The stages

| Stage | Length | Content |
|---|---|---|
| Recruiter call | 30 minutes | Background, level, and the format of the later stages |
| Technical screen | 45 to 60 minutes | Two or three SQL questions and one short Python problem |
| Final rounds | 4 to 5 rounds of 45 to 60 minutes | One round per skill, described below |
| Decision | About 1 week | Interviewers submit written feedback, then meet |

## The final rounds

| Round | A typical question | What separates a strong answer |
|---|---|---|
| [SQL](sql.md) | "Return the three most played episodes for each show." | Stating how ties are handled before writing the query |
| [Coding](coding.md) | "Parse this log file and return the top ten addresses." | Handling bad rows and input larger than memory |
| [Data modeling](data-modeling.md) | "Model the data for a pharmacy chain's sales." | Declaring the grain, meaning what one row represents, first |
| [Pipeline design](pipeline-design.md) | "Design a pipeline for clickstream events." | Explaining reruns, late data, and cost, not only the case in which nothing fails |
| [Behavioral](behavioral.md) | "Describe a time when published data was wrong." | A measured result and a change that prevents a repeat |

Some companies add a product sense round. The candidate defines a metric for a product, designs the tables that support it, and writes the SQL that computes it. Candidates commonly report this combined format at large consumer companies.

## How the loop changes with level

| Level | Emphasis |
|---|---|
| Entry | SQL and coding carry most of the decision. Design questions are small. |
| Mid | All five rounds count. The design round expects one complete batch pipeline. |
| Senior | Pipeline design and data modeling carry the most weight. Follow-up questions cover failure, cost, and migration. |
| Staff | The design round widens to several teams: ownership, contracts between producers and consumers, and governance. |

## What interviewers write down

Interviewers usually score a small set of signals in each round. Knowing them helps a candidate decide what to say aloud.

- **Correctness.** Does the query or design return the right result on edge cases such as ties, nulls, and duplicates?
- **Reasoning.** Does the candidate state assumptions and explain each choice?
- **Operational thinking.** Does the candidate ask what happens when a job fails, runs twice, or receives late data?
- **Communication.** Can a non-specialist follow the explanation?

## A worked example of pacing

A 45-minute SQL round with three questions leaves about 12 minutes for each, after 5 minutes of introductions and 4 minutes of candidate questions. A candidate who spends 25 minutes on the first question cannot finish. A practical rule is to state the approach in the first 2 minutes, write for 7, and test against a three-row example for 3.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-interview-loop) follows the same round structure, one module per skill.
- [Four-week plan](../cheat-sheets/four-week-plan.md) turns this loop into a study schedule.
- [Grokking Modern Behavioral Interview](https://www.designgurus.io/course/grokking-behavioral-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-interview-loop) covers the behavioral round in full.
- Free article: [Data engineering interviews](https://www.designgurus.io/blog/data-engineering-interviews-how-to-stand-out-in-a-competitive-field?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-interview-loop), which explains what separates strong candidates.
