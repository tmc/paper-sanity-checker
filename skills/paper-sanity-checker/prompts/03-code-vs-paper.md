# Probe 3 — Code matches the paper's description

The paper describes a method, algorithm, or system. The companion repo (if
present in the notebook) is the implementation. Check whether they match.

For each algorithm, formula, or system component the paper describes:

1. Quote the paper's description with section reference (especially: pseudo-
   code blocks, algorithm boxes, equations that define a procedure, and
   architecture diagrams referenced in prose).
2. Find the corresponding implementation in the repo. Quote the relevant code
   with file path and line numbers.
3. Compare: does the code do what the paper says it does? In particular:
   - Same inputs / outputs / control flow?
   - Same hyperparameter values (defaults, constants)?
   - Same algorithmic steps in the same order?
   - Are there steps in the code that the paper omits, or vice versa?
4. Verdict: MATCH / DRIFT / CONTRADICTED / NOT IN REPO.

Pay particular attention to code paths the paper is least likely to discuss
in detail but which directly shape the headline numbers:

- **Scoring, aggregation, and reporting code.** How are per-item scores
  combined into per-bucket scores? How are per-bucket scores combined into
  the headline number? Are there clamps, floors, ceilings, or tie-breaks
  applied that the paper doesn't mention?
- **Preprocessing and filtering code.** What inputs are dropped, transformed,
  or imputed before reaching the algorithm the paper describes?
- **Default constants.** When a function takes a parameter with a default,
  is that default the value the paper claims to use, or something else?
- **Branching on edge cases.** What does the code do when the algorithm
  hits NaN, an empty result, a timeout, a parse failure? Silent fallbacks
  to safe values can change reported numbers without changing prose.
- **The exact entrypoint that produced the reported tables/figures.** Trace
  from a number in a table back to the script that prints it. Anything
  between the algorithm-as-described and the printed number is a candidate
  for paper/code drift.

In addition to the matches-vs-claims pass above, run an explicit
**negations pass** — paper/code drift hides best in claims of the form
"we do *not* do X":

1. List every paper-text claim of the form "we do not", "without",
   "no <X> is applied", "scores are not <floored | clipped | dropped |
   normalized | smoothed | filtered | imputed | thresholded>", "results
   are unmodified", etc. Quote each with section reference.
2. For each negation, search the repo for evidence that the operation
   *is* nonetheless performed somewhere on the path to the reported
   number. Look in scoring, aggregation, reporting, and post-processing
   code — not just the named algorithm.
3. Verdict per negation:
   - **CONFIRMED** — code path matches the paper's negation.
   - **CONTRADICTED** — code does the operation the paper says it doesn't.
     This is among the most damaging findings the probe can produce.
4. Cross-check duplicated implementations of the same component (e.g.
   multiple scripts that score model outputs) — when there are several,
   confirm they agree with each other AND with the paper's claim.

DRIFT means the code implements something close but not identical (e.g.
different default constant, additional preprocessing step, slightly different
loss). CONTRADICTED means the code does something materially different from
what the paper claims.

Output format:

```
### Component
Paper: > <quoted description> — §<section>
Code: > <quoted code> — <repo>/<path>:<line>

Differences: <bulleted list, or "none">
Verdict: <MATCH / DRIFT / CONTRADICTED / NOT IN REPO>
```

End with a summary list of CONTRADICTED findings.
