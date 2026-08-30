# Audit Log Format

The guard writes one JSON object per line to:
`~/.openclaw/logs/audit/rl-guard-decisions.jsonl` (configurable).

## Schema

```typescript
type AuditEntry = {
  timestamp: string;         // ISO 8601 UTC
  sessionKey: string;        // session identifier (anonymized by caller)
  decisions: string[];       // list of guards that fired
  userContentPreview: string; // first 80 chars of user message
  guardPromptLength: number; // bytes of injected prompt
  error?: string;            // only present if the guard failed
};
```

## Example entries

```jsonl
{"timestamp":"2026-08-22T19:12:34.123Z","sessionKey":"agent:main:abc123","decisions":["complex_task","platform_path_hint"],"userContentPreview":"请帮我部署这个项目到 ~/Desktop，配置好后执行以下步骤：1. build 2. test 3.","guardPromptLength":1240}
{"timestamp":"2026-08-22T19:13:01.456Z","sessionKey":"agent:main:abc123","decisions":["retry_loop"],"userContentPreview":"还是不行啊，重新跑一下","guardPromptLength":620}
{"timestamp":"2026-08-22T19:14:00.000Z","sessionKey":"agent:main:abc123","decisions":[],"userContentPreview":""}
```

Wait — entries with empty decisions only happen if the guard fired an
error. Most requests that don't match any pattern produce no audit entry.

## Common queries

### How often is each guard firing?

```bash
jq -r '.decisions[]' ~/.openclaw/logs/audit/rl-guard-decisions.jsonl \
  | sort | uniq -c | sort -rn
```

Output:
```
  145 retry_loop
   87 complex_task
   23 platform_path_hint
```

### Which sessions have the most guards?

```bash
jq -r '"\(.sessionKey) \(.decisions | length)"' ~/.openclaw/logs/audit/rl-guard-decisions.jsonl \
  | awk '{s[$1]+=$2} END {for (k in s) print s[k], k}' | sort -rn | head
```

### Error rate in guard execution

```bash
jq -r 'select(.error != null) | .error' \
  ~/.openclaw/logs/audit/rl-guard-decisions.jsonl | sort | uniq -c
```

### Time-window filtering

```bash
# Last 24 hours
START=$(date -u -d '24 hours ago' +%Y-%m-%dT%H:%M:%SZ)
jq -r "select(.timestamp > \"$START\") | .decisions[]" \
  ~/.openclaw/logs/audit/rl-guard-decisions.jsonl | sort | uniq -c
```

## Privacy notes

The audit log contains:
- **sessionKey** — opaque identifier, no PII unless you put it there
- **userContentPreview** — first 80 chars of user messages (may contain
  sensitive info)

If you're deploying in a privacy-sensitive environment:

1. Set `auditLogPath` to a path inside a restricted directory
2. Or set `auditLogPath` to `/dev/null` to disable logging entirely
3. Add the path to your log rotation policy (default retention: 30 days
  recommended)

## Log rotation

The guard never rotates its own log. Use `logrotate` or equivalent:

```
# /etc/logrotate.d/openclaw-rl-guard
~/.openclaw/logs/audit/rl-guard-decisions.jsonl {
    daily
    rotate 30
    compress
    missingok
    notifempty
}
```