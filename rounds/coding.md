# The Coding Round

The coding round for a data engineer usually uses Python on data, not abstract algorithm puzzles. The candidate receives records, a file, or a stream of events and must parse, group, or combine them. Interviewers look for correct handling of messy input and for awareness of memory, in addition to a working solution.

## The five problem shapes

| Shape | A typical question | The structure to reach for |
|---|---|---|
| Counting and grouping | Total shipped weight per warehouse | `dict`, `collections.defaultdict`, `collections.Counter` |
| Parsing | Read a CSV file where some fields contain commas | The `csv` module, never `split(",")` |
| Top K | The ten most frequent addresses in a log | `Counter.most_common`, or `heapq` for large input |
| Intervals | Merge overlapping parking sessions | Sort by start, then one pass |
| Large files | Process a 40 gigabyte file with 4 gigabytes of memory | Generators and chunked reads |

## Worked example: top K from a log

Each line of a web server log starts with a client address. Return the three most frequent addresses.

```python
from collections import Counter

def top_addresses(path, k=3):
    counts = Counter()
    with open(path, encoding="utf-8") as lines:
        for line in lines:
            parts = line.split(maxsplit=1)
            if not parts:
                continue  # skip blank lines
            counts[parts[0]] += 1
    return counts.most_common(k)
```

Three points are worth saying aloud. The file is read one line at a time, so memory depends on the number of distinct addresses, not on the file size. Blank lines are skipped instead of raising an error. If the distinct addresses also exceed memory, the next step is to split the file by a hash of the address and count each part separately.

## Worked example: merging intervals

```python
def merge(intervals):
    merged = []
    for start, end in sorted(intervals):
        if merged and start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged
```

For `[(1, 4), (2, 6), (8, 9)]` the result is `[[1, 6], [8, 9]]`. Sorting costs O(n log n), and the pass is linear.

## What interviewers add after the first solution

- **Bad rows.** "Line 5,000 has a missing field." Decide among three options: reject the file, repair the row, or write the row to a quarantine file and continue.
- **Scale.** "The file is now 100 times larger." Move from lists to generators, and from one sort in memory to sorted chunks that are merged.
- **Reruns.** "The job failed halfway." Write to a temporary file and rename it at the end, so a partial output is never read.
- **Time zones.** "Events are in local time." Convert to Coordinated Universal Time (UTC) at the first step and store the original offset.

## Habits that score well

1. Restate the input format and ask about its size before writing code.
2. Write a small test input with one blank line, one duplicate, and one malformed row.
3. Name the time and memory cost of the solution.
4. Use standard library tools. Knowing `Counter`, `defaultdict`, `heapq`, `itertools.groupby`, and `csv` saves several minutes.

One trap deserves a mention: `itertools.groupby` groups only adjacent equal keys, so the input must be sorted by that key first.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-coding) has a Coding for Data Work module with six lessons on these shapes.
- [Idempotency and backfills](../concepts/idempotency-and-backfills.md) explains the rerun follow-up in depth.
- [Question bank](../cheat-sheets/question-bank.md) lists ten coding questions to practice.
