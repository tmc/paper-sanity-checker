# paper-sanity-checker

A Claude Code skill that sanity-checks an arXiv paper against its own released
evidence by mirroring its full evidence trail (paper + companion code +
companion websites + optionally cited papers) into a NotebookLM notebook,
priming the notebook as a vigorous reviewer, and running a battery of probes
that must cite specific source lines for every finding.

The skill targets the failure modes that surface when a paper is not
independently reviewed: paper/code drift, undocumented methodological choices,
dataset issues, reporting selection, and companion-artifact inconsistencies.
It does *not* fact-check claims against the world — only against the paper's
own ecosystem.

## Layout

```
skills/paper-sanity-checker/
  SKILL.md                  # the skill — phases, commands, anti-patterns
  chat-instructions.md      # reviewer persona for `nlm chat instructions set`
  prompts/                  # one file per probe, fed to `nlm chat -f`
    01-claims-vs-evidence.md
    02-numbers-vs-tables.md
    03-code-vs-paper.md
    04-reproducibility.md
    05-citation-misuse.md
    06-internal-consistency.md
    07-sensitivity-and-selection.md
    08-dataset-audit.md

~/.paper-sanity-checks/<arxiv-id>/    # per-paper artifacts (default home;
                                      # override with $PAPER_SANITY_HOME)
  paper.txtar
  links.txt
  links.tsv
  mirror/repos/<owner>__<repo>/
  mirror/web/<host><path>.md
  notebook-id
  out/<probe>.md
  report.md
```

## Requirements

- `arxiv`    (https://github.com/tmc/arxiv)         — paper fetch + pack
- `nlm`      (https://github.com/tmc/nlm)           — NotebookLM CLI; run `nlm auth` first
- `churl`    (https://github.com/tmc/cdp/cmd/churl) — Chrome-rendered fetch + recursive mirror for SPA companion sites
- `html2md`  (https://github.com/tmc/misc/html2md)  — HTML → markdown for NotebookLM ingestion
- `git`, `curl`                                     — repo cloning + raw fetch fallback

## Use

When working in a Claude Code session that has this skill discoverable, ask
Claude to "sanity-check arxiv:<id>" or "pressure-test https://arxiv.org/abs/<id>"
and the `paper-sanity-checker` skill activates. Artifacts land in
`~/.paper-sanity-checks/<id>/`.
