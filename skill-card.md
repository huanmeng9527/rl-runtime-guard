## Description:

Pre-tool-call runtime guardrails for AI agents. Injects three soft prompt augmentations (complex_task_guard, retry_loop_guard, tool_guard) that catch an estimated 82.5% of common agent-fault errors before the model responds. Auditable, threshold-tunable, and disablable per session.

This skill is ready for commercial/non-commercial use.

## Publisher:

[huanmeng9527](https://clawhub.ai/user/huanmeng9527)

### License/Terms of Use:

MIT-0

## Use Case:

Operators of multi-step coding or file-operation agents use this skill to add a first line of defense against retry loops, complex-task overload, and tool-argument or path mismatches. Pairs naturally with the claw-rl-prm-judge skill to form a closed feedback loop where guard triggers feed offline scoring.

### Deployment Geography for Use:

Global

## Known Risks and Mitigations:

Risk: Soft prompt augmentation adds token overhead (roughly 150-400 tokens per request when a guard fires), which can increase latency and cost on every tool call.

Mitigation: Tune thresholds in references/thresholds.md for your workload. Disable globally or per session via the documented env var or config switch if overhead is unacceptable.

Risk: Default thresholds are calibrated for OpenClaw workstation usage; cloud or sandbox deployments with different message lengths or tool patterns may produce false positives or miss real failures.

Mitigation: Review the audit log format in references/audit-log.md and run the recommended companion skill (claw-rl-prm-judge) on guard-flagged turns to validate threshold choices before relying on the guards in production.

Risk: Guardrails are advisory only and never block requests or modify tool output. They cannot enforce policy on their own.

Mitigation: Use this skill as a soft signal layer; pair with OpenClaw ClawGuard's intent-verifier or a similar enforcement layer when blocking is required.

## Reference(s):

- [ClawHub skill listing](https://clawhub.ai/huanmeng9527/skills/rl-runtime-guard)
- [Official GitHub repository](https://github.com/huanmeng9527/rl-runtime-guard)

## Skill Output:

**Output Type(s):** [text, markdown, code, shell commands, configuration, guidance]

**Output Format:** [Markdown guidance with inline JSON config examples and ESM handler reference]

**Output Parameters:** [1D]

**Other Properties Related to Output:** [Reads config from a JSON file and an optional env var. Writes structured decisions to a local audit log. Does not perform network calls. The handler is pure Node.js stdlib and ships as a standalone ESM module.]

## Skill Version(s):

1.0.0 (source: server release evidence)

## Ethical Considerations:

Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment.
