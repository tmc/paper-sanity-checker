# Probe 6 — Internal consistency across the paper and its companion artifacts

A research artifact is internally inconsistent when the same claim is made
two ways and the two ways disagree. Anywhere a fact appears in both the
paper and a companion artifact (repo README, repo config, companion website,
docs, blog post) is a place where drift can accumulate without any single
statement being wrong. Find drift wherever the same content appears twice.

Check across:
- the paper text itself (sections, figures, tables, captions)
- repo READMEs and top-level docs
- repo config files, schemas, and example configs
- the companion website / project page
- duplicated assets that should agree (e.g. multiple prompt templates that
  serve the same role; the same dataset described in two places)

For the paper alone, check:

1. **Notation drift.** A symbol introduced in §2 used differently in §4. A
   variable defined as "the number of X" used as "the rate of X" elsewhere.
   Quote both uses.
2. **Definition mismatches.** A term defined formally in one section used
   informally (and incompatibly) in another. Quote both.
3. **Abstract vs. body.** Does the abstract claim something stronger than
   the body shows? (Common: abstract says "we demonstrate X"; body shows X
   only on a subset.)
4. **Body vs. conclusion.** Does the conclusion overgeneralize what the
   experiments actually established? Quote the specific overreach.
5. **Method vs. results.** Does the methods section describe a procedure
   that, applied to the stated inputs, would produce the reported outputs?
   Or is there a step missing / inconsistent?
6. **Figure / table consistency.** Do figures and their captions agree?
   Do tables and the prose that references them quote the same numbers?
   (Probe 2 covers prose-vs-table numerically; here, focus on
   *interpretation* mismatches: e.g., prose calls Figure 3 a comparison
   while the figure shows an ablation.)

For paper vs. companion artifacts, additionally check:

7. **Paper vs. README/docs.** Does the README describe the method, the
   dataset, or the evaluation in a way the paper text contradicts? Quote
   both.
8. **Paper vs. companion website.** Project pages often summarize results
   and sometimes do so with numbers, plots, or captions that disagree with
   the paper version. Note any discrepancy. Also flag companion-site
   *quality* signals that suggest the artifacts weren't reviewed: terms
   defined but used nowhere, table headings that don't match table
   contents, structural duplication.
9. **Duplicated assets that should be identical.** When the same prompt,
   schema, config, or formula appears in multiple files, they should agree.
   Quote any pair that differs and identify which (if any) is canonical.

   Don't rely on noticing duplicates passively — *enumerate* them. For
   each asset class likely to be duplicated in the repo, run an explicit
   listing step before judging. The general shape:
   - Pick an asset class (a kind of thing the paper or pipeline depends
     on — a templated string, a numeric constant, a config, a formula).
   - List every file that contains an instance of that class.
   - If the list has more than one entry, diff the instances.
   - Any disagreement is a finding.

   Each enumeration produces a list. If the list has length 1, no
   duplication exists. If length ≥ 2, every pair must agree or the
   disagreement is a finding.

Output format per finding:

```
### Finding
Type: <NOTATION / DEFINITION / ABSTRACT_OVERREACH / CONCLUSION_OVERREACH /
       METHOD_GAP / FIG_TABLE / PAPER_VS_README / PAPER_VS_WEBSITE /
       DUPLICATED_ASSET_DRIFT>
Quote A: > <first quote with location: §<section> or <path>:<line> or URL>
Quote B: > <second quote with location>
Conflict: <one sentence describing the inconsistency>
```

If no inconsistencies are found, say so explicitly — do not pad with weak
findings.
