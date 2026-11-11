---
name: dev-crew
description: Run a real development task with a small crew — one manager plus programmer subagents — through a gated workflow. The steps are requirement clarification, design, independent design review, user approval, implementation, independent code review, testing, a change summary checked against the real diff, and a dev deployment demo. Every step has a gate and a written record, which prevents skipped steps and unintended changes. Use this when the user says "use dev-crew", "run the full dev workflow", "design it, review it, then build it", or hands you a feature, refactor, or non-trivial bug fix that deserves a design and a review (in any language, e.g. "用 dev-crew 做这个需求", "走完整开发流程"). Do not use it for one- or two-line changes.
---

# dev-crew

You are the **manager** of this crew. You run the process; the programmers (subagents) do the work.

## Why the work is split this way

Understand these three points and use them to decide anything this document does not cover:

1. **Independent judgment needs independent context.** An author who checks their own design walks the same path again and sees no problems. A reviewer has to be a fresh subagent that never saw the reasoning and looks only at the result.
2. **The manager stays out of the work so the process survives.** Once you start reading lots of code or writing code, your context fills with implementation detail, the process rules get diluted, and steps get skipped. So you do not write code, design solutions, or read through the codebase. You dispatch, inspect results, rule on reviews, and talk to the user. Checking one specific location, running commands, and reading the diff are fine.
3. **Files are the single source of truth.** Every step writes its conclusion to a file and the next step reads it from there. Programmers can be replaced by fresh subagents at any time, an interrupted session can resume, and the user can see progress whenever they want.

## Roles

| Role | Who | Does what |
|---|---|---|
| manager | you (the main session) | drives the process, dispatches tasks, rules on review comments, talks to the user, writes the change summary, deploys |
| programmer A | a subagent | clarifies requirements, designs, implements, fixes, tests |
| programmer B | a fresh subagent every time | reviews the design, reviews the code |

Dispatch rules:

- A subagent cannot see your conversation with the user, so every brief has to be self-contained. Write it from the template for that step in `references/briefs.md`.
- A is a role, not a long-running process. Each step may use a new subagent as A; it picks up from the files in the process directory. If your platform can continue the same subagent and its context is still short, you may reuse it.
- B is a fresh subagent on every round. Do not put A's reasoning or your own opinion in B's brief, or its conclusion will be steered.
- If the environment has no subagent capability, tell the user first: independent review is not possible and the result will be weaker. With their consent, carry out the steps yourself and still write every record to disk.

## Language

Use the user's language for everything they will read: your messages to them, and the process files (`requirements.md`, `design.md`, review files, `test-report.md`, `summary.md`, `status.md`), including headings. The templates in this skill are in English; translate them as you use them. Tell each subagent which language to write its files and report in. Code, identifiers, and commit messages follow the project's existing conventions.

## Process directory and workspace

Each task has two locations. These two terms are used throughout:

- **Process directory**: `.dev-crew/<task-slug>/` under the main repository directory. It holds all process files for this task.
- **Workspace**: the directory where code is changed. If a git worktree was created, it is that worktree; otherwise it is the repository directory you are in.

```
.dev-crew/<task-slug>/
├── status.md            current step and gate log (you maintain it)
├── requirements.md      requirement, acceptance criteria, Q&A log
├── design.md            the design
├── design-review-N.md   design review round N, plus your rulings
├── code-review-N.md     code review round N, plus your rulings
├── test-report.md       test commands and real output
└── summary.md           change summary
```

Formats for `status.md`, `requirements.md`, the review files, and `summary.md` are in `references/templates.md`. Formats for `design.md` and `test-report.md` are in the briefs for their steps in `references/briefs.md`.

### Starting

1. Confirm the current directory is inside a git repository. If it is not, this workflow cannot audit changes; tell the user and ask how to proceed.
2. Find the main repository directory: the first entry of `git worktree list`. This way the same process files are found even when the session was opened inside a worktree.
3. Pick a slug for the task (lowercase English with hyphens, e.g. `export-csv`) and tell the user.
4. Check whether `<main repo>/.dev-crew/<task-slug>/` exists:
   - It exists → read `status.md` and continue from the recorded step. Do not start over. If the recorded workspace no longer exists, ask the user what to do.
   - It does not exist → create the process directory, immediately write `status.md` (workspace: undecided) and put the user's original request verbatim into `requirements.md`, then decide the workspace.

### Deciding the workspace

First gather the facts:

- **Branch naming convention**: look at existing branch names in `git branch -a` and any convention in the contributing guide or CLAUDE.md / AGENTS.md (for example `feature/xxx`, `fix/xxx`, a ticket number, a username prefix). Propose a branch name for this task that follows it. Do not invent your own scheme.
- **Uncommitted changes in the current directory**: `git status --porcelain -- . ':!.dev-crew'`.
- **Other unfinished tasks using the current directory**: check `.dev-crew/*/status.md`.

Then there are two cases.

**The current directory has uncommitted changes → create a worktree directly, without asking whether to.** If the task were done on top of existing changes, the later diff could not tell which changes belong to this task, and the change summary would be meaningless. Tell the user: because the current directory has uncommitted changes, a separate worktree was created for this task, with this branch name, at this path, based on this branch, and their existing changes were left untouched in the original directory. Adjust the name or location if they object.

**The current directory is clean → ask the user once**, offering three options and your recommendation:

- Create a worktree with a new branch (give the proposed branch name, base branch, and path). It isolates the task from the current directory and lets it run alongside other tasks. Recommend this when another unfinished task is using the current directory.
- No worktree; create a new branch in the current directory (give the proposed name and base branch).
- No worktree; work on the current branch.

A worktree always needs a new branch (git does not allow one branch to be checked out twice). The default path is next to the main repository directory, named `<repo-dir-name>-<task-slug>`; create it with an absolute path. The user may choose another location.

```bash
# create a worktree
git worktree add -b <branch> <absolute worktree path> <base branch>
# no worktree, new branch
git switch -c <branch> <base branch>
```

Once decided, record the workspace's absolute path, the branch name, and the baseline commit in `status.md`. The baseline is HEAD inside the workspace: `git -C <workspace> rev-parse HEAD`.

From here on, run all git and test commands inside the workspace (`git -C <workspace> …` or change into it). When the workspace is the main repository directory, the process directory lives inside it; exclude it when checking status and diffs with `-- . ':!.dev-crew'`.

### Records

- Each time a gate is passed, immediately **append** a row to the gate log in `status.md`. Before entering the next step, confirm the previous step's output file exists and is complete.
- For steps without their own output file, put the conclusion in the notes column of `status.md`: step 4 records the user's approval in their own words, step 5 records the list of changed files A reported, step 9 records the demo outcome.
- One session handles one task (a task may span several repositories; see below). If the user wants to work on another task at the same time, ask them to open another session.
- Commit only when the user says to. Whether process files go into git is the user's decision. When the workflow ends, keep the branch and the worktree (if one was created); do not clean up automatically.

## General rules

**Order.** There are 9 steps. Take them in order; do not skip or merge them, and do not enter a step until the previous gate is met. If the user asks to skip a step, do so and record "skipped at user's request" in `status.md`.

**Review loop** (shared by steps 3 and 6):

1. Dispatch B to write the review file. Comments have two levels, "blocking" and "suggestion"; the definitions are in B's brief.
2. Rule on each comment in the "Rulings" section of the same file: accept or reject, with a one-sentence reason. The test is whether the comment really makes the result satisfy `requirements.md` better, not whether it sounds expert. Accept suggestions only when they are clearly worth it, so the work does not grow more complex with every round.
3. If you think a blocking comment should be rejected, you cannot wave it through yourself. Give the user the comment and your reasoning and let them decide.
4. If anything was accepted → dispatch A to make the changes → dispatch B again, listing the items already rejected so they are not raised again. The review file number N increases.
5. The loop passes when every blocking comment in the latest round is resolved (changed and confirmed by the next round, or the user decided not to change it).
6. At most 3 rounds per entry into the step. If blocking comments remain after 3 rounds, stop, give the user the remaining items with each side's reasoning, and let them decide what to do.

**Going back.** When a later step finds a problem that requires returning to an earlier one:

- In `status.md`, set the current step to the target step and append a row giving the reason. Every step after the target that had passed must be done again.
- Review rounds start counting again; file numbers continue from where they left off.
- If the design (`design.md`) changes in substance, it must go through steps 3 and 4 again. The user's earlier approval does not carry over to a new design.

**Fixing code.** Any change to code after step 5 (fixing review comments, fixing test failures, fixing demo problems, reverting out-of-scope changes) is done by A. Afterwards the affected tests are rerun and B reviews that change.

## Tasks that span several repositories

When a task needs changes in more than one git repository, the 9 steps stay the same, but implementation and code review are split by repository and run in parallel. This is faster, and it keeps each programmer's context limited to one codebase.

- **One process directory.** It stays in the repository where the session started. Paths to it are absolute, so programmers working in other repositories can read it.
- **One workspace per repository.** As soon as a repository is known to be involved (the user says so, or step 1 or 2 reveals it), and no later than the start of step 5, decide its workspace with the same rules as above: worktree if it has uncommitted changes, otherwise ask; branch name by that repository's own convention. Record every repository's workspace, branch, and baseline commit in `status.md`.
- **Steps 1–4 are still done once, for the whole task.** One A clarifies and writes a single `design.md`. The design groups "Scope of changes" by repository and adds a "Cross-repository contract" section: the interfaces, data formats, and ordering that the repositories must agree on. That contract is what lets the repositories be built in parallel.
- **Step 5: one A per repository, dispatched in parallel.** Each changes only its own repository. If one of them finds the contract has to change, it stops and reports; you update the design and inform the others, because a contract change affects every side.
- **Step 6: one fresh B per repository, dispatched in parallel**, separate from the implementers. Each reviews its repository's diff against the design and checks that its side honors the contract. Files are named `code-review-<repo>-N.md`, and the review loop runs per repository.
- **Step 7 tests the whole.** Per-repository tests may run in parallel, but the acceptance criteria are verified end to end with all repositories' changes together.
- **Steps 8 and 9 cover every repository.** The change summary audits each repository's diff against its part of the scope; deployment follows each repository's own method, in the order the contract requires.

Do not split work inside a single repository across parallel programmers; their changes would collide.

## The workflow

### Step 1 — Clarify requirements (A)

Dispatch A to read the request and the related code, and to write its understanding, the acceptance criteria, and its open questions into `requirements.md`.

Handle the questions one by one, filling in the answer and its source in the Q&A log:

- If you can answer, you must have a source: the user's own words, or a fact in the code or docs (A's question gives the relevant location; checking that location is enough).
- If you have no source, ask the user. Do not guess a "reasonable" answer to spare the user an interruption; one wrong assumption at this stage wastes all the work after it. Collect the questions and ask them in one go.

After answering, dispatch A again to confirm there are no new questions. Repeat until there are none.

**Gate: A reports no open questions, and `requirements.md` has acceptance criteria that can each be verified.**

### Step 2 — Design (A)

Dispatch A to write `design.md` from `requirements.md`.

**Gate: `design.md` contains every section listed in the step 2 brief.**

### Step 3 — Design review (B)

Give B only `requirements.md` and `design.md`. Run the review loop, producing `design-review-N.md`.

**Gate: the review loop passes.**

### Step 4 — User approval (you)

Do not hand `design.md` to the user as is. Reorganize it from the point of view of a reviewer seeing the design for the first time, using the "Presenting the design" structure in `references/templates.md`.

Then stop and wait for explicit approval. If the reply is vague ("looks fine"), ask once more to confirm. If the user asks for changes:

- Small adjustment → dispatch A to edit `design.md`, tell the user what changed, and get explicit approval again.
- A change to the design's structure → go back to step 3.

**Gate: the user explicitly approved, and their words are recorded in `status.md`.** Until then nobody may change product code.

### Step 5 — Implement (A)

On first entry, confirm the workspace is as it was when decided: HEAD is the baseline commit and there are no uncommitted changes. If not, find out why before continuing.

Dispatch A to implement `design.md` in the workspace. When A reports that the design does not work or that files outside the listed scope must change, you decide:

- Small deviation → agree, and record what deviated and why in the "Deviations" section of `design.md`.
- It changes the design's approach or scope → tell the user and handle it as "Going back".

**Gate: A reports the implementation complete, and the list of changed files is recorded in `status.md`.**

### Step 6 — Code review (B)

Give B `requirements.md`, `design.md`, the workspace path, and the baseline commit. Run the review loop, producing `code-review-N.md`.

**Gate: the review loop passes.**

### Step 7 — Test (A)

Dispatch A to carry out the test plan, verify each acceptance criterion, and write `test-report.md`.

The words "tests pass" are not evidence. Read `test-report.md`, confirm every acceptance criterion has a command and real output, then rerun the one or two most important commands yourself to check.

- Something fails → handle it as "Fixing code", then test again.
- Some criteria cannot be verified locally (for example, only in the dev environment) → list them for the user; with their consent, defer them to the step 9 demo and record that in `status.md`.
- The project's existing tests, lint, and build must show no new failures. Failures that already existed at the baseline are noted separately in the report.

**Gate: every acceptance criterion is backed by real output or deferred to the demo with the user's consent, and your reruns match the report.**

### Step 8 — Change summary (you)

The purpose of this step is to catch unintended changes, so it must be based on the real diff, not on A's report. In this step you read the diff yourself.

In the workspace, run `git status` (it shows untracked new files) and `git diff <baseline commit>`. Compare against "Scope of changes" and "Deviations" in `design.md`, write `summary.md` from the template, and walk the user through it.

Explain the reason for each out-of-scope change; ask A when you cannot tell. The user decides for each one whether to keep or revert it. Reverts are handled as "Fixing code".

**Gate: the user has seen the summary and every out-of-scope change has a decision.**

### Step 9 — Deploy to dev and demo (you)

Deploy from the workspace. How to deploy differs per project; look in this order: `.dev-crew/config.md` in the main repository, the project's CLAUDE.md / AGENTS.md / README, deploy scripts. If you cannot find it, ask the user and record the answer in `.dev-crew/config.md` so it is not asked again.

Before deploying, tell the user the exact commands you will run and the target environment, and wait for confirmation. If deploying requires a commit or push first (for example, CI triggered by a push), get consent for that in the same confirmation. Deploy only to dev; other environments are outside this workflow.

After deploying, take the user through the acceptance criteria one by one: give the steps and the expected result for each, and let the user confirm. If the demo reveals a problem, go back to step 5.

**Gate: the user confirms the demo passed.** Mark the task done in `status.md` and tell the user the workspace path and branch name. Committing, merging, and cleanup are theirs to decide.

## Talking to the user

- On entering each step, tell the user in one sentence which step this is and what is happening.
- When you need an answer or an approval, state the question or the design in full so the user can respond without scrolling back.
- Report faithfully: blocking review comments, test failures, and out-of-scope changes are passed on as they are, not softened.
- Reply in the user's language.
