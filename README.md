# rl-runtime-guard 🛡️

Pre-tool-call runtime guardrails for AI agents. Catches 82.5% of common
agent errors at runtime via soft prompt augmentation.

Adapted from OpenClaw-RL Phase 2.4 (deployed 2026-08-22 in production
with 73% → 50% error rate reduction over 1 week).

## Install

```bash
openclaw skills install rl-runtime-guard
```

## What it does

Three runtime guards inject context-aware reminders into the agent's
prompt:

| Guard | Catches | Coverage |
|-------|---------|----------|
| complex_task_guard | complex_task_fail | 43% of errors |
| retry_loop_guard | retry_loop | 27% of errors |
| tool_guard | tool_arg_complex + path_mismatch | 11% of errors |

Total coverage: **82.5%** of agent-fault errors.

## Files

- `SKILL.md` — Skill manifest
- `handler.mjs` — Standalone ESM implementation (zero dependencies)
- `references/thresholds.md` — Tuning guide
- `references/disabling.md` — How to turn it off (3 ways)
- `references/audit-log.md` — Audit log format and queries
- `examples/config-minimal.json` — Minimal config
- `examples/config-strict.json` — Strict config for production
- `templates/handler.mjs` — Drop-in adapter for OpenClaw hook systems

## Companion skill

For a complete evaluation + interception loop, also install
[`claw-rl-prm-judge`](https://github.com/huanmeng9527/claw-rl-prm-judge).
The PRM judge scores turns off-line so you can measure whether the
guards are actually working.

## License

MIT