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
- A workflow in this repo's `.github/workflows/` that runs the template's `scripts/ci` on every change to it.

## Adding or changing a template

A `factory`-class PR. An agent may draft it; a human code owner approves it. Start from the framework's official generator, commit its output untouched, then add the factory layer on top so the difference is reviewable.

## Not built yet

- The script that creates a product repo from a template: it copies the template and the policy files in `factory/`, and fills in `REPLACE_WITH_PROJECT_NAME` and `REPLACE_WITH_OWNER`.
- The scheduled agent that keeps product repos in sync with their template.
