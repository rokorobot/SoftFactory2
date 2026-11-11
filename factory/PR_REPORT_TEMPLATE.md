# PR Report Template

Every agent-authored PR uses this as its description. The Implementer fills sections 1-6. The Verifier appends section 8 and the Reviewer appends section 7. Humans review this report first, and open the diff only when something here raises a flag (Staltz #10).

Rules for whoever fills it in:
- Write in plain English, short sentences. Write so someone who has not seen the code can decide whether to merge.
- Never leave a section out. Write "None." if it is empty.
- Every claim needs evidence: a command and its result, a test name, or `file:line`.
- Never claim something you did not run. "Not run" is a valid answer; a guess is not.

Copy everything below the line.

---

## 1. Intent
<!-- One or two sentences: what changes for the user or system, and why. No implementation detail. -->

**Spec / issue:** <!-- link -->
**Change class:** <!-- from factory/change-classes.yaml, e.g. docs, feature -->
**Autonomy level:** <!-- L1, L2, L3, as the class currently allows -->

## 2. Requirements coverage
| Spec ID | Done? | Where (file:line) | Test |
|---|---|---|---|
| R1 | yes / partial / no | | `test_name` |

## 3. What changed
<!-- Bullets, one per behavior change, in English. Not a file list. -->
-

**Files touched:** <!-- count, and any outside the expected area with a one-line reason -->
**Diff size:** <!-- +added / -removed, excluding generated and lockfiles -->

## 4. Evidence it works
**Fail-to-pass:** each requirement's test failed on base and passes on head.
| Test | Base | Head |
|---|---|---|
| `test_name` | FAIL | PASS |

**Checks run:**
| Check | Command | Result |
|---|---|---|
| Format + lint | | |
| Type check | | |
| Tests (changed scope) | | |
| Full suite | | not run / pass / fail |

**Demo:** <!-- screenshot, CLI output, or curl transcript showing the behavior. "None" for non-observable changes, with a reason. -->

## 5. Risk
- **What could break:** <!-- the most likely failure in production, specifically -->
- **Blast radius:** <!-- who or what is affected if it does -->
- **Reversibility:** <!-- revert is enough / needs data fix / one-way -->
- **Flag:** <!-- name and default, or "none" with reason -->
- **New dependencies:** <!-- name@version and why, or "None" -->

## 6. Not done, assumptions, questions
- **Deliberately not done:** <!-- things the spec could be read to include that you left out -->
- **Assumptions:** <!-- every guess you made where the spec was silent -->
- **Environment problems hit:** <!-- missing tools, broken setup, flaky tests. You must report these, not work around them. -->
- **Questions for a human:**

---

## 7. Review (appended by Reviewer agent)
**Checklist version:** <!-- git SHA of factory/REVIEW_CHECKLIST.md -->

**Summary:** <!-- BLOCK count, WARN count. One sentence: merge-ready or not, and why. -->

| ID | Verdict | Evidence |
|---|---|---|
| A1 | PASS / FAIL / N/A / UNSURE | `file:line`, one sentence |

**Other observations (max 3, non-blocking):**
-

## 8. Verification (appended by Verifier agent)
**Verdict:** PASS / FAIL
**Holdout scenarios:** <!-- n run, n passed. Names of any failures. Scenario contents stay hidden from the Implementer. -->
**Built artifact checked against spec:** <!-- what was run, and what was seen -->

---

<!-- Commit trailers the Implementer adds to every commit:
Factory-Run: <run id>
Factory-Model: <pinned model id, never an alias>
Factory-Skill: <skill name>@<git sha>
Factory-Spec: <spec id>
Factory-Class: <change class>
-->
