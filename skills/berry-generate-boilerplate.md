---
name: berry-generate-boilerplate
description: Generate tests, docs, config, migrations, or synthetic data with an auditable evidence-cited trace. Use when generating boilerplate where correctness must be verifiable — assumptions are explicit, APIs are cited.
---

You are generating **boilerplate/content** (tests, docs, synthetic data, config, migrations).
The artifact can be low-stakes, but **the assumptions must be explicit and checkable**.

## Evidence rules
- Collect an Evidence pack of spans `S0`, `S1`, ... for interfaces/contracts/expected behavior.
- Do not invent APIs, file paths, flags, config keys, or schema details unless cited.

## Verification strategy (use the right tool)
- Code blocks themselves are hard to verify sentence-by-sentence. Instead, verify the **design intent**.
- Produce a short **trace** of key claims (5–15 steps) that the artifact depends on.
- Each step must include `claim` + `cites: ['S#', ...]`.
- Call `audit_trace_budget(steps=..., spans=..., require_citations=true, context_mode='cited')`.
- If flagged: revise the trace AND the generated artifact until the trace passes, or downgrade items to Assumptions.
- If `audit_trace_budget` is run 3 times in a row, STOP and return only the claims that passed plus the claims that flagged and why they flagged.

## Output format
### Artifact request
- What are we generating (and for what target: language/framework/test runner/etc.)?

### Evidence pack
> List spans `S0`, `S1`, ...

### Constraints extracted from evidence
- Bullet the interface/behavior constraints you can prove. Each bullet must be cited.

### Generated artifact
- Provide the code/docs/config in code blocks.

### Verification trace (JSON)
- Output a JSON array of `{idx, claim, cites}` that justifies the artifact.

### Audit result
- Paste the `audit_trace_budget` summary + any flagged steps.

### Test/validation plan
- Minimal commands or checks that would validate the artifact works (cite when possible).
