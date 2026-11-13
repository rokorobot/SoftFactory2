# Software Factory: State of the Art and Ideal-Case Stack Preplan

Compiled 2026-11-11. Amended 2026-11-11: templates (Layer 0), `project.yaml` (Layer 1), and the list of adopted open standards (Part 2), after the owner decided the factory builds software in general. Sources: André Staltz, Birgitta Böckeler (Thoughtworks / martinfowler.com), Simon Willison, Mitchell Hashimoto, Steve Yegge, Dex Horthy, Addy Osmani, Andrej Karpathy, StrongDM's factory team, Anthropic's Claude Code guidance, and the Sawyer Middeleer study of seven operating factories (Spotify, Stripe, Anthropic, Ramp, WorkOS, Uber, Monzo). Links at the end.

---

## Part 1. The consensus (what the experienced people agree on)

Where sources disagree, I say so. Where they agree, treat it as settled.

### 1.1 A factory is a program, written in English, running on agents
- **Staltz:** engineering craft is poured into instructions and infrastructure instead of code. Agent misbehavior is a bug you own. You debug runs, reasoning, and tool calls instead of stack traces.
- **Karpathy / Osmani:** "mostly programming in English now." Errors moved from syntax to concept: wrong assumptions, over-abstraction, dead code, sycophancy.
- **Horthy (12-Factor Agents):** production agents are mostly deterministic software with targeted LLM steps. Own your prompts, own your context window, small focused agents, stateless reducers, trigger from anywhere.
- **Implication:** version, review, test, and roll back your instructions like code. No prompt lives outside git.

### 1.2 Harness = Guides + Sensors (Böckeler's model is the cleanest)
- **Guides (feedforward):** instruction files, conventions, templates, codemods. Make the first attempt likely right.
- **Sensors (feedback):** linters, type checks, tests, structural tests, review agents. Catch what guides missed.
- **Computational sensors** (tests, linters, types) are cheap and deterministic. **Inferential sensors** (LLM review, LLM judge) are expensive and noisy. Run computational on every change, inferential only where judgment is needed.
- **Steering loop:** every recurring agent mistake becomes a new guide or sensor. This is the actual job of the factory operator. Hashimoto calls the same thing "harness engineering": build the tool that stops the repeat mistake.
- **Keep quality left:** fast checks before commit, expensive checks (mutation testing, architecture review) in the pipeline.
- **Harnessability matters:** typed languages, explicit module boundaries, opinionated frameworks shrink the agent's output space (Ashby's law). Pick your stack for how constrainable it is.
- **Harness templates (Böckeler):** most organizations need only a few service topologies. A harness template is "a bundle of guides and sensors that leash a coding agent to the structure, conventions and tech stack of a topology." She presents this as a likely future development, not established practice, and warns that templates are hard to version and keep in sync.

### 1.3 Fail closed, narrow scope, pristine environment
- **Staltz #12:** agents must not hack around constraints. A missing tool or broken env is a bug; the agent should stop, not improvise.
- **Thoughtworks Radar 34:** decide explicitly where nondeterminism is acceptable and where deterministic guardrails are mandatory.
- **fdeinterviews pipeline:** gates are code, not prompts. CODEOWNERS blocks agents from editing tests, CI, and factory config. Egress and dependency allowlists. Token, time, and retry caps in the orchestrator.

### 1.4 Verification is the bottleneck, not generation
- **Spotify at QCon London 2026:** generation is no longer the constraint, PR review is.
- **Osmani:** high-adoption teams merge ~2x more PRs, but review time rises up to 91% and PR size 154%. "Comprehension debt" when nobody can explain merged code.
- **Willison:** have agents write tests first; it raises the odds of working code dramatically. Have agents demo what they built (his Showboat/Rodney tools).
- **StrongDM:** tests can be reward-hacked, so they validate with "scenarios" held outside the agent's reach, and run against a "Digital Twin Universe" of cloned third-party services (Okta, Jira, Slack, Google Docs).
- **Pipeline consensus:** a separate verifier agent with fresh context, holdout scenarios never shown to the coder, fail-to-pass test per requirement, mutation score on changed lines.

### 1.5 Review like you review, then make it systematic
- **Staltz #8-#10:** introspect your own review checklist, teach one agent to do exactly that, invest in its report format. Bot review beats human review because it is objective, systematic, auditable. Review in English (the PR report), not in diff.
- **Osmani:** fresh-context self-review catches real mistakes.
- **fdeinterviews:** LLM judges are advisory only, calibrated against human-labeled examples, recalibrated on every model change. Separation of duties: the agent that wrote it cannot approve it, and neither can the requester.

### 1.6 Scale autonomy slowly, per change class
- **Staltz #13:** premature automation is the root of all evil. Get one agent correct in a correct environment, then widen.
- **Böckeler (QCon):** developer work is now risk assessment: probability × impact × detectability. Strong forces tempt you out of the loop; resist by design.
- **Pipeline consensus:** autonomy is assigned per change class (docs, deps, refactor, feature, auth/payments). A class earns more autonomy after a clean track record and loses it after an incident. Error budgets pause agent merges.
- **Hashimoto:** still rejects code you don't understand. Runs one background agent, not a swarm, for most work. Keeps research tasks running while away.
- **Yegge (Gas Town):** the counter-position: 20-30 parallel agents, state as JSON "beads" in git, a "Refinery" that cleans up all output, "Dogs" for maintenance. His own caveat: counterproductive for most teams; simplicity beats power. Takeaway worth keeping: persist all agent state as files in git so crashes are survivable.

### 1.7 Scheduled agents are the underused half
- **Staltz #2, #11:** scheduled agents make previously infeasible hygiene routine: dead-code deletion, dependency updates, tech-debt pattern elimination.
- **Böckeler:** continuous drift sensors outside the change lifecycle: dead code, coverage quality, dependency scanning. OpenAI's "garbage collection sweeps", Thoughtworks' "janitor army".
- **Middeleer's factories:** Ramp's on-call assistant joins incident channels and posts findings in minutes. Monzo triggers investigations from failed quality checks. WorkOS lets agents that hit gaps trigger a review of their own session logs to propose harness fixes.
- **Anthropic:** Claude Code routines (research preview since 2026-04) run on schedule, API call, or GitHub event in managed cloud. Local scheduled tasks exist for the desktop.

### 1.8 Measure the factory, not the agent
- Cost per **accepted** change, including failed attempts. Not cost per run.
- Rework rate, pass^k (consistency, not just average), review latency, incident rate per change class.
- Pin model versions to dates. Qualify upgrades with a replay suite of closed issues that had human fixes.
- Provenance on every commit: run ID, model, prompt/skill version, spec ID, change class.
- Weekly random deep audit of accepted changes by a human.

---

## Part 2. Ideal-case stack preplan

Opinionated. Assumes a small team (1-5 humans), GitHub, and willingness to spend on cloud agents. The factory builds software in general: the factory lives in one repo, and each product gets its own repo, created from a template. Tools named are the ones with the strongest track record as of now; swap equivalents freely, keep the layer.

### Built on existing open standards
Do not reinvent what a major open-source project already maintains. Checked on GitHub 2026-11-11.

| Piece of the factory | Standard | Status |
|---|---|---|
| Agent instruction file | `AGENTS.md` (agentsmd/agents.md) | In use |
| Specs with requirement IDs | GitHub Spec Kit's spec template (github/spec-kit): `FR-001` requirements, Given/When/Then scenarios, "needs clarification" markers | Approved |
| Running the Reviewer on PRs | anthropics/claude-code-action | Approved |
| Creating a product from a template, and keeping it in sync | Copier (copier-org/copier): `copier copy`, then `copier update` | Approved |
| Scheduled agents and triggers | GitHub Agentic Workflows (github/gh-aw): Markdown compiled to a GitHub Action; the agent is read-only and sandboxed, and writes go through validated "safe outputs" | Approved, from rollout stage 3 |
| Skills | Agent Skills spec (agentskills/agentskills): a folder with a `SKILL.md` | Proposed |
| Dependency updates | Dependabot or Renovate, instead of an agent | Proposed |
| Pinned environments (Layer 2) | Dev Containers spec | Proposed, later |
| Agent traces (Layer 5) | OpenTelemetry GenAI semantic conventions | Proposed, later |

No open standard was found for these, so they stay the factory's own: `change-classes.yaml`, `REVIEW_CHECKLIST.md`, `PR_REPORT_TEMPLATE.md`, `project.yaml`, and the gate self-test in each template.

Not adopted: whole systems with their own way of working (Gas Town, BMAD, OpenHands). Taking one would replace this plan, not fill a gap in it.

### Layer 0. Codebase made harnessable (do this first, before any automation)
| Decision | Choice | Why |
|---|---|---|
| Language | Strictly typed (TypeScript strict, Go, Rust, or Python with pyright strict) | Types are the cheapest sensor |
| Structure | Explicit module boundaries enforced by a structural lint (dependency-cruiser, ArchUnit, import-linter) | Shrinks output space |
| Framework | One opinionated framework per layer, no bespoke plumbing | Fewer degrees of freedom for the agent |
| Formatting | Single formatter + linter with autofix, errors written as fix instructions for an LLM | Sensor output must be actionable |
| Tests | Fast unit suite under 2 min, filtered runner script (`scripts/test-changed`) | Agents loop on it |
| Scripts | `scripts/` for every repeated action: screenshot, run-filtered-tests, seed-db, smoke | Hashimoto's harness engineering |

**Templates: Layer 0, decided once per kind of product**
- Every product repo starts from a prepared template in `templates/<name>/` in the factory repo, protected like `factory/`. A template is the table above already decided and wired: stack, lint and boundary rules, test setup, scripts, CI gates, and a filled-in `factory/project.yaml`.
- The factory selects a template. It never invents one. A generated scaffold would have the agent writing the sensors that judge its own work.
- Triage picks the template from a short decision table (web app, service or CLI, data work). A human confirms it when approving the spec.
- Copier creates the product repo from the template. No LLM in that step.
- New templates arrive as `factory`-class PRs. An agent may draft one when a product fits none; a human approves it.
- Build one template first. Add the next only when a real product needs it; a template no product has shipped on is untested.
- A scheduled agent runs `copier update` in each product repo and opens the PR. This answers Böckeler's versioning warning.

**Operating modes: three ways to point the factory at work**
The factory repo holds the agents, skills, gates, and templates. Each product lives in its own repo with a small `factory/` folder (`project.yaml`, checklist overrides).
- **New product:** Copier creates a repo from a template (above). The product passes Layer 0 on day one and starts at rollout stage 1.
- **Existing repo:** the factory audits it against the Layer 0 table and opens one PR per gap: `factory/project.yaml`, gates, `scripts/test-changed`, boundary lint. The product's stack is kept, not replaced by a template. The repo starts at rollout stage 0 and moves on only when the stage 0 exit criteria hold.
- **Self-improvement:** the factory is its own target. The harness improver (Layer 3) is controlled by a switch with three positions:
  - `off` (default): it does nothing.
  - `propose`: it reads failed runs and reverted PRs and opens PRs. A human merges every one.
  - `auto`: low-risk changes to skills and prompt wording merge after passing the gates. Everything else stays at `propose`.
  - Turning it on fails closed: the factory refuses to enable it until the rollout stage 3 exit criteria hold, because the improver has nothing to learn from without run history.
  - The switch is human-owned in `factory/` (CODEOWNERS), so the factory cannot raise its own setting.
  - It drops back one position on its own after a self-made change is reverted or breaks the error budget.
  - Gates, run caps, CODEOWNERS, change-class policy, and the switch itself are always human-merged, in every position. Without this rule, self-improvement becomes self-loosening.

### Layer 1. Instructions (the "source code" of the factory)
- `AGENTS.md` / `CLAUDE.md` at repo root: short, only what the agent cannot infer from the code. Build/test commands, hard rules, what never to touch. Over-specified files degrade performance (Anthropic).
- `skills/` directory: one skill per repeatable procedure (release, migration, perf check, PR report format). Skills are versioned and reviewed in PRs like code.
- `REVIEW_CHECKLIST.md`: your own introspected review criteria (Staltz #8). The review agent reads only this.
- `PR_REPORT_TEMPLATE.md`: mandatory report format: intent, spec IDs covered, tests added (fail-to-pass evidence), risk class, what was not done, open questions. Humans review this, not the diff, by default.
- `change-classes.yaml`: class → autonomy level → required gates → who approves. Stack-agnostic.
- `project.yaml`: the per-repo profile. Paths, gate commands, thresholds, and the Layer 0 decisions for that product. Each template ships one filled in.
- All of the above live in `factory/` and are protected by CODEOWNERS. Agents cannot edit them.

### Layer 2. Environment (pristine, fail-closed)
- Isolated sandbox per run: Cloudroom cloud threads, or Firecracker/E2B/Daytona. Fresh clone, pinned toolchain via devcontainer or Nix.
- No production secrets in sandboxes. Credentials injected by the orchestrator or a gateway, never visible to the agent.
- Egress allowlist (package registries, GitHub, the APIs you've cloned).
- Dependency allowlist; new dependencies require human approval.
- Digital twins or recorded fixtures for every external service the code touches. Start with the one integration that causes the most incidents.
- Hard caps per run: tokens, wall clock, retries. Exceeding a cap fails the run, it does not degrade gracefully.

### Layer 3. Agents (small, narrow, single-purpose)
| Agent | Trigger | Scope | Can write? |
|---|---|---|---|
| Triage | New issue / alert | Classify: ready, needs spec, needs info, defer. Assigns change class. Picks the template for a new product. | No (labels only) |
| Spec | Triage says "needs spec" | Requirements with IDs, each mapped to a named test. Human approves. | Spec file only |
| Implementer | Approved spec | One branch, one sandbox, runs local checks, opens PR with report. | Code + tests, not `factory/` |
| Verifier | PR opened | Fresh context. Checks built artifact against spec and holdout scenarios. Writes verdict. | No |
| Reviewer | PR opened | Runs `REVIEW_CHECKLIST.md` systematically. Posts structured report. Advisory. | No |
| Janitors (scheduled) | Cron | Dead code, dep updates, lint-rule migration, flaky test quarantine, doc drift, template sync. One concern per janitor. | PRs only, low-risk class |
| On-call | Alert | Reads logs/metrics, posts interim findings, may open a PR. Never deploys. | PR only |
| Harness improver (scheduled, weekly) | Cron | Reads failed runs and reverted PRs, proposes new guides/sensors as PRs to `factory/`. | PR to `factory/`, human merges |

Orchestration: GitHub Actions for triggers and gates. The Reviewer runs through claude-code-action. Scheduled agents run as GitHub Agentic Workflows; Claude Code routines or Cloudroom Loops remain the fallback. Add Temporal only when you need durable multi-step workflows with retries across days. Do not build a custom orchestrator early.

### Layer 4. Gates (code, not prompts)
Order of execution, cheapest first:
1. Format, lint, type check (pre-commit in sandbox).
2. Unit + structural tests, filtered to changed scope.
3. Fail-to-pass check: every spec requirement ID has a test that failed on base and passes on head.
4. Mutation score on changed lines above threshold.
5. Full suite + integration against twins (CI).
6. Verifier verdict + Reviewer report.
7. Merge queue: rebase onto head, retest.
8. Human approval per change class policy.
9. Progressive rollout behind a default-off flag, canary with automated baseline comparison, auto-rollback.

### Layer 5. Observability and trust
- OpenTelemetry traces for every agent run: prompt version, tool calls, tokens, cost. Store transcripts.
- Commit trailers: `Factory-Run`, `Factory-Model`, `Factory-Spec`, `Factory-Class`.
- Dashboard: cost per accepted change, rework rate, pass^k, review latency, incidents per class, janitor PR acceptance rate.
- Weekly human audit: random sample of accepted PRs read in full.
- Model pinning with a replay suite before any upgrade.

### Layer 6. Humans
- Humans own: product intent, architecture, spec approval, change-class policy, `factory/` merges, the weekly audit.
- Humans review the PR report, and drop to the diff only when the report or verdict raises a flag.
- Every escalation from an agent is a signal to improve a guide or sensor, not to answer the same question twice.

---

## Part 3. Rollout order (Staltz #13 applied)

Stages are gated by evidence, not by calendar. A stage opens only when every exit criterion of the stage before it holds, however long that takes. Each product repo moves through the stages on its own. A class or repo moves back a stage after an incident.

| Stage | Build | Exit criteria (all must hold to move on) |
|---|---|---|
| **0. Harnessable codebase** | Layer 0: the first template, and the first product repo created from it (or an existing repo adopted) | Format, lint, type check, and filtered tests run green in a fresh sandbox with one command, in under 2 minutes |
| **1. Instructions + Reviewer** | Layer 1. Run log from the first run: one JSON line per run (run ID, pinned model, skill version, class, tokens, cost, outcome). Reviewer on human PRs only. No autonomy. | Reviewer findings match the human's own review on N consecutive PRs (e.g. 10), with no important misses |
| **2. Implementer, lowest-risk class** | Implementer on docs, tests, lint fixes, with full human review | N PRs accepted with only minor edits; every recorded mistake has become a guide or sensor |
| **3. Verifier + first janitors** | Verifier with holdout scenarios; the first two janitors (deps, dead code) | The Verifier caught at least one real defect the Implementer's tests missed; janitor PR acceptance is above threshold |
| **4. Promote a class + observability** | One more change class, twins for the top incident-causing integration, provenance, and the dashboard (the first UI) | The promoted class meets its error budget over N changes |
| **5. Ongoing** | Promote or demote classes on evidence. Weekly harness-improver run, if its switch is on. Never skip the weekly human audit. | n/a |

N and the thresholds are set in `factory/change-classes.yaml`, not here.

**When to build the dashboard:** at stage 4, and only when both hold:
1. The run log has enough accepted and rejected changes for the metrics to be stable.
2. Someone keeps answering the same question by hand from the run log. That repeated query defines the first screen.

Stop condition at every stage: if a run fails because the environment is wrong, fix the environment, do not patch the prompt.

---

## Part 4. Anti-patterns to refuse
- "Send a dozen LLMs to review and see what sticks" (Staltz #8).
- Reviewing in diff while authoring in English (Staltz #10).
- Letting the coder agent edit tests, CI, or `factory/`.
- Letting an agent generate a new product's scaffold, including its own lint, test, and CI config.
- Building templates for stacks no product needs yet.
- `latest` model aliases in production.
- Gracefully degrading when a tool is missing.
- Measuring cost per run instead of cost per accepted change.
- Multi-agent swarms before one agent is reliable.
- Treating silent sensors as proof of quality rather than proof of weak detection.

---

## Sources
- Staltz, X thread on Software Factory lessons (user-provided, 2026-11).
- Böckeler, Harness engineering for coding agent users, martinfowler.com 2026-04-02: https://martinfowler.com/articles/harness-engineering.html
- Böckeler, QCon London 2026 coverage: https://theregister.com/2026/03/18/ai_for_software_developers_qcon and https://infoq.com/news/2026/03/ai-coding-state
- Willison, Agentic Engineering Patterns: https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/ and StrongDM visit: https://simonwillison.net/2026/Feb/7/software-factory
- Hashimoto, My AI Adoption Journey: https://mitchellh.com/writing/my-ai-adoption-journey
- Yegge, Gas Town series: https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04 and Maggie Appleton's analysis: https://maggieappleton.com/gastown
- Horthy, 12-Factor Agents: https://github.com/humanlayer/12-factor-agents
- Osmani, The 80% Problem in Agentic Coding: https://addyosmani.com/blog/the-80-problem-in-agentic-coding/
- StrongDM factory: https://factory.strongdm.ai/ and Stanford CodeX critique: https://law.stanford.edu/2026/02/08/built-by-agents-tested-by-agents-trusted-by-whom/
- Middeleer, You should take software factories seriously: https://exponentialoperator.substack.com/p/you-should-take-software-factories
- Agentic Software Factory pipeline: https://fdeinterviews.com/blog/agentic-software-factory
- Anthropic, Claude Code best practices: https://code.claude.com/docs/en/best-practices and routines: https://code.claude.com/docs/en/routines
- Adopted standards: https://github.com/agentsmd/agents.md , https://github.com/github/spec-kit , https://github.com/anthropics/claude-code-action , https://github.com/copier-org/copier , https://github.com/github/gh-aw , https://github.com/agentskills/agentskills
- Thoughtworks Technology Radar vol. 34 (2026-04): https://www.thoughtworks.com/content/dam/thoughtworks/documents/radar/2026/04/tr_technology_radar_vol_34_en.pdf
