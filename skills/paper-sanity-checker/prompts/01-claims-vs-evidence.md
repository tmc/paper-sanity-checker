# Probe 1 — Claims vs. evidence

Enumerate every headline claim from the paper's abstract, introduction, and
conclusion. For each one:

1. Quote the claim verbatim, with section reference.
2. Identify what evidence the paper offers for it (specific table, figure,
   appendix, repo artifact, or cited source).
3. Open that evidence source in the notebook and check it. Quote the lines
   that support or contradict the claim.
4. Verdict (SUPPORTED / PARTIALLY SUPPORTED / UNSUPPORTED / CONTRADICTED /
   OUT OF SCOPE).

Be especially skeptical of:
- Claims about generality ("works on any model", "any language", "in general")
- Claims phrased as observations ("we observe that...") with no table or plot
- Claims about "significant" improvements without statistical tests
- Claims about novelty ("first to...") — are the cited related works mirrored?

Output format per claim:

```
### Claim N
> <quoted claim> — §<section>

Evidence cited: <table/figure/repo path/URL>
Quoted evidence: > <quote from the cited source>
Verdict: <one of the labels>
Notes: <one sentence on the gap, if any>
```
