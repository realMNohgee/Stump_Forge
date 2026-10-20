# Failure Analysis — agent-session-reconciliation

## What I built

A two-stage Python pipeline (`ingest.py` -> `aggregate.py`) that reconciles a
stream of AI-agent session events into a final-status summary. The task is to fix
the broken pipeline so it produces the correct summary.

The pipeline has four defects:

1. A loud crash: `ingest.py` dies on a `cancelled` event.
2. A non-local seam bug: `ingest.py` silently drops the `ts` field, so
   `aggregate.py`'s correct-looking timestamp comparison degrades to "line order
   wins".
3. A predicate-scope bug: `completed` is sticky, so a session that completed then
   failed is still counted complete.
4. An idempotency bug: `events_processed` counts redelivered events twice.

The verifier runs the fixed pipeline against a held-out events file and
exact-matches the summary on all five fields. The environment's events file
triggers only the crash, so fixing the crash alone produces a correct-looking
summary; the three silent bugs fire only on the held-out data.

## Verification strategy

- Separate verifier container, binary reward, exact match on all five fields.
- Held-out data split: environment data exercises the loud bug, verifier data
  exercises the silent bugs.
- Ground truth computed by an independent reference implementation, never
  hardcoded.
- Result: all 27 static checks pass, oracle = 1.0, nop = 0.0.

## Iteration trail

I iterated across three domains and twelve distinct attempts. Every genuine task
was solved by DeepSeek v4-pro, a reasoning model weaker than the target frontier
models (gpt-6.1-sol xhigh and claude-opus-5.5 max):

| Attempt | Shape | Result |
|---|---|---|
| framing v1 | compute-from-spec takeoff | solved in 2m27s |
| framing v2 | coordinate-plan takeoff | solved in 4m50s |
| framing v3 | minimum-stock cutting-stock | solved in 20m18s |
| probe battery | 13 reasoning-shape prompts | 12/13 aced |
| trap probes | 4 single-concept traps | all solved |
| feed v1 | hinted diagnosis | solved in ~2.5m |
| feed v2 | discovery-chain diagnosis | solved in ~26m |
| MCP probe | conformance edge cases | 5/6 known, 1 churn |
| precedence probe | terminality rules | no clean seam |
| reconciliation v1 | coupled defect, local bugs | solved in 3.5m |
| reconciliation v2 | coupled defect, data-split | solved in 2m |
| reconciliation v3 | coupled defect, non-local seam | solved in 2.5m |

## The empirical finding

A reasoning model, given a clear specification and readable code, reconstructs
the correct behavior in minutes, every time. The levers that do not work:
transcription (a complete spec gets transcribed and solved), reasoning shapes,
rule-recognition traps, diagnosis chains, and coupled defects, whether local,
data-split, or non-local. The one thing that made it genuinely churn was thin
domain precedence knowledge (construction load-bearing rules), which is exactly
the kind of thing that is hard to make verifiable without sliding into
unverifiable judgment or obscure-trivia, both of which the rubric rejects.

## Why this matters

This is the real difficulty frontier. To stump a frontier reasoning model with a
genuine, verifiable, non-gimmick task, the difficulty has to come from thin
domain knowledge the model cannot reconstruct from a clear spec. That is a
narrow, hard-to-hit seam, and my honest conclusion is that this task does not
clear it. It passes every mechanical check and demonstrates the full pipeline,
but a stronger model than DeepSeek will solve it. I would rather say that
plainly than claim a 3/3 failure I did not observe.
