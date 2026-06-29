# Prompt Builder

A single-file, static web tool for running a prompt pipeline: store reusable AI
prompts, fill them with each person's / career's data, copy the assembled prompt
out to run in an LLM, paste the result back, and chain one step's output into the
next. No AI is built in — it assembles prompts and stores results.

**Live:** https://sophophobia.github.io/PromptBuilder/

## Cross-device setup

```
git clone https://github.com/Sophophobia/PromptBuilder.git
```

The local folder name does not matter — git tracks the remote, not the folder.

- **Edit** `index.html` (the whole app; inline CSS + vanilla JS, no build).
- **Commit & push** to `main`. GitHub Actions redeploys Pages automatically.
- On another machine: `git pull` to get the latest, or `git push` after edits.

## How it works (quick tour)

- **People / Careers** hold a shared set of **fields** (grouped, orderable). A
  field is `manual` (you type) or `output` (filled by an AI result). Each person
  links to one career.
- **Prompts** are reusable instruction text.
- **Recipes** pair a prompt with the fields to append under it and the output
  field to save into. Output fields feed downstream recipes, forming a pipeline.
- **Generate**: pick a recipe + a person (or career), copy the assembled prompt,
  run it externally, paste the result back, save. **History** keeps every run.

## Data

Real data is not stored in this repo. It lives in a **private companion repo**,
configured per device in the tool's **⚙ Settings** (owner, repo, branch, paths,
your name, and a fine-grained Personal Access Token with Contents read/write).
The token stays in your browser only. Reads use the raw media type and writes use
the Git Data (blobs) API, so there is no 1MB file-size limit.

## Deployment notes

- Pages is built by **GitHub Actions** (`.github/workflows/deploy.yml`), not
  Jekyll. `.nojekyll` is present because the page uses literal `{{ }}` tokens.
- `index_legacy.html` and the other `index_*.html` files are older prototypes.

See `CLAUDE.md` for the full working context and data shape.
