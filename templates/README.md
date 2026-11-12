# Templates

Every product repo starts from one of these. The factory selects a template; it never invents one (preplan, Layer 0). A scaffold generated per task would have the agent writing the sensors that judge its own work.

## Which template

| The product is | Template | Status |
|---|---|---|
| A web app, with or without an API and a database | `web-nextjs` | Ready |
| A backend service or CLI with no UI | none yet (Go) | Add when the first product needs it |
| Data or ML work | none yet (Python) | Add when the first product needs it |

Triage picks the row. A human confirms it when approving the spec. If no row fits, stop and ask; do not improvise a scaffold.

## What every template contains

- `factory/project.yaml` with the Layer 0 decisions, paths and gate commands filled in.
- `scripts/ci`: every gate, in pipeline order.
- `scripts/gates-selftest`: plants a known-bad change for each gate and requires the gate to reject it.
- `AGENTS.md` with the commands, and `CODEOWNERS` covering every sensor config file.
- Files that need the product name or owner end in `.jinja` and use `{{ project_name }}` and `{{ owner }}`. Everything else is copied as it is.
- A workflow in this repo's `.github/workflows/` that runs the template's `scripts/ci` on every change to it, and `scripts/render-check`, which creates a product with Copier and runs its fast gates.

## Adding or changing a template

A `factory`-class PR. An agent may draft it; a human code owner approves it. Start from the framework's official generator, commit its output untouched, then add the factory layer on top so the difference is reviewable.

## Creating a product

```bash
pipx install copier==9.19.1
copier copy --trust gh:rokorobot/SoftFactory2 <new-folder>
```

Copier asks for the template, the product name and the code owner's GitHub handle. It copies the template, fills in the name and owner, adds the policy files from `factory/`, and adds the Reviewer workflow (`.github/workflows/reviewer.yml`). The Reviewer only runs once the product repo has a `CLAUDE_CODE_OAUTH_TOKEN` or `ANTHROPIC_API_KEY` secret. No LLM takes part. `--trust` is needed because Copier runs the copy commands listed in `copier.yml`.

## Keeping a product in sync

```bash
copier update --trust      # inside the product repo, with a clean git status
```

Copier reads `.copier-answers.yml` to see which factory release the product was built from, then applies what changed since, keeping the product's own edits. Conflicts show up as normal merge markers. A release is a git tag on this repo (`v0.1.0`, `v0.2.0`): one tag covers the policy and every template.

Copier recommends one template per repository, because tags are shared. This repo keeps several under `templates/` by choice, using Copier's documented templated-subdirectory option; the cost is that a release for one template is a no-op update for products built on another.

## Not built yet

- The scheduled agent that runs `copier update` in each product repo and opens the PR.
