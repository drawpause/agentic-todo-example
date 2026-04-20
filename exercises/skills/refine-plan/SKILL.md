---
name: refine-plan
description: Perform a rigorous self-critique of the current plan file, then iteratively improve it until it meets a quality bar or further progress is blocked by missing information.
---

# refine-plan

Treat the current plan as a draft that is probably wrong. Score it honestly, research the weak parts, rewrite them, and re-score until it's good enough to execute.

## When to Use

Invoke this skill when:

- A plan file exists in the conversation (from plan mode, `/plan`, or written explicitly) and needs hardening before execution.
- The user says "refine the plan", "critique this plan", "tighten up the plan", or similar.
- You're about to execute a complex plan and want a self-check pass first.

Do **not** invoke for:

- Initial plan creation (use plan mode or architecture work instead).
- Plans already at score 8+ with no known gaps.
- One-line or trivial plans where critique is overhead.

## Step 1: Locate the plan

Read the current plan file. Check, in order:

1. The plan file path in system context (if provided).
2. The most recently written `plan.md` / `PLAN.md` in the working directory.
3. Ask the user to point to the plan file if neither is found.

If no plan exists, tell the user and stop — do not fabricate one.

## Step 2: Self-assessment (score 0–10)

Evaluate the plan against these criteria and assign a score from 0 to 10. Be adversarial: assume the plan is flawed and try to prove it.

- **Completeness**: Are all identified issues/tasks addressed? Are there gaps?
- **Specificity**: Are changes described at the file + line level with concrete code, or hand-waved with vague phrases like "update tests" or "fix the config"?
- **Correctness**: Are the proposed changes actually correct? Do imports exist? Do APIs match? Are parameter names right?
- **Ripple effects**: Are all downstream impacts traced? If a function signature changes, are ALL callers updated? If a schema changes, are all consumers covered?
- **Verifiability**: Can someone execute the plan and confirm success with the verification steps provided?
- **Loose ends**: Are there any "verify later" or "TBD" items that should be resolved now?
- **Risk & blast radius**: Are destructive or hard-to-reverse steps called out? Is rollback considered?
- **Simplicity**: Can anything be cut? Is the plan over-engineered for the task?

Present the score with a breakdown of strengths and weaknesses, organized by the criteria above. Flag each weakness with severity: **blocker**, **major**, or **minor**.

## Step 3: Research weak areas

For each weakness:

1. Use read-only tools (Read, Grep, Glob, Explore agents) to gather missing information.
2. Verify assumptions against actual code — don't trust the plan's claims.
3. Resolve any "TBD" or "verify later" items now, not later.

If information isn't available (e.g., requires user decision, external system access, or runtime data), note it explicitly rather than guessing.

## Step 4: Rewrite weak sections

Edit the plan file in place, replacing only the weak sections with improved content. Do **not** rewrite sections that scored well — that's churn.

When rewriting:

- Prefer concrete file + line references and code snippets over prose.
- Cut weak content rather than pad it — a shorter accurate plan beats a longer vague one.
- Preserve the plan's original structure and headings where possible.

## Step 5: Re-score

Re-evaluate the updated plan using the same criteria. Present the new score with a brief comparison to the original.

**Stop conditions** — exit the loop when any of these are true:

- Score reaches 8+ with no remaining blockers or majors.
- Further improvement requires information that is not available (explain what's missing and who/what could unblock it).
- Three iterations have passed without the score improving — further changes would be churn.

Otherwise, repeat steps 3–5.

## Step 6: Report

Summarize:

- Starting score → final score.
- What was changed (by section, briefly).
- Any tradeoffs accepted or items deferred, and why.
- Anything the user should confirm before executing the plan.

Keep the report tight — the plan file itself is the deliverable; the report is just a pointer to what moved.
