---
name: berry-rca-fix-agent
description: Evidence-first 6-phase root cause analysis and verified fix loop. Use when debugging a bug or incident where the root cause is unclear — never decides root cause from vibes, all claims must be evidence-backed.
---

# RCA Fix Agent (verified)

Use this workflow when debugging something and shipping a fix (failing test, incident, broken build, etc.).

**Key rule: never decide root cause from vibes.** All claims must be evidence-backed.

## Evidence Pack
Maintain spans `S0`, `S1`, ... where each span is raw evidence with source (file:lines, command output, URL).

## Minimum verification claims
You must verify these with `audit_trace_budget`:
1. **ROOT_CAUSE**: "The issue is because of X." [cited]
2. **FIX_MECHANISM**: "The fix works because it changes X which prevents Y." [cited]
3. **FIX_VERIFIED**: "The original repro now passes." [cite test output]
4. **NO_NEW_FAILURES**: "The regression suite passes." [cite test output]
If `audit_trace_budget` is run 3 times in a row, STOP and return only the claims that passed plus the claims that flagged and why they flagged.

## Workflow

### Phase 1 — Baseline
1. Run the repro command before any changes
2. Capture failure signal (test names, stack traces, errors) as spans
3. Identify closest code (file/line from trace) and add as spans

### Phase 2 — Hypotheses (be thorough)
1. **Generate as many root-cause hypotheses as possible** (aim for 5+)
   - Don't stop at the obvious answer
   - Consider: config issues, race conditions, edge cases, upstream changes, env differences
2. For each hypothesis: write predictions and discriminating experiments
3. **Run experiments and collect evidence BEFORE verification**
   - Run smallest experiments first
   - Add ALL outputs as spans (even negative results are evidence)
   - The more evidence you gather now, the better verification will work
4. Pick leading hypothesis as PRIMARY CLAIM only after experiments narrow it down
5. **Verify ROOT_CAUSE claim before implementing fix**
   - If flagged: gather more evidence or downgrade to hypothesis

### Phase 3 — Fix Plan
1. Write fix plan: files to change, invariant restored, tests to run
2. List likely failure modes the fix might introduce
3. Define a check for each failure mode

### Phase 4 — Implement + Test
1. Implement the fix
2. Run test plan, capture outputs as spans
3. If repro still fails: update hypotheses, go to Phase 2
4. If repro passes: run regression checks

### Phase 5 — Verification
1. Draft report with `[S#]` citations
2. Run `audit_trace_budget` on claims
3. If flagged: gather more evidence, revise claims, or add tests

### Phase 6 — Deliverables
Output:
- Root cause (verified)
- Fix summary
- Test plan + results
- Evidence that fix works
- Known risks (explicitly marked)

## Stop conditions
Stop when: original repro passes, regression checks pass, and minimum claims are not flagged.
If any are false, continue iterating.
