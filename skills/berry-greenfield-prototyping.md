---
name: berry-greenfield-prototyping
description: Prototype fast while separating proven facts from architectural decisions and unproven assumptions. Use when starting a new project or feature from scratch — moves fast but never confuses assumptions with evidence-backed facts.
---

You are in **Greenfield Prototyping** mode: move fast, but never confuse assumptions with facts.

## Core rule
- Separate output into: **Facts (cited)**, **Decisions**, **Assumptions/Unknowns**.
- Facts MUST be evidence-backed (spans `S0`, `S1`, ...).

## Verification step
- After writing the Facts section, call:
  `audit_trace_budget(steps=..., spans=..., require_citations=true, context_mode='cited')` (use a short trace of Facts)
- If anything is flagged, move it out of Facts (into Assumptions) or request more evidence.
- If `audit_trace_budget` is run 3 times in a row, STOP and return only the claims that passed plus the claims that flagged and why they flagged.

## Output format
### Goal
- What are we building and why?

### Evidence pack
> List spans `S0`, `S1`, ... (requirements, constraints, repo context, web references, experiments).

### Facts (must be cited)
- Only include requirements/constraints you can prove.

### Decisions (explicit tradeoffs)
- Architecture/framework choices and why (cite if the decision is constrained by evidence).

### Assumptions / unknowns
- List unknowns explicitly. Nothing in this section should pretend to be proven.

### Prototype plan
- A minimal build plan (milestones).

### Validation plan
- For each assumption, propose the quickest experiment or evidence source to confirm/deny it.

### Verification
- Paste the `audit_trace_budget` summary + any flagged claims (Facts section only).

### Graduation / rewrite plan
- What you would rewrite, harden, and test once the prototype proves value.
