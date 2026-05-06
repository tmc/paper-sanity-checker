# Probe 2 — Numbers in prose vs. numbers in tables and code

The paper's body quotes specific numbers: percentages, counts, model sizes,
deltas, p-values. Cross-check every numeric claim against the source-of-truth
in the notebook.

For each numeric claim:

1. Quote the prose with section reference and the exact number quoted.
2. Find the corresponding number in:
   - A table or figure caption in the paper itself, or
   - A CSV / JSON / log file in the companion code repo, or
   - A table or chart on the companion website, or
   - A code constant or default in the repo.
3. State the source-of-truth value verbatim with its location.
4. Verdict: MATCH / MISMATCH / NOT FOUND.

Pay extra attention to:
- Inconsistent rounding between prose and tables ("≈42%" vs. table "41.6%"
  is a MATCH; "42%" vs. "47.6%" is a MISMATCH).
- Aggregate stats (means, medians, totals) — recompute from raw data in the
  repo when possible and report any disagreement.
- Sample sizes ("N=...", "evaluated on K models") — confirm the repo or data
  files actually contain that many entries.
- Ranges ("from X to Y") — confirm both endpoints exist in the data.

Output format per number:

```
### Number N
> <quoted prose with number> — §<section>

Source-of-truth: <table/file/path> — value: <number>
Verdict: <MATCH / MISMATCH / NOT FOUND>
```

End with a summary list of all MISMATCH findings.
