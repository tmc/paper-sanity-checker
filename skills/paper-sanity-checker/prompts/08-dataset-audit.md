# Probe 8 — Dataset / benchmark / corpus audit

A paper's results inherit the quality of its inputs. If the dataset, eval
set, or training corpus has systematic problems, the headline numbers
quantify the dataset's flaws as much as the method's strengths. Audit the
inputs.

Scope: this probe applies to whatever the paper treats as ground-truth
input — a benchmark it introduces, an eval set it uses, a curated corpus,
human annotations, gold labels, etc. Skip if the paper has no such artifact
(e.g. pure theory papers).

**Independence guardrail.** The authors' own audits, self-reported
flagged-probe lists, and "we removed bad examples" sections are the
*subject* of this probe, not its evidence. Do not rely on them. Draw your
own sample (favoring the hardest, rarest, most-likely-ambiguous slice) and
report what *you* find when you check the labels against authoritative
sources. If the only evidence you can quote is "the authors say they
audited this," the verdict is OUT OF SCOPE — not "verified."

When verification requires capabilities you don't have in this notebook
(re-querying Wikidata / OpenAlex / Wikipedia for entity-collision counts,
running an LLM judge against a held-out sample, reproducing an annotation
pipeline), say so explicitly: list the specific verification step as TODO
(needs <web | live API | execution>). Do not silently substitute the
authors' audit as a stand-in for that check.

For the dataset in question:

1. **Composition.** How was it built? What sources, filters, splits, and
   sample-selection rules produced it? Quote the description from the
   paper, then check the repo for the actual construction script — does
   the script match the description?
2. **Diversity and coverage.** Where does the dataset draw from, and where
   doesn't it? Are some buckets (difficulty, topic, language, time, source
   type) much more represented than others? If the paper claims
   generality, does the dataset support that claim?
3. **Label correctness.** Sample 10–30 entries from the dataset and
   verify the label against an authoritative source where one exists in
   the corpus (paper citations, repo metadata, named external sources).
   Report the rate of suspect labels and quote a few examples.

   Draw the sample from at least two different slices — labeling errors
   cluster differently in each:
   - The hardest / rarest / most-likely-ambiguous slice. Errors at the
     tail compound with the difficulty signal.
   - The slice where evaluated systems *underperformed relative to
     their expected level* (e.g. a system that gets most items right
     but misses a specific entry). When evaluated systems consistently
     fail an item that lower-performing systems also fail, that's a
     mild signal the item itself may be mis-labeled — worth checking.

   When the sample is too small to ground a population-level rate
   (e.g. you sample 10 of 1000), report the sample rate explicitly
   ("2/10 ambiguous in this sample") rather than extrapolating.
4. **Ambiguity and underspecification.** Are there entries where multiple
   answers are reasonable but only one is marked correct? Multiple
   entities sharing a name? Time-dependent answers without a time anchor?
   Multi-valued targets reduced to a single value? Estimate the rate.
5. **Leakage and circularity.** Is any of the dataset present in training
   sets the evaluated systems likely saw? Is the dataset constructed
   *using* one of the systems being evaluated (e.g. one model generated
   the questions another model is graded on)? Note any such risk.
6. **Documentation completeness.** Is enough of the construction process
   documented that a reader could rebuild the dataset and predict whether
   their rebuild would be similar? Or is the dataset effectively a magic
   constant?

Output format:

```
### Composition and construction
<paragraph + verdict: MATCHES_DESCRIPTION / DRIFT / UNDOCUMENTED>

### Coverage gaps
<bulleted list of buckets that are under- or over-represented relative to
the paper's claimed scope>

### Label quality (sample of N)
Sampled: <how the sample was drawn>
Suspect rate: <X / N>, breakdown by category:
- wrong: <quoted examples with citations>
- ambiguous: <quoted examples with citations>
- underspecified / time-dependent: <quoted examples>
- multi-valued reduced to single: <quoted examples>

### Leakage / circularity
<note any concerns with citations to where in the corpus this is visible>

### Documentation
<verdict: SUFFICIENT_TO_REBUILD / GAPS / OPAQUE>

### Bottom line
<one paragraph: how much do these issues affect the headline result, and
which issues if any would change the paper's claims if corrected>
```

Anti-patterns:
- Don't grade the dataset against an idealized standard. The question is
  whether its problems are large enough to affect the *paper's* claims.
- Don't extrapolate a suspect rate from 3 examples. If you sample N=10
  and find 2 issues, report "2/10" — not "20% of the dataset is bad."
- If the dataset isn't accessible from the notebook (not in the repo,
  external link unreachable), say so explicitly. Do not invent a sample.
