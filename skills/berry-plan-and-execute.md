---
name: berry-plan-and-execute
description: Explore a repo with evidence, propose a verified plan with explicit file changes and tests, then execute only after user approval. Use for multi-step implementation tasks requiring evidence-backed planning with an approval gate before any edits.
---

You are in **Plan and Execute** mode.

## Phase 1 — Search & Learn (repo understanding)
- Use the Search & Learn verification pattern to explore and understand the repo.
- Build an Evidence pack of spans `S0`, `S1`, ... (repo excerpts, docs, configs).
- Every factual sentence must end with citations like `[S0]`.
- If you cannot cite, label it **Unknown** or **Assumption**.
- After drafting the repo understanding, call:
  `audit_trace_budget(steps=..., spans=..., require_citations=true, context_mode='cited')`
- If flagged: gather more evidence and re-run.
- If `audit_trace_budget` is run 3 times in a row, STOP and return only the claims that passed plus the claims that flagged and why they flagged.

## Phase 2 — Plan (Greenfield-style, but for changes)
- Produce **Facts (cited)**, **Decisions**, **Assumptions** based on the evidence.
- Then propose a plan with **explicit steps** that includes:
  - unit tests to add/update
  - integration tests to add/update
  - exact files to change (paths and what will change)

## Phase 3 — Dry-run plan only
- Do NOT run commands or edit files.
- Output only a dry-run plan that outlines the exact file changes.

## Phase 4 — Approval gate
- Ask the user to approve the plan before any execution.
- If not approved, return to Phase 2 and revise the plan.

## Phase 5 — Execute (only after approval)
- Implement the planned edits as real patches.
- Run the planned unit and integration tests.
- If tests fail or evidence contradicts the plan, return to Phase 2 and revise.
- Repeat until tests pass or the user stops the loop.

## Verification (plan steps)
- Create a trace where each step is a plan step `{idx, claim, cites}`.
- Call `audit_trace_budget(steps=..., spans=..., require_citations=true, context_mode='cited')` on the plan steps.
- If any step is flagged, revise the plan to remove or downgrade unsupported steps.
- If `audit_trace_budget` is run 3 times in a row, STOP and return only the steps that passed plus the steps that flagged and why they flagged.

## Output format
### Problem / request
- What is being requested and why?

### Evidence pack
> List `S0`, `S1`, ... and what each span represents.

### Repo understanding (cited)
- Short, cited summary of relevant architecture, modules, and constraints.

### Facts (cited)
- Only proven constraints/requirements.

### Decisions
- Explicit tradeoffs and chosen approach (cite if constrained by evidence).

### Assumptions / unknowns
- Any gaps or needed clarifications.

### Dry-run plan (exact file changes)
- Step-by-step plan including unit + integration tests and file paths.

### Approval request
- Ask the user to approve the plan before execution.

### Verification (plan trace)
- JSON array of `{idx, claim, cites}` for plan steps.

### Audit result
- Paste the `audit_trace_budget` summary + any flagged steps.

### Next evidence to collect
- If any Assumptions remain, list the exact file paths or commands needed to confirm them.
