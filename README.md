# TB3 Task — agent-session-reconciliation

A Terminal-Bench 3 task, built for the Klavis AI Founding Engineer assignment.

## The task

`tasks/agent-session-reconciliation/` is a two-stage Python pipeline
(`ingest.py` -> `aggregate.py`) that reconciles a stream of AI-agent session
events into a final-status summary. The pipeline has four planted defects, and
the agent must fix them so the pipeline produces the correct summary.

See `tasks/agent-session-reconciliation/instruction.md` for the task instruction.

## Check results

Run with harbor CLI v0.24.0 and Docker (colima) on macOS.

- **Static checks:** 27/27 pass
- **Oracle validation:** reward 1.0
- **Nop validation:** reward 0.0

## Trial results

Trials were run against DeepSeek v4-pro (deerflow agent), a reasoning model
weaker than the assignment's target models. Every version was solved:

| Version | Shape | Result |
|---|---|---|
| v1 | coupled defect, local bugs | solved in 3m34s |
| v2 | coupled defect, data-split | solved in 2m04s |
| v3 | coupled defect, non-local seam | solved in 2m28s |

The frontier trials (codex gpt-6.1-sol xhigh, claude opus-5.5 max) were not run;
see the failure analysis for why.

## Failure analysis

`FAILURE-ANALYSIS.md` documents the full iteration trail (12 attempts across 3
domains), the verification strategy, and the empirical finding.

## Commands

```
# static checks (from a terminal-bench checkout)
for c in scripts/checks/check-*.sh; do bash "$c" tasks/agent-session-reconciliation; done

# oracle and nop
harbor run -p "tasks/agent-session-reconciliation" -a oracle
harbor run -p "tasks/agent-session-reconciliation" -a nop

# cheap-model trial
harbor run -p "tasks/agent-session-reconciliation" -a deerflow -m deepseek/deepseek-v4-pro
```
