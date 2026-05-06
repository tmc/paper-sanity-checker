# Probe 7 — Methodological sensitivity and selection effects

A headline result is only as strong as its insensitivity to choices the
paper made along the way. This probe targets two failure modes:

- **Sensitivity** — the paper picks one value for a methodological knob
  (a threshold, a default, a tie-break, a clamp, a metric variant, an
  inclusion filter, a hyperparameter). What happens to the headline number
  if a different reasonable value were used? If the paper doesn't say,
  that's a finding.
- **Selection** — the paper reports one number when multiple were
  available (e.g. best-of-N runs, max over variants, a single seed,
  a single evaluator, a single subset). Is the reported number
  representative or selectively presented?

For each piece of these — and you do not need a complete enumeration; ten
of the most load-bearing choices is more useful than thirty trivial ones —
identify:

1. **The choice.** A specific methodological decision that shapes a
   reported result. Quote where it's made (paper section, code constant,
   config value).
2. **The justification, if any.** Did the paper explain *why* this value
   was chosen? Quote the justification, or note "no justification given."
3. **Plausible alternatives.** What other values would be defensible? List
   1–3 alternatives a careful reviewer would consider.
4. **Evidence of sensitivity.** Does the repo allow varying this knob? Are
   there ablations or sensitivity tables that report results under the
   alternatives? If so, quote them. If not, note the absence — sensitivity
   that isn't measured is sensitivity that isn't ruled out.
5. **Selection check** (where applicable). When the paper reports a single
   number, does the repo or appendix contain the underlying distribution
   the number was selected from? If reported = max(variants), say so.
6. **Peak-vs-aggregate check.** When the same subject is evaluated under
   multiple variants (different settings, seeds, prompts, configurations,
   inference modes, judge versions), how does the paper roll those
   variants up into the reported number?
   - **mean** / **median** — averaging exposes variance.
   - **max** / **best** / **selected variant** — peak-reporting hides
     variance.
   If the rollup is a peak, find the variant-level data in the repo and
   quote the dispersion the headline collapses. A correct number for
   one variant is still a misleading headline if the dispersion is
   omitted; flag the omission.

Verdict per choice:
- **JUSTIFIED & ROBUST** — choice is explained and ablations show the
  result is insensitive.
- **JUSTIFIED, NOT TESTED** — explained but no sensitivity check.
- **UNJUSTIFIED, NOT TESTED** — neither explained nor tested.
- **SELECTIVE** — the reported number is selected from a wider set without
  acknowledging it.
- **MATERIAL** — repo or paper data shows the result is materially
  different under a reasonable alternative. Quote both numbers.

Output format per choice:

```
### Choice
What: <one-sentence description>
Where: <§<section> | <repo path>:<line> | <config>>
Justification: <quoted, or "none">
Alternatives: <list>
Sensitivity test in repo/paper: <quoted, or "absent">
Verdict: <one of the labels>
```

End with a summary list of MATERIAL and SELECTIVE findings — these are the
ones that change the paper's contribution if true.

Anti-patterns:
- Don't list every hyperparameter. Focus on choices that gate the headline.
- Don't speculate about what would happen under alternatives if the repo
  doesn't let you check. "If we changed X, Y might change" without
  evidence is not a finding — note the choice as UNJUSTIFIED, NOT TESTED.
