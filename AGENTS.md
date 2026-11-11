# AGENTS.md

This repo designs and builds a software factory: agents that build software under human-owned rules. Design rationale: `SOFTWARE_FACTORY_PREPLAN.md`.

## Hard rules
- Never edit `factory/`, `.github/`, `AGENTS.md`, `CLAUDE.md`, or `skills/` unless your task's change class is `factory`. Those changes need a human code owner.
- Never edit the `specs`, `infra`, `sensor_config`, or `flag_config` paths listed in `factory/project.yaml` unless your change class is `specs` or `infra-ci`.
- Every PR description follows `factory/PR_REPORT_TEMPLATE.md`. Every section is filled in; claim only what you ran.
- Classify your change with `factory/change-classes.yaml`. If several classes match, use the riskiest one.
- Fail closed. If a tool, dependency, credential, or environment step is missing or broken, stop and report it in report section 6. Do not work around it: no `--no-verify`, `--force`, added sleeps or retries, disabled lint rules, or ignore comments.
- Never delete, skip, or loosen an existing test.
- No new dependency without a human-applied `dep-approved` label.
- Never commit secrets or `.env` files.
- One concern per PR. Do nothing the spec or issue did not ask for.

## Commands
None yet: no application code exists. Add build, test, and lint commands here when they do.
