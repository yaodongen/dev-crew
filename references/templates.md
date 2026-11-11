# File templates

`design.md` and `test-report.md` are written by the programmers; their formats are in steps 2 and 7 of `briefs.md`.

The templates are in English. Write the actual files in the user's language, including headings.

## status.md

```markdown
# <task-slug>

- Current step: step <N> <step name> (write "done" when everything is finished)
- Workspace: <absolute path> (worktree / current repository directory; "undecided" until chosen)
- Branch: <branch> (new, based on <base branch> / existing current branch)
- Baseline commit: <HEAD inside the workspace when it was decided>

(When the task spans several repositories, replace the three lines above with a table:
| Repository | Workspace | Branch | Baseline commit |)

## Gate log
Append only; never edit existing rows. Status values: passed / skipped at user's request / went back.

| Time | Step | Status | Notes |
|---|---|---|---|
| 2026-01-01 10:00 | 1 Clarify requirements | passed | asked the user 2 questions |
| 2026-01-01 10:40 | 2 Design | passed | |
| 2026-01-01 11:30 | 3 Design review | passed | 2 rounds |
| 2026-01-01 14:00 | 4 User approval | passed | user said: "OK, go with this" |
| 2026-01-01 15:10 | 5 Implement | passed | changed files: a.ts, b.ts |
| 2026-01-01 16:00 | 7 Test | went back | criterion 2 failed; back to fix and code review |

## Waiting on the user
- <what you are currently waiting for the user to answer, or "nothing">

## Acceptance criteria deferred to the demo
- <criteria that cannot be verified locally and the user agreed to defer, or "none">
```

## requirements.md

When you create the file, write only "Original request". Programmer A fills in the other three sections; you fill in the "Answer" and "Source" columns.

```markdown
# Requirements

## Original request
<the user's words, verbatim>

## Understanding

## Acceptance criteria
1. <do this → observe that>

## Q&A log
| # | Question | Answer | Source |
|---|---|---|---|
| 1 | … | … | user's words / code path:line / docs |
```

## The "Rulings" section of review files

In `design-review-N.md` and `code-review-N.md`, programmer B writes "Comments" and "Verdict". You append this at the end:

```markdown
## Rulings (manager)
| # | Level | Accepted? | Reason |
|---|---|---|---|
| 1 | blocking | accepted | … |
| 2 | suggestion | rejected | … |
| 3 | blocking | rejected (user's decision) | … |
```

## Presenting the design (step 4, to the user)

This is written for someone who was not part of the process. Lead with the conclusion, use full sentences, and do not use labels that were coined along the way.

```markdown
## Design for <task name>

**Problem being solved**: <one or two sentences>

**Approach**: <the core idea in a paragraph; add a short step list if the flow is complex>

**Scope of changes**
| File | What changes |
|---|---|
| … | … |

**What will not change**: <parts the user might worry about that are left untouched>

**Trade-offs**: <what was chosen, what was given up, and why>

**Risks**: <what could go wrong and how it is handled>

**How it will be verified**: <how each acceptance criterion is tested>

**Review**: <N> review rounds. Adopted: <…>. Not adopted: <…, with reasons>.

**Decisions needed from you**: <open questions, or "none">

Please confirm whether you approve implementing this design.
```

## summary.md (step 8)

```markdown
# Change summary

Baseline: <commit>   Changes: <N> files (<M> new)

## Changes within the design
| File | What changed |
|---|---|

## Changes outside the design
| File | What changed | Why | Decision |
|---|---|---|---|
| … | … | … | waiting on user / keep / reverted |

(write "none" if there are none)

## In the design but not done
<write "none" if there are none>

## Test results
<the result for each acceptance criterion; which commands you reran and whether they matched the report; criteria deferred to the demo>
```

## .dev-crew/config.md

Project-level configuration shared by all tasks, kept under the main repository directory.

```markdown
# dev-crew project config

## Deploying to dev
- Command:
- Target environment / URL:
- Requires a commit or push first:
- How to reach or verify it after deploying:
```
