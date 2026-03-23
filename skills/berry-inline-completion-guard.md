---
name: berry-inline-completion-guard
description: Review a tab-complete or inline suggestion using a Berry micro-trace audited against evidence spans. Use when verifying that a code completion is correct before accepting it — catches "almost right" suggestions that look plausible but are not evidence-backed.
---

You are reviewing an **inline completion** (tab-complete) that was just inserted into code.
Goal: catch "almost right" suggestions that look plausible but are not evidence-backed.

## Inputs
- Ask for (or use provided) spans: surrounding code, docstrings/contracts, failing test/log, and the completion itself.

## Rules
- Do not assume intent; if unclear, ask the user for the intended behavior.
- Do not claim safety/compatibility unless you can cite it.

## Verification step
- Write a micro-trace of 3–8 steps: `{idx, claim, cites}`.
- Call `audit_trace_budget(steps=..., spans=..., require_citations=true, context_mode='cited')`.
- If flagged: propose the smallest edit that makes the change evidence-consistent, or recommend rejecting it.
- If `audit_trace_budget` is run 3 times in a row, STOP and return only the claims that passed plus the claims that flagged and why they flagged.

## Output format
### What changed
- One paragraph summary of the completion's effect (no speculation).

### Evidence pack
> List spans `S0`, `S1`, ...

### Risk checklist
- Behavior change?
- Error handling / edge cases?
- Security / input validation?
- Performance / algorithmic complexity?
- API/ABI compatibility?
- Logging / observability?

### Micro-trace (JSON)
- A JSON array of `{idx, claim, cites}` justifying why the completion is correct.

### Audit result
- Paste the `audit_trace_budget` summary + flagged steps (if any).

### Verdict
- **Accept** / **Accept with edits** / **Reject**
- If edits: show the minimal patch diff.
