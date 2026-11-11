# Review Checklist

The Reviewer agent reads this file and nothing else for review criteria. It runs every item, in order, on every PR, and reports per item using `PR_REPORT_TEMPLATE.md` section 7.

**Status: DRAFT.** This is a starting default. Staltz #8: it only works once it encodes what *you* actually check. To make it yours:
1. Review your next 10 PRs by hand. Write down every question you asked yourself and every comment you left.
2. Delete items below you never check. Add yours. Keep IDs stable; retire IDs instead of reusing them.
3. Every time a bug reaches main that review should have caught, add or sharpen an item here (the steering loop).

## Rules for the Reviewer agent
- Fresh context. Do not read the implementer's transcript. Read: the spec, the PR report, the diff, and the code it touches.
- Every item gets exactly one verdict: `PASS`, `FAIL`, `N/A`, or `UNSURE`.
- Every `FAIL` and `UNSURE` must cite evidence: `file:line` and one sentence. No evidence, no finding.
- `N/A` needs a one-line reason.
- Do not invent items. Observations outside this list go under "Other observations", capped at 3, and are never blocking.
- Do not suggest style changes the formatter or linter would catch. Those are sensors, not review.
- You are advisory. You never approve or merge.

## Severity
- **BLOCK**: a `FAIL` here stops merge regardless of change class.
- **WARN**: a `FAIL` here needs a human to accept it in writing on the PR.
- **NOTE**: reported, never gating.

---

## A. Intent and scope
| ID | Severity | Check | How to verify |
|---|---|---|---|
| A1 | BLOCK | The PR does what the linked spec or issue asks, and every requirement ID is addressed. | Map each spec ID to a hunk in the diff and a test. List any unmapped ID. |
| A2 | BLOCK | The PR does nothing the spec did not ask for. | List every changed file that no spec ID explains. Drive-by refactors count. |
| A3 | WARN | The PR is one concern. | If the report's summary needs "and" to join two unrelated changes, FAIL. |
| A4 | WARN | Diff size fits the change class limit in `change-classes.yaml`. | Compare added+removed lines, excluding generated files and lockfiles. |

## B. Correctness
| ID | Severity | Check | How to verify |
|---|---|---|---|
| B1 | BLOCK | Error paths are handled, not swallowed. | Find every new `catch`/`except`/`err`/`Result`. Each one rethrows, returns an error, or logs with context and has a reason. Empty handlers FAIL. |
| B2 | BLOCK | Edge cases: empty, null, zero, max, duplicate, concurrent. | For each new function taking external input, name which of these apply and whether a test covers it. |
| B3 | WARN | No new behavior depends on wall clock, locale, randomness, or ordering of unordered collections without control. | Search the diff for time, random, and map/set iteration feeding output. |
| B4 | BLOCK | Data changes are reversible or explicitly approved as one-way. | Any migration, delete, or overwrite has a down path or a note in report section 5. |

## C. Tests (the sensor on the sensor)
| ID | Severity | Check | How to verify |
|---|---|---|---|
| C1 | BLOCK | Every spec requirement ID has a test that fails on base and passes on head. | Check the fail-to-pass evidence in report section 4. Missing evidence FAILs. |
| C2 | BLOCK | No existing test was weakened, skipped, deleted, or had its assertion loosened. | Diff all test files. Any removed assertion, added skip, widened tolerance, or changed expected value needs a spec ID that justifies it. |
| C3 | WARN | Tests assert behavior, not implementation. | FAIL if a test only checks that a mock was called, or would pass with the function body deleted. |
| C4 | NOTE | Mutation score on changed lines meets threshold. | Read from CI. Report the number. |

## D. Design and maintainability
| ID | Severity | Check | How to verify |
|---|---|---|---|
| D1 | WARN | No new abstraction with a single caller. | For each new interface, base class, factory, or wrapper, count call sites. One caller FAILs unless the spec requires extension. |
| D2 | WARN | Reuses existing helpers instead of duplicating them. | For each new function over 10 lines, search the repo for one with a similar name or body. |
| D3 | BLOCK | Module boundaries respected. | Structural lint must pass. Also FAIL on new imports that cross a boundary the lint does not yet cover; propose a lint rule in "Other observations". |
| D4 | WARN | No dead code left behind. | Unused exports, unreachable branches, commented-out code, TODOs without an issue link. |
| D5 | NOTE | Names say what things are. | Only flag names that are misleading, not ones you would merely have chosen differently. |

## E. Security and safety
| ID | Severity | Check | How to verify |
|---|---|---|---|
| E1 | BLOCK | No secrets, tokens, or credentials in code, tests, fixtures, or logs. | Scan the diff for high-entropy strings, key prefixes, and new log lines that print request or env objects. |
| E2 | BLOCK | All external input is validated at the boundary. | For each new endpoint, handler, CLI arg, or file read, point to the validation. |
| E3 | BLOCK | No new dependency without human approval. | Any lockfile or manifest change adding a package FAILs unless the PR has the `dep-approved` label. |
| E4 | BLOCK | Authz checked on every new data access path. | For each new query or resource read, name the permission check. |
| E5 | WARN | No new outbound network call to a host outside the egress allowlist. | Search for URLs and client constructors. |

## F. Operability
| ID | Severity | Check | How to verify |
|---|---|---|---|
| F1 | WARN | New user-facing behavior is behind a flag, default off. | Feature class only. Name the flag. |
| F2 | WARN | Failures are observable. | New failure paths emit a log or metric someone could alert on. |
| F3 | NOTE | Performance: no new N+1, unbounded loop over external data, or sync call in a hot path. | Name the suspect line or PASS. |

## G. Factory integrity
| ID | Severity | Check | How to verify |
|---|---|---|---|
| G1 | BLOCK | No changes to `factory/`, CI config, CODEOWNERS, or test infrastructure unless the change class is `factory`. | Path check. |
| G2 | BLOCK | The agent did not work around the environment. | FAIL on new retries, sleeps, `--force`, `--no-verify`, disabled lint rules, type `any`/`ignore` comments, or vendored copies of tools, unless the spec asks for them. Staltz #12: a broken environment is a bug to report, not route around. |
| G3 | WARN | The PR report is complete and true. | Every section of the template filled. Claims in the report match the diff. A report that overstates FAILs. |

---

## Changelog
Record every add, change, or retirement with date and the incident or PR that caused it.
- 2026-11-11: initial draft from the preplan. Not yet calibrated against human reviews.
