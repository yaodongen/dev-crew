# Brief templates for subagents

Every brief states the goal, the files to read, the files to produce, and what to report back. Replace the angle-bracket placeholders with real values:

- `<process-dir>`: absolute path of `.dev-crew/<task-slug>/` under the main repository
- `<workspace>`: absolute path of the directory where this task's code is changed (see `status.md`)
- `<language>`: the user's language

The templates are in English. Write the actual brief in whatever language works best for the subagent, but keep the language instruction below so the files come out in the user's language.

End every brief with this block:

```
Workspace: <workspace>
All code for this task lives in the directory above. Read code and run commands there,
and do not touch code in any other directory.
Process files are in <process-dir>. They are not part of the code change.

Constraints:
- Do only what this brief asks. If you find something outside it that needs attention,
  put it in your report instead of handling it.
- Do not git commit, push, or deploy.
- Report faithfully: say plainly what is unfinished, unverified, or uncertain.
- Write the files you produce and your report in <language>, including headings.
```

---

## Step 1: Clarify requirements (A)

```
You are a programmer who has just received a new task. For now, only clarify the
requirements. Do not design a solution and do not change any code.

The original request is in the "Original request" section of <process-dir>/requirements.md.
Read it, read the related parts of the codebase, then fill in these three sections of
the same file (do not edit "Original request"):

1. Understanding: a few sentences in your own words.
2. Acceptance criteria: a list that can be verified item by item. Each item must be
   checkable as "do this → observe that".
3. Q&A log: anything in the request that is unclear, self-contradictory, in conflict
   with the existing code, or missing necessary information. One table row per question,
   filling in only the "Question" column: why it is a problem, the relevant code location
   (file:line), the possible answers, and how each answer would affect the implementation.
   Only raise questions that affect the design or the implementation. Add no rows if
   there are none.

Report: how many questions you raised, or "no open questions".
```

Later rounds:

```
The "Q&A log" in <process-dir>/requirements.md now has answers to your earlier questions.
Read it again and update "Understanding" and "Acceptance criteria" accordingly. If you
have new questions, append rows to the Q&A log.
Report: how many questions you added, or "no open questions".
```

## Step 2: Design (A)

```
You are a programmer responsible for designing the implementation of a task. For now,
only write the design. Do not change any product code.

The requirement and acceptance criteria are in <process-dir>/requirements.md. Read it
and the related code, then write the design to <process-dir>/design.md with these sections:

1. Goals and non-goals
2. Current state: how the related code works today
3. Design: what changes, with the key data structures, interfaces, and flows
4. Scope of changes: the files to be added or modified, one sentence each on what changes
5. Alternatives and trade-offs: other approaches considered and why they were not chosen
6. Risks and compatibility: what could go wrong, impact on existing behavior, whether it
   can be rolled back
7. Test plan: how each acceptance criterion will be verified (concrete commands or steps)
8. Deviations: leave empty; it is filled in during implementation

<When the task spans several repositories, add: This task involves these repositories:
<name: path, …>. Group "Scope of changes" by repository, and add a section
"Cross-repository contract" after "Design": the interfaces, data formats, and ordering
the repositories must agree on, precise enough that each repository can be implemented
by a different programmer in parallel.>

Write all file paths relative to the repository root.
Prefer patterns and utilities that already exist in the codebase. The design should be
the simplest approach that satisfies the requirement.
Report: a summary of the key points, and anything you are unsure about.
```

Revising the design:

```
<process-dir>/design.md is the current design. The "Rulings" section of
<process-dir>/design-review-<N>.md lists the review comments that were accepted.
Revise design.md for the accepted comments; leave the rejected ones alone.
Do not change product code.
Report: how each accepted comment was handled.
```

## Step 3: Design review (B)

```
You are a senior programmer reviewing a design. Apart from the review file named below,
do not create or modify any file.

The requirement is in <process-dir>/requirements.md and the design is in
<process-dir>/design.md. Read the requirement first, then the design, and check the
design's claims about the current state against the codebase yourself.

Focus on:
- whether the design fully satisfies every acceptance criterion
- whether its description of the existing code is accurate
- missed edge cases, concurrency, error handling, compatibility
- whether there is a simpler approach, or something reusable already in the codebase
- whether the scope of changes is missing anything
- whether the test plan can actually verify the acceptance criteria

Write the review to <process-dir>/design-review-<N>.md in this format:

# Design review, round <N>
## Comments
### 1. [blocking] <title>
- Problem:
- Evidence: <code location or requirement item>
- Suggested change:
### 2. [suggestion] <title>
…
## Verdict
pass / fail

Number comments consecutively from 1. Levels:
- blocking: leaving it unchanged would fail the requirement, introduce a bug, create a
  security or data risk, or make the design impossible to implement
- suggestion: better if changed, but deliverable as is
Any blocking comment means fail. If there are no problems, simply write pass; do not
invent comments to have something to say.

<From round 2 on, add: The following were discussed and deliberately not adopted; do not
raise them again: …>
```

## Step 5: Implement (A)

```
You are a programmer implementing a task from an approved design.

The requirement is in <process-dir>/requirements.md and the design is in
<process-dir>/design.md. The design has been reviewed and approved by the user.

Requirements:
- Follow the design strictly and change only the files listed in its "Scope of changes".
- If you find the design does not work, or that files outside the scope must change,
  stop immediately and report why. Do not decide on your own.
- Do not refactor, reformat, upgrade dependencies, or fix unrelated problems along the
  way. If you notice unrelated problems, put them in your report.

<When the workspace is a newly created worktree, add: This is a fresh git worktree with
no dependencies or local config files. First install dependencies and set up local
config as the project docs describe. That does not count as an out-of-scope change, but
do not add those files to git.>

<When the task spans several repositories, add: You are implementing only the
<repo name> part: the files listed for this repository in "Scope of changes". Other
programmers are implementing the other repositories at the same time, relying on the
"Cross-repository contract" in the design. Implement your side exactly as the contract
says. If the contract needs to change, stop and report; do not change it on your own.>

Report: the files actually changed (what changed in each), anything that differs from
the design, and anything left unfinished.
```

Fixing code (review comments, test failures, demo problems, reverting out-of-scope changes):

```
You are a programmer modifying an already implemented task.

The requirement is in <process-dir>/requirements.md and the design is in
<process-dir>/design.md.
Items to handle: <the accepted comments in the "Rulings" section of code-review-<N>.md /
the failing items in test-report.md / a concrete problem description / the files and
content to revert>

Requirements:
- Handle only the items listed above; make no other changes.
- Afterwards rerun the affected tests and include the commands and real output in your
  report.

Report: how each item was handled, the files changed, and the test results.
```

## Step 6: Code review (B)

```
You are a senior programmer reviewing a code change. Apart from the review file named
below, do not create or modify any file.

The requirement is in <process-dir>/requirements.md and the design is in
<process-dir>/design.md (its "Deviations" section records deviations that were agreed).
The change is everything in the workspace that differs from commit <baseline commit>.
Inspect it in the workspace with `git diff <baseline commit>` and `git status` (watch for
untracked new files).
<When the workspace is the main repository directory, add: The .dev-crew/ directory holds
process files and is not part of the code change; ignore it.>
<When reviewing a single fix, replace with: Review only the following change: …>
<When the task spans several repositories, add: You are reviewing only the <repo name>
part of the task. Also check that this repository's side matches the "Cross-repository
contract" in the design. Write the review to <process-dir>/code-review-<repo>-<N>.md.>

Focus on:
- whether the implementation matches the design and satisfies every acceptance criterion
- correctness: logic errors, edge cases, error handling, concurrency, resource leaks
- changes outside the design's "Scope of changes" and "Deviations"; list each one
- leftover debug code, commented-out code, weakened or skipped tests
- consistency with the codebase's existing style and patterns

Write the review to <process-dir>/code-review-<N>.md in this format:

# Code review, round <N>
## Comments
### 1. [blocking] <title>
- Problem:
- Evidence: <code location, or requirement / design item>
- Suggested change:
### 2. [suggestion] <title>
…
## Verdict
pass / fail

Number comments consecutively from 1. Levels:
- blocking: leaving it unchanged would fail the requirement, introduce a bug, create a
  security or data risk, or contradict the design
- suggestion: better if changed, but deliverable as is
Any blocking comment means fail. If there are no problems, simply write pass; do not
invent comments to have something to say.

<From round 2 on, add: The following were discussed and deliberately not adopted; do not
raise them again: …>
```

## Step 7: Test (A)

```
You are a programmer testing an implemented task.

The requirement and acceptance criteria are in <process-dir>/requirements.md and the
test plan is in <process-dir>/design.md.

Carry out the test plan and verify each acceptance criterion. Write the results to
<process-dir>/test-report.md:

# Test report
## Acceptance criteria
### 1. <the criterion, verbatim>
- Command or steps:
- Real output: <paste the relevant part as is>
- Result: pass / fail / could not verify (say why)
…
## Existing project checks
<the command and result for tests, lint, type checks, and build; for each failure, note
whether it already failed at the baseline commit>

Requirements:
- Output must come from actually running things now. Do not write "should pass" from
  inference.
- For items you could not verify, say why. Do not count them as passing.
- If something fails, do not fix it. Put the symptom and your analysis in the report.
- Do not modify, skip, or delete tests, and do not change product code.

Report: the result for each acceptance criterion, and the list of failing and
unverifiable items.
```
