# Prompt Builder — working context (read this first)

This file is auto-loaded by Claude Code at session start. It is the single
source of truth for picking up work on this project on any machine, with no
extra explanation needed.

## What this is

**Prompt Builder** is a single-file, static web tool for running a prompt
pipeline: store reusable AI prompts, fill them with each person's / career's
data, copy the assembled prompt out to run in an LLM, paste the result back, and
chain one step's output into the next step's input. No AI API is built in
(copy-paste). Hosted on GitHub Pages.

**Live:** https://sophophobia.github.io/PromptBuilder/

## Hard rules (do not break)

1. **Public site.** This repo and the deployed page are public. Never put any
   client, brand, or product name in user-visible copy, the repo, commit
   messages, or docs. Keep all naming generic ("Prompt Builder").
2. **Single self-contained file.** The whole app is `index.html` (inline CSS +
   vanilla JS, no framework, no build step, no dependencies). Edit it directly.
3. **Pages deploys via GitHub Actions, not Jekyll.** The page contains literal
   `{{ }}` tokens that break Jekyll, so `.nojekyll` is present and
   `.github/workflows/deploy.yml` publishes the repo as-is. Don't switch Pages
   back to the legacy/Jekyll build.
4. **No real data in this repo.** Data lives in a separate private companion
   repo (see Data, below). `data.json` / `history.json` are gitignored here.

## Repo layout

| File | Role |
|---|---|
| `index.html` | **The whole app.** This is what's edited and deployed. |
| `.github/workflows/deploy.yml` | Deploys the repo to Pages on every push to `main`. |
| `.nojekyll` | Disables Jekyll so `{{ }}` tokens survive. |
| `index_legacy.html`, `index_category.html`, `index_test.html`, `indext.html` | Old prototypes, kept for reference. Ignore them. |

## Develop & deploy

```
# edit index.html, then:
git add index.html
git commit -m "..."
git push           # -> GitHub Actions builds & deploys Pages automatically
```

Local preview: the page fetches its data over HTTP, so serve the folder with any
static server (e.g. `npx serve` or `python -m http.server`) rather than opening
`file://`. With no data configured it shows an empty "set GitHub" state, which is
expected locally.

## The tool's concepts (what the code implements)

- **Field** — a named content slot on a person or a career. `kind` is `manual`
  (you type it) or `output` (filled by an AI result via Generate, reusable
  downstream). All people share one field set; all careers share another.
- **Group** — fields are organized into ordered groups; group lists are stored
  and reorderable (drag). Same idea for prompts and recipes.
- **Person → one career** — each person links to exactly one career; career
  fields come from that linked career.
- **Prompt** — plain instruction text (may contain `{{Token}}` markers, which are
  NOT substituted; they are just for readability).
- **Recipe** — a prompt + which person/career fields to **append below it** (as
  labeled `## sections`) + which **output field** the result is saved to.
  `subject` = whether Generate asks you to pick a person (uses their linked
  career) or a career. Mapping an output field of an upstream recipe as input to
  a later recipe is how steps chain.
- **History** — every saved output, with the exact prompt snapshot, who saved it,
  and a timestamp.

## Data (private companion repo)

The tool reads/writes its data from a **private** companion repo, configured
**per device** in the tool's **⚙ Settings**: owner, repo, branch, `data.json` /
`history.json` paths, your name, and a **fine-grained Personal Access Token**
(Contents: read & write, scoped to that one private repo). The token is stored
only in the browser's localStorage.

- Reads use the GitHub Contents API `raw` media type; writes use the Git Data
  (blobs) API. Neither hits GitHub's 1MB Contents-API size limit.
- On a new machine: open the live site, open ⚙ Settings, fill in the private
  repo + a token, Save. (Find the exact private repo under your GitHub account;
  the token is per-device and must be re-entered.)

### Data shape (`data.json`, in the private repo)

```jsonc
{
  "personFields": [ { "key": "...", "label": "...", "kind": "manual|output", "group": "..." } ],
  "personGroups": ["..."],        // ordered; also careerGroups / promptGroups / recipeGroups
  "careerFields": [ ... ],
  "people":  [ { "id", "name", "careerId", "values": { "<key>": "..." } } ],
  "careers": [ { "id", "title", "values": { "<key>": "..." } } ],
  "prompts": [ { "id", "title", "group", "body" } ],
  "recipes": [ { "id", "title", "group", "promptId", "subject": "person|career",
                 "personKeys": ["..."], "careerKeys": ["..."],
                 "outputTarget": "person|career", "outputKey": "..." } ]
}
```

`history.json` is an array of `{ id, createdAt, by, recipeId, recipeTitle,
personId, personName, careerId, careerName, prompt, output }`.
