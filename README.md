# n8n 2.x migration breakage: Start-node removal — reproduce, diagnose, fix

> **Lab demo, not a client case study.** Built to show how I diagnose n8n version-migration failures. No client data involved.

## What broke

n8n 2.0 removed the **Start** node — [official announcement](https://blog.n8n.io/introducing-n8n-2-0/), migration guide (`docs/changelog/v20-breaking-changes.md`): *"Start node removed"*. Manual executions move to the **Manual Trigger** node.

The same workflow JSON, two versions:

| n8n version | result | log |
|---|---|---|
| 1.114.2 | ✅ `Execution was successful` | `logs/success_1x.log` |
| 2.3.0 | ❌ `Missing node to start execution` | `logs/failure_2x.log` |

```
Missing node to start execution
Please make sure the workflow you're calling contains an Execute Workflow Trigger node
```

## Diagnosis

The failure happens at CLI startup-node selection: 1.114.2's executable-start list still includes `start`; 2.3.0 dropped the node implementation, so the workflow has no valid entry point and the run never starts. The error text alone doesn't tell you how the UI behaves — the root cause was confirmed against the installed package, not guessed from the message.

## Fix (official migration path)

Replace Start with **Manual Trigger** (`workflow_start_node_fixed.json`) — the exact remedy from n8n's own migration guide. Re-ran on 2.3.0: ✅ `Execution was successful` (`logs/success_2x_fixed.log`).

## Files

- `workflow_start_node.json` — the breaking workflow (identical file fails on 2.x)
- `workflow_start_node_fixed.json` — fixed per the official migration path
- `logs/` — real CLI run logs for all three runs (self-hosted n8n via `n8n execute`, no Web UI, not n8n Cloud)

## How I work

Reproduce → locate the exact breaking-change clause → apply the smallest official fix → verify in the same environment. I don't touch API keys or production data; staging only.
