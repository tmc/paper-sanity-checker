---
name: paper-sanity-checker
description: "Sanity-check an arXiv paper against its own released evidence by mirroring the paper, its companion code repos, its companion websites, and (optionally) its cited papers into a NotebookLM notebook, priming it as a vigorous reviewer, and running a battery of probes that must cite specific source lines for every finding. Targets paper/code drift, undocumented methodological choices, dataset issues, and reporting selection — i.e. the failure modes that surface when a paper is not independently reviewed."
when_to_use: "User wants to sanity-check, audit, pressure-test, or find issues in an arXiv paper. Triggers on phrases like 'sanity check this paper', 'is this paper correct', 'find issues in', 'pressure test', 'fact check', or when the user supplies an arXiv id/URL with critical intent. Do NOT use for plain summarization — use the nlm skill directly for that."
allowed-tools: Bash(arxiv:*), Bash(nlm:*), Bash(git clone:*), Bash(git rev-parse:*), Bash(curl:*), Bash(html2md:*), Bash(grep:*), Bash(sed:*), Bash(awk:*), Bash(sort:*), Bash(find:*), Bash(ls:*), Bash(mkdir:*), Bash(cat:*), Bash(rm:*), Read, Write, Edit, Glob, Grep
---

# paper-sanity-checker — sanity-check a paper against its own evidence

Goal: pressure-test a paper (given an arXiv id or URL) for factual errors,
unsupported claims, and prose/data mismatches. Do this by **assembling a corpus
of verifiable sources** (paper + linked code + linked websites + optionally
cited papers) into a NotebookLM notebook, then running probes that must cite
specific source lines for every finding.

NotebookLM can only verify against sources it has. Curating the corpus is the
experiment — do not skip steps 2–4.

## Inputs

- An arXiv id (`<arxiv-id>`) or URL (`https://arxiv.org/abs/<arxiv-id>`).
  Strip any `vN` version suffix.
- If the user hasn't provided one, ask before starting any work.

The skill assumes `arxiv`, `nlm`, `git`, `curl`, and `html2md` are on
`$PATH`. Tool-specific concerns (auth state, quotas, retry behavior) belong
to those tools' own skills — surface failures to the user and let those
skills handle the recovery.

**Output location.** Intermediate artifacts and the final report go under
`~/.paper-sanity-checks/<slug>/`, *not* the current working directory.
This survives `cd` between runs, keeps audits from polluting unrelated
projects, and groups all paper audits in one obvious place. The slug is
the arXiv id (no `v` suffix). The user may override with the
`PAPER_SANITY_HOME` environment variable.

## Phases

Run phases sequentially. Pause for user input at the explicit checkpoints —
don't blast through.

---

### Phase 1 — Resolve the paper

```bash
ID=<id>                                                   # normalized, no vN
WORK="${PAPER_SANITY_HOME:-$HOME/.paper-sanity-checks}/$ID"
mkdir -p "$WORK"
arxiv fetch "$ID"                                         # populates ~/.cache/arxiv/src/<YYMM>/<id>
arxiv get "$ID"                                           # title, authors, abstract
arxiv pack -bib -o "$WORK/paper.txtar" "$ID"              # flags MUST precede the positional id
```

Tell the user where `$WORK` resolved to (e.g.
`~/.paper-sanity-checks/<arxiv-id>/`) so they know where the artifacts
will land. If `$WORK` already exists from a prior run, *do not* clobber —
the skill should be re-runnable and incremental, picking up where a
previous run left off.

Show the user the title and authors. **Confirm it's the right paper before
spending the rest.** Wrong-paper audits are worse than none.

If `arxiv fetch` fails, show the error and stop. Do not invent the paper from
memory.

---

### Phase 2 — Discover linked artifacts

Read the TeX sources and extract every URL:

```bash
SRC=$HOME/.cache/arxiv/src/${ID%%.*}/$ID
grep -hoE 'https?://[^[:space:]}{,)\\"]+' "$SRC"/*.tex \
  | sed -E 's/[.,;:]+$//' \
  | sort -u > "$WORK/links.txt"
```

Classify each link into:
- **github** — `github.com/<owner>/<repo>` (normalize: strip `/tree/...`,
  `/blob/...`, trailing `.git`, dedupe)
- **arxiv** — `arxiv.org/abs/<id>`
- **web** — everything else

Write the classification to `$WORK/links.tsv` (`kind<TAB>url`).

Show the user the full classified list, then ask which to mirror. The
default policy is **narrow** — only artifacts the paper presents as its own
deliverables, not arbitrary references it happens to link.

For each github link, classify by the surrounding TeX context:

- **PRIMARY** — appears in a sentence with "Code:", "We release …", "Our
  implementation", "Companion repo", or in an "Artifacts" / "Reproducibility"
  section. These are what the paper offers as its evidence. Mirror by
  default.
- **ILLUSTRATIVE** — appears as an example, motivating case, or background
  reference ("e.g., …", "see …", "we probed against repositories at …",
  "the official writeups at …"). These are *targets of the experiment*, not
  *evidence for the experiment's conclusions*. Skip by default; offer to
  the user as opt-in. Including them in the corpus invites NotebookLM to
  conflate "the data the paper analyzed" with "the paper's own evidence".
- **OTHER** — boilerplate links (license badges, CI status, generic tools).
  Skip.

For web links, only mirror the explicitly named companion site/project
page. Skip blog mentions, social links, and generic references unless the
user opts in.

For cited arXiv papers, ask. Default no — papers can cite hundreds, and
the corpus cap matters more than coverage.

Show the proposed mirror set with reasons next to each item, then **wait
for the user's selection before proceeding**. Make it easy to override:
"primary only" / "primary + illustrative" / "all" / explicit list.

---

### Phase 3 — Mirror artifacts

Idempotent — skip anything already on disk. Always continue past individual
failures and record them so the report can flag the gap.

For each selected link:

**github** — shallow clone, with idempotent recovery from half-clones:
```bash
slug=$(echo "$url" | sed -E 's#^https?://github\.com/##; s#/#__#g')
dest="$WORK/mirror/repos/$slug"
# A half-clone leaves only .git/ behind. Detect by checking whether the work
# tree has any non-.git entry; if not, wipe and reclone.
if [[ -d "$dest" ]] && [[ -z "$(find "$dest" -mindepth 1 -maxdepth 1 ! -name .git -print -quit)" ]]; then
  rm -rf "$dest"
fi
if [[ ! -d "$dest" ]]; then
  if ! git clone --depth=1 --quiet "$url" "$dest" 2>/dev/null; then
    rm -rf "$dest"
    echo "FAILED clone: $url" >> "$WORK/mirror/errors.log"
  fi
fi
```

**web** — fetch + html2md, with SPA-shell detection:
```bash
hostpath=$(echo "$url" | sed -E 's#^https?://##; s#[?#].*$##; s#/$##')
dest="$WORK/mirror/web/$hostpath.md"
mkdir -p "$(dirname "$dest")"
curl -fsSL --max-time 30 "$url" 2>/dev/null | html2md > "$dest" \
  || { rm -f "$dest"; echo "FAILED fetch: $url" >> "$WORK/mirror/errors.log"; }
# JS-rendered SPAs hand back a 1-2KB shell that html2md reduces to just the
# <title>. A bare `test -s` lets these slip through (e.g. 40 bytes is
# "non-empty" but useless). Treat anything under 256 bytes as suspicious;
# also treat "title only, no body prose" as suspicious.
if [[ -s "$dest" ]]; then
  size=$(wc -c < "$dest" | tr -d ' ')
  body_lines=$(grep -cE '[[:alpha:]]{20,}' "$dest" || true)
  if (( size < 256 )) || (( body_lines < 2 )); then
    echo "SUSPICIOUS web mirror (likely JS-SPA shell, size=${size}B body_lines=${body_lines}): $url" >> "$WORK/mirror/errors.log"
    # Keep the file — title alone is sometimes useful — but flag it loudly.
  fi
fi
```

When a web mirror is flagged SUSPICIOUS, *also* check whether the same
content might be available inside one of the mirrored repos (companion sites
are often Vite/React apps with the prose in a `website/` or `docs/`
directory). If so, recommend the user drop the web mirror — the repo source
is strictly better.

**arxiv** (cited papers, if user opted in):
```bash
cid=$(echo "$url" | sed -E 's#^https?://arxiv\.org/abs/##; s#v[0-9]+$##')
arxiv fetch "$cid" >/dev/null 2>&1 \
  || echo "FAILED: arxiv $cid" >> "$WORK/mirror/errors.log"
```

After mirroring, show the user a summary:
- N repos cloned (list owner/repo)
- N web pages mirrored
- N arxiv papers fetched
- Contents of `$WORK/mirror/errors.log` if non-empty

If any websites came back empty, tell the user they were likely JS-rendered
SPAs and suggest reviewing them manually.

---

### Phase 4 — Build the notebook

Create the notebook. Defer to the `nlm` skill for any error handling; surface
its message to the user and wait for them to act on it (free a slot, refresh
auth, etc.) — don't try to repair the account state from this skill.
```bash
NB=$(nlm notebook create "sanity: $ID — <short-title>") \
  || { echo "create failed; see error above and resolve before retrying"; exit 1; }
echo "$NB" > "$WORK/notebook-id"
```

If `nlm` reports the account is at the notebook cap, do not silently reuse
an existing notebook — that pollutes unrelated work. Wait for the user to
free a slot, then re-run this phase.

Upload the paper first so its name is unambiguous:
```bash
nlm source add --name "paper:$ID" "$NB" "$WORK/paper.txtar"
```

Each cloned repo as one txtar source. Defer to the `nlm` skill for the
upload mechanics (chunk sizing, exclude patterns for ML repos, retry
classification) — don't reinvent that knowledge here.
```bash
for d in "$WORK"/mirror/repos/*/; do
  name=$(basename "$d" | sed 's/__/\//')
  nlm sync --name "repo:$name" "$NB" "$d" \
    || echo "sync failed: $name" >> "$WORK/mirror/errors.log"
done
```

After all uploads, list the notebook to confirm coverage and show the user
both the source list and any errors from `mirror/errors.log` before running
probes:
```bash
nlm source list "$NB"
cat "$WORK/mirror/errors.log" 2>/dev/null
```

Partial uploads are acceptable for the run if the gaps are clearly data
files (CSVs, large JSON dumps) rather than source code. Note any gap so
probe results can flag it as a known limitation rather than a verdict.

Each web markdown:
```bash
while IFS= read -r f; do
  rel=${f#"$WORK"/mirror/web/}
  nlm source add --name "web:${rel%.md}" "$NB" "$f"
done < <(find "$WORK/mirror/web" -type f -name '*.md' 2>/dev/null)
```

Each cited arxiv (only those the user opted into):
```bash
for cid in <list>; do
  d="$HOME/.cache/arxiv/src/${cid%%.*}/$cid"
  nlm sync --name "arxiv:$cid" "$NB" "$d"
done
```

Set the chat instructions so every probe runs under the reviewer persona:
```bash
nlm chat instructions set "$NB" "$(cat skills/paper-sanity-checker/chat-instructions.md)"
```

Show the user `nlm source list "$NB"` and ask whether to add or remove
sources before running probes. **This is the corpus checkpoint — don't skip
it.**

---

### Phase 5 — Run probes

For each probe under `skills/paper-sanity-checker/prompts/`, run a fresh
one-shot chat (no conversation id — probes are independent). Use
`nlm generate-chat` (the non-interactive primitive — `nlm chat` opens a
conversation we don't need) with `--citations stream` so every claim must
cite a source.

Probes are independent, so run them in parallel — serial on 8 probes adds
~5–10 minutes of needless wall-clock. Cap parallelism modestly (4) to stay
inside NotebookLM's per-account rate envelope:

```bash
mkdir -p "$WORK/out"
SKILL_DIR=<absolute path to this skill dir>   # resolve before launching probes

run_probe() {
  local p="$1"
  local name; name=$(basename "$p" .md)
  echo "running probe: $name" >&2
  if ! nlm generate-chat --citations stream "$NB" "$(cat "$p")" \
       > "$WORK/out/$name.md"; then
    echo "probe failed: $name" >&2
  fi
}
export -f run_probe
export NB WORK

find "$SKILL_DIR/prompts" -maxdepth 1 -type f -name '*.md' -print0 \
  | xargs -0 -n1 -P4 bash -c 'run_probe "$0"'
```

If `xargs -P` isn't available or the user prefers serial output (e.g.
debugging a single probe), fall back to a `for` loop — same body, just
sequential.

Probes you'll run (curated under `prompts/`):
1. `01-claims-vs-evidence` — every headline claim, with verdict + cited line.
2. `02-numbers-vs-tables` — prose numbers vs. tables vs. raw repo data.
3. `03-code-vs-paper` — does the released code actually do what the paper
   describes? Are the algorithms in the repo what the methods section
   describes?
4. `04-reproducibility` — could a competent reader reproduce results from
   what's released? Specifically flag missing seeds, configs, datasets,
   eval harnesses.
5. `05-citation-misuse` — does each cited paper actually support the claim
   it's attached to (when the cited paper was mirrored)?
6. `06-internal-consistency` — notation drift, definition mismatches across
   sections, contradictions between abstract/intro/body/conclusion.

---

### Phase 6 — Aggregate and hand off

Build `$WORK/report.md` with this structure:

```markdown
# Sanity report: <title> (arxiv:<id>)

## Summary
<bulleted list of every UNSUPPORTED / CONTRADICTED / MISMATCH finding,
each with a one-line description and a link to the per-probe section below>

## Corpus
- paper:<id>
- repo:<owner>/<repo> (× N)
- web:<host><path> (× N)
- arxiv:<cited-id> (× N)
- gaps: <contents of mirror/errors.log, if any>

## Probe 1 — Claims vs. evidence
<contents of out/01-claims-vs-evidence.md>

## Probe 2 — Numbers vs. tables
... etc.
```

Then tell the user:
- Notebook id (and how to open it: `nlm chat $NB`)
- Path to `$WORK/report.md`
- Suggested follow-up: `nlm chat $NB` to drill into any flagged finding —
  the chat instructions are already set to the reviewer persona, so they
  can ask follow-ups directly without re-priming.

---

## Anti-patterns (do not do these)

- **Don't summarize from prior knowledge.** Every finding must cite a
  specific source line in the notebook. If a probe answer doesn't cite,
  re-run with "cite the specific source line; if you can't, say so."
- **Don't auto-clone arbitrary URLs.** Confirm the mirror set with the user.
  Some papers link to private infra or huge datasets.
- **Don't skip the corpus checkpoint.** The corpus is the experiment.
- **Don't conflate "paper says X" with "X is true."** The probes ask whether
  the paper's *evidence* supports its *claims*.
- **Don't write findings as soft critique.** "This could be clarified" is
  not a sanity finding. Either evidence supports the claim or it doesn't.
