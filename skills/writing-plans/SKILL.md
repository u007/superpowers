---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write implementation plans for an engineer who has not seen this codebase or this spec. Assume they write idiomatic code in the project's language once they know the exact interface and the exact test, and that they will make a reasonable choice wherever the plan leaves one open. What they cannot know is what you decided: which files, which names and signatures, which values from the spec, which tests prove each task. Document those. Give them the whole plan as bite-sized tasks, written as decisions rather than as code — see **Detail Level** below. DRY. YAGNI. TDD sized to risk.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `superpowers:using-git-worktrees` skill at execution time.

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Detail Level

**Default: decisions, not code.** A plan names the file, the function or
type, and the technical aspect — the contract, the data that flows through
it, the values the spec pins, the failure modes it must handle — and stops
there. That is the bar, and it is a real bar: an engineer who has never
seen this codebase writes the actual code from your plan without
re-deriving the design, and without guessing a name, a signature, or a
required value.

The implementer writes the implementation. Do not write it for them.

Naming enough to write code is not an invitation to include the code.
"Specific" means every decision is pinned, not that every line is written.

**Technical plan — only when the user asks for one.** When the request is
explicitly for a technical plan ("technical plan", "code level", "with
code", "include the code"), add the implementation detail the default
withholds: exact signatures with parameter and return types, the body of
any algorithm the signature and tests do not determine, and the exact copy,
literals, and schema the spec fixes. Technical is additive — keep every
decision the default requires, then add the code.

Never infer technical from the subject matter. A request for a plan, a
spec, a feature, or a refactor is a default-level plan. Do not ask which
level is wanted just because a plan feels thin: thin is the default, not a
defect. Record the level in the plan header so the reader and the executor
know which one they are holding.

**Test steps carry code in both modes.** A test's name and assertions are
the contract, so they go in the plan as a short snippet. It is the
shortest block in the document and the one that earns its place.
Implementation bodies are what the default withholds.

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.

Aim for the fewest tasks that keep each one reviewable — usually 3-6 for a
feature. Every task boundary costs an implementer start-up and a review;
a plan of fifteen two-minute tasks spends most of its time on handoffs.

**Mark every task's risk.** Each task carries `**Risk:** standard` or
`**Risk:** high` (superpowers:using-superpowers, "Right-Size the
Process"). High: bug fixes, branching logic with edge cases, concurrency,
persistence or migrations, auth/security, money, public API contracts,
refactors of untested code. Everything else is standard. The marking
drives how the task is built (red-first vs tests alongside) and whether
it gets its own review gate. When unsure, mark it high.

## Step Granularity

**High-risk tasks — each step is one action with a checkable result:**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step

**Standard tasks — two steps:** implement the change together with its
tests (named, with their assertions), and run the task's tests (command and
expected result). The implementer still writes a behavioral test
for each new behavior; it just is not a separate red step.

A plan does not dictate commits. Leave the implementer to commit where
they judge the work is coherent; a plan step that exists only to say
"commit" carries no decision.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans (default) or superpowers:subagent-driven-development (long plans of independent tasks) to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Spec:** [path to the spec/design doc this plan implements — the plan
argues from the spec, so the spec travels with it; executors read both]

**Detail:** decisions, not code — the default; the implementer writes the
implementation. Write "technical plan, requested by <who>" instead only
when the user explicitly asked for one.

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

## Review Focus

[The five input classes or failure modes the spec implies but no task's
tests exercise that are most likely to bite a person using this software
— one line each, naming the input or condition and the behavior a
reasonable person would expect, most likely first. The spec is a vision
document: it says what the software must do, not everything it will
meet, and its silence on an input is not permission for that input to
break the program. Write the list here, once, with the spec in front of
you. Then, for each line, add the test that pins it to the task that
owns the code, in that task's own step style.]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Risk:** standard | high — [one clause: why]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Implement `function(input: InputType) -> ResultType` in `exact/path/to/file.py`**

One line on the approach when the signature and the test leave a choice
(which library call, which data structure). **The signature, the file, and
the values the spec pins are the whole step — the implementer writes the
body.** A body appears only for an algorithm the signature and tests do
not determine, for exact copy the spec fixes, or in a technical plan.

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS
````

## What a Step Contains

A step is done when the implementer can write exactly one reasonable thing
from it. That is the whole requirement: unambiguous, not complete. Each kind
of step carries what makes it unambiguous and nothing more:

- **A test step:** the test's name and its assertions, as code, with the
  spec's exact values in them. This is the one step that carries a code
  block in a default plan.
- **A code step:** the exact signature (name, parameters, return type), the
  file it lives in, and the specific values the spec pins. The implementer
  writes the body. In a default plan a body appears only for an algorithm
  the signature and tests do not determine, or for exact copy the spec
  fixes; in a technical plan, add it throughout.
- **A verification step:** the command to run and the output that means it
  passed.
- **A reference to another task:** that task's Interfaces block says what
  to use; the plan does not repeat that task's code.

Sufficiency, not completeness, is the test. A step names the file, the
function or type, and enough technical detail — the contract, the data
flow, the values, the edge cases — that the implementer writes real code
without a second design pass. Stop there. Writing the code as well is not
thoroughness; it is the implementer's work, done for them.

A plan is the set of decisions the implementer cannot make alone. A plan
longer than the code it describes has written the code instead. Lines that
decide nothing ("TBD", "handle edge cases", "add appropriate validation",
"write tests for the above", a type or function no task defines) are the
opposite failure, and the self-review catches both.

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Step scan:** Every step must let the implementer write exactly one reasonable thing, and no step may carry more than that: a line that decides nothing is a gap, a function body the signature and tests already determine is a transcript. Fix both.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**4. Review Focus:** For each input class or failure mode the spec implies, is there a task whose tests exercise it? The five uncovered ones most likely to bite a person go in the Review Focus section, and each line there gets its test added to the owning task. An empty section means you checked and found none, not that you skipped the check.

**5. Proportion:** Compare the plan's length to the spec's. A plan several times longer than the spec it implements is a transcript of the program, not a plan. If code blocks are most of the document, replace bodies with signatures, test names and assertions, and check that each step is still unambiguous.

**6. Detail level:** Does the plan match the level its header claims? For a default plan, every implementation body the signature and tests already determine comes out, leaving test snippets and exact copy. For a technical plan, check the opposite — that the bodies and signatures the header promised are actually there. A default plan with implementation bodies in it is the failure this check exists for; do not leave a thin plan alone on the grounds that thin is the default, because a step that names no file or no function is a gap under either level.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving and self-reviewing the plan, link it for your human partner
to read. If they have already explicitly supplied an execution method, ask
them to review the plan and confirm it captures what they want; wait for that
review before implementation, then use the preserved method. Otherwise, ask
them to review the plan and choose an execution method before implementation.

**When no execution method has already been supplied:**

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Please review the plan. Which execution approach would you prefer?**

- **Subagent-driven** - A fresh subagent implements each task and a fresh reviewer checks it before the next one starts, then a whole-branch review at the end. Most thorough; costs a fresh context per task and per review.
- **Native** - I implement every task myself in this session, the way this harness runs work, then one fresh reviewer on the most capable model checks the whole branch. Cheapest and fastest; no independent review until the end. Runs well with a mid-tier session model, since the plan carries the design.

Default to **Native** unless the plan has more than about six tasks, or
most tasks are marked high-risk and independent enough to review one at a
time — per-task review gates pay for themselves only there.

**For this plan I recommend <one of the two>, because <one sentence from the plan: how much the tasks depend on each other's interfaces, how many there are, what a shipped mistake would cost>. Does the plan capture what you want, and which approach should we use?"**

**When an execution method has already been supplied:**

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Please review the plan. Does it capture what you want?"**

**If Subagent-driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development

**If Native chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
