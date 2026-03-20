---
name: improve-loop
description: >
  Run an automated iterative improvement loop for any project. Reads project docs to understand
  context, proposes an evaluation strategy, then runs a generate-evaluate-fix loop with optional
  expert panel review. Use this skill whenever the user says "improve loop", "improvement loop",
  "run the loop", "iterate on quality", "make it better", or wants to iteratively improve the
  quality of some output their project produces. Even if the user just says "the output isn't
  good enough" or "let's make this more realistic/accurate/performant", this skill applies.
  Do not use this skill if a more specific project-level /improve command exists.
---

# Iterative Improvement Loop

You are running an automated improvement loop that generates output from a project, evaluates
it, optionally has domain experts assess quality, then fixes the underlying code or configuration
to address findings. Each iteration: generate, evaluate, fix, test, commit.

The power of this approach is that it creates a tight feedback loop between output quality and
code changes, with each iteration building on the last. The expert panel (when used) catches
issues that automated evaluation misses because domain experts notice things that scoring
functions can't anticipate.

## Phase 1: Discovery

Before asking the user anything, learn about the project on your own.

### 1. Read project documentation

Look for and read these files (skip any that don't exist):
- `PRD.md` or `docs/PRD.md` — product requirements
- `AGENTS.md` — AI developer guide
- `TODO.md` — current work state and plans
- `CLAUDE.md` — project conventions and instructions
- `README.md` — project overview
- `package.json`, `pyproject.toml`, `Cargo.toml`, etc. — project tooling

Your goal is to understand:
- What the project does and what it produces
- How to build/run/test it
- What CLI commands or scripts exist
- What "quality" means in this context

### 2. Check git status

If this is a git repo, run `git status` and `git log --oneline -5` to understand the current
state. You'll use this later when proposing a branch.

## Phase 2: Propose a Plan

Based on what you learned, present a plan to the user. Be specific — show actual commands,
actual file paths, actual persona descriptions. The user should be able to say "looks good"
or make targeted corrections.

Your proposal should cover:

### What to improve

Explain what output you'll be evaluating and why. Be specific: "I'll run `make generate` and
evaluate the CSV files it produces in `output/`" not "I'll evaluate the project output."

### How to generate output

The command(s) to produce the output that will be evaluated. This could be:
- A CLI command that writes files (e.g., `uv run forge generate scenario.yaml -o output/`)
- A script that produces output (e.g., `python scripts/render.py`)
- A command whose stdout/stderr needs to be captured to a file
- A multi-step build process

### How to evaluate

Propose one or more evaluation approaches. Be clear about what each one provides:

**Option A: Automated evaluation** — if the project has an eval command or scoring system,
use it. Show the exact command and explain what metrics it produces.

**Option B: Expert panel** — spawn 3-4 domain-expert agents who review the output blind
(they don't know they're part of an improvement loop). Each expert focuses on a different
aspect of quality. Describe each proposed expert: their role, what they focus on, and why
that perspective matters.

**Option C: Both** — run automated eval for scores, then expert panel for qualitative
findings that scoring can't capture.

**Option D: Custom** — something else entirely (a script, a checklist, manual review).

If you're not sure which approach fits, present the options and let the user choose.

### Expert panel design (if applicable)

Propose a team of 2-5 expert agents, where each expert covers a different key dimension of
quality for this project's output. The goal is complementary coverage — each expert should be
an authority on one of the critical aspects involved in producing or consuming this output.

To choose experts, identify the key quality dimensions for the output (e.g., for a REST API:
correctness, performance, security, developer experience; for generated data: field fidelity,
cross-source consistency, temporal realism, domain authenticity). Then assign one expert per
dimension — someone whose real-world role makes them the natural authority on that dimension.

For each proposed expert, describe:
- **Role**: Their title and experience level (e.g., "Senior API consumer with 8 years integrating payment APIs")
- **Focus**: Which specific quality dimension they own and what they look for
- **Why**: What this perspective catches that the other experts would miss

The panel should feel like assembling a review board — each person brings a distinct lens,
and together they cover the full surface area of quality.

### How to run tests

The command to verify code changes don't break anything. If you found a test suite, show the
command. If there isn't one, say so — the loop can still work, it just skips the test step.

### Git workflow

If it's a git repo, propose a branch name (e.g., `improve/<what-you're-improving>`).
If the user doesn't want git integration, skip this.

### Budget and iterations

Suggest defaults:
- **Budget**: $3.00 (ask the user to set this)
- **Max iterations**: 5

## Phase 3: Get Confirmation

Wait for the user to confirm or adjust the plan. Do not proceed until they approve.

Common adjustments:
- Different expert personas
- Different eval commands
- Adding or removing evaluation approaches
- Changing budget or iteration count
- Focusing on specific quality dimensions

## Phase 4: Pre-Loop Setup

### 1. Verify the baseline

Run the test suite (if one exists) to confirm everything passes before you start. If tests
fail, STOP and tell the user — don't begin the loop with a broken baseline.

### 2. Create the working branch (if using git)

```bash
git checkout -b <branch-name>
```

If the branch already exists from a previous run, check it out and continue where it left off.

### 3. Establish baseline scores

Run an initial generate + evaluate cycle. Record the results and print a summary:

```
=== Improvement Loop: Starting Baseline ===
Target: <what you're improving>
Overall Score: XX/100 (or whatever the eval produces)
Budget: $X.XX
Max Iterations: N

<dimension/metric breakdown if available>

Priority targets:
  - <lowest scoring area>
  - <next lowest>
```

If there's no automated eval, skip the scores — the first iteration's expert panel will
establish the baseline.

---

## Phase 5: The Loop

For each iteration (1 through max_iterations):

### Step 1: Generate Output

Run the agreed-upon generation command, outputting to a per-iteration directory:

```
output/improve-iter-<N>/
```

### Step 2: Evaluate

Run whichever evaluation approach(es) were agreed upon:

#### Automated evaluation (if configured)

Run the eval command, parse results, note score changes from baseline and previous iteration.

#### Expert panel (if configured)

Spawn expert agents **in parallel** using the Agent tool. Each agent gets:

- The output directory path so they can read files directly
- Brief context about the project and what the output represents
- Their specific review focus area
- Instructions to use Glob/Read/Grep tools (not cat/grep/find)

**Important framing**: Tell each expert what the output is supposed to be and ask them to
assess its quality. They should identify specific issues, rate severity, and explain what
they'd expect to see instead. Use this severity scale:

- **P0 — Critical**: Fundamentally broken, would be immediately rejected
- **P1 — Significant**: Expert would flag this in review, needs fixing
- **P2 — Minor**: Polish issue, noticeable but not blocking
- **P3 — Nitpick**: Marginal improvement, fix if easy

### Step 3: Consolidate Findings

After evaluation completes:

1. Collect all findings from all sources (automated eval + expert panel).
2. Deduplicate — if multiple experts flag the same issue, merge and note consensus.
3. Sort by severity (P0 > P1 > P2 > P3), then by how many sources flagged it.
4. Cross-reference with any TODO.md items — note if a finding matches known work.

Print the consolidated findings:

```
=== Iteration N: Findings ===
Score: XX/100 (delta: +/-X from baseline)

<Expert verdicts if panel was used>

Consolidated findings (N total, M unique after dedup):

P0 — Critical:
  1. [sources] <description>

P1 — Significant:
  2. [sources] <description>
  ...

P2 — Minor:
  ...
```

### Step 4: Plan and Apply Fixes

Select the top 3-5 findings to fix this iteration, prioritizing:
1. P0 items always first
2. P1 items that align with the lowest scores
3. Items where multiple sources agree
4. Items that are feasible to fix in code (not fundamental architecture changes)

For each fix:
- Read the relevant code before changing it
- Make targeted, incremental changes — not large refactors
- Preserve existing patterns and conventions from the project's CLAUDE.md/AGENTS.md
- If the fix requires configuration/input changes alongside code changes, make both

### Step 5: Run Tests

Run the test suite. If tests fail:
1. Read the failure carefully.
2. If the test asserts old behavior that the code change intentionally altered, update the test.
3. If the fix introduced a regression, fix the code.
4. Re-run until all pass.
5. If stuck after 3 attempts on the same issue, revert and move on.

If there's no test suite, skip this step.

### Step 6: Commit (if using git)

```bash
git add <specific-files-changed>
git commit -m "$(cat <<'EOF'
Iteration N: <brief summary of fixes>

Fixes applied:
- <finding 1 summary>
- <finding 2 summary>

Score: XX/100 (was XX/100)

Co-Authored-By: Claude <model> <noreply@anthropic.com>
EOF
)"
git push -u origin <branch-name>
```

### Step 7: Check Stop Conditions

**Stop** (finish current iteration, then stop) if:
- Cost exceeds budget
- Max iterations reached
- User requests stop

**Important**: Good scores alone are NOT a stop condition. Automated evals rarely capture
everything — expert findings often reveal improvements the eval can't measure. Keep iterating
as long as there is budget and findings to address.

### Step 8: Iteration Report

Print a summary, then proceed directly to the next iteration:

```
=== Iteration N Complete ===
Score: XX/100 (was XX/100, baseline XX/100)
Delta this iteration: +/-X
Cumulative delta: +/-X

Fixes applied:
  1. <fix> — <files changed>
  2. ...

Remaining findings:
  P0: N remaining
  P1: N remaining
  P2: N remaining (not targeted)

Tests: XXX passing
Budget: ~$X.XX spent of $X.XX

[Continuing to iteration N+1 / Stopping: <reason>]
```

---

## Phase 6: Final Report

When the loop ends, print a comprehensive summary:

```
=== Improvement Loop: Final Report ===

Target: <what was improved>
Branch: <branch-name>
Iterations completed: N of M max

Score Progression:
  Baseline:     XX/100
  Iteration 1:  XX/100 (+/-X)
  ...
  Final:        XX/100

<Dimension breakdown if available, showing start -> end>

Total fixes applied: N
  - Code changes: N files modified
  - Config/input changes: N files updated
  - Test updates: N tests modified

Findings resolved: N of M total
  P0: N/M resolved
  P1: N/M resolved

Remaining findings (for manual follow-up):
  1. <unaddressed finding>
  2. ...

Commits: N commits on branch <branch-name>
To merge: git checkout main && git merge <branch-name>
```

## Constraints

1. **Incremental changes** — prefer small, targeted fixes over large refactors. Each iteration
   should fix 3-5 specific issues, not rewrite modules.
2. **Preserve test coverage** — every code change must pass existing tests. Add new tests for
   new behavior when practical.
3. **Respect project conventions** — follow the patterns in CLAUDE.md, AGENTS.md, and the
   existing codebase. Don't introduce new frameworks or paradigms.
4. **Cross-cutting consistency** — changes that affect one component often need corresponding
   changes elsewhere. Think about downstream effects before committing.
5. **Backward compatibility** — code changes should not break existing configurations or inputs
   unless explicitly agreed with the user.
