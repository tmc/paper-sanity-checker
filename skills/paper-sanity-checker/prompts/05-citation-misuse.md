# Probe 5 — Citation misuse

Citations are evidence. A paper that cites X for claim Y is asserting that X
supports Y. When the cited paper is in this notebook, you can check whether
it actually does.

Scope: only run this probe on cited papers that have been mirrored into this
notebook (look for sources prefixed with `arxiv:`). Skip citations whose
target isn't in the notebook — note them as OUT OF SCOPE.

For each cited paper present in the notebook:

1. Find every place in the paper-under-review where this work is cited.
   Quote the surrounding sentence with section reference.
2. Open the cited paper. Look for the specific claim or result that's being
   attributed to it.
3. Check: does the cited paper actually say what it's being cited for?
4. Verdict per citation site:
   - **CORRECT** — cited paper supports the claim attributed to it.
   - **OVERREACH** — cited paper says something weaker; the citing paper
     overstates it.
   - **MISCITED** — cited paper does not support this claim at all.
   - **WRONG ATTRIBUTION** — claim is made by a different work; this is the
     wrong citation.

Output format per citation site:

```
### Citation: <ref-key or first-author-year>
Citing context: > <quoted sentence from paper-under-review> — §<section>
Cited paper says: > <quoted line from cited paper>
Verdict: <CORRECT / OVERREACH / MISCITED / WRONG ATTRIBUTION>
```

End with a summary list of all OVERREACH and MISCITED findings.

If no cited papers are mirrored in this notebook, output:
"No cited papers are present as sources; this probe cannot run."
