# Probe 4 — Reproducibility

Could a competent reader reproduce the paper's reported results from what's
released? Audit the notebook (paper + repo + website) against the standard
reproducibility checklist.

For each item below, state whether it's present, partial, or missing. Cite
the specific source where you found it (or note that you searched and didn't
find it).

1. **Code released?** Is the repo present and does it contain the actual
   experiment code, or just a stub / readme?
2. **Data released?** Are the datasets used in evaluation included or linked?
   If linked externally, is the link reachable from the notebook?
3. **Model artifacts?** If the paper trains a model, are weights / checkpoints
   released, or only training code?
4. **Random seeds?** Are seeds for stochastic procedures fixed in the code or
   documented in the paper?
5. **Hyperparameters?** Are all hyperparameters used in the reported results
   either in a config file in the repo or stated in the paper?
6. **Compute environment?** Hardware, framework versions, OS — is enough
   stated to reproduce?
7. **Eval harness?** Is the script that produces the reported tables/figures
   included? Can you trace from a number in a table back to a script that
   produces it?
8. **Statistical tests?** If the paper claims significance, are the tests run
   in the repo and the test scripts present?

Output format per item:

```
### N. <Item name>
Status: PRESENT / PARTIAL / MISSING
Evidence: <file path or section, or "no evidence found">
Notes: <one sentence>
```

End with a verdict on overall reproducibility:
- FULL — every item PRESENT.
- LIKELY — small gaps but the headline experiments are reproducible.
- WEAK — major gaps; reproduction would require contacting authors.
- INFEASIBLE — too much is missing.
