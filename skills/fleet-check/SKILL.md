---
name: fleet-check
description: Give a short health overview of all the user's foro-hosted MCP servers. Use for "how are my servers doing", "anything broken overnight", "morning check", "status of everything". Not for digging into one server's errors (use debug-server).
---

# Check every server

The user's explicit instructions take priority over these steps.

1. **Pick the workspace.** If the user has several, `list_workspaces` and ask which one, or whether to check all of them.
2. **Collect.** Call `list_incidents` and `list_projects`, then `get_metrics` for each server that is `live` or `degraded`. Skip `get_metrics` for servers that have never been deployed.
3. **Report one line per server**, worst first: status, calls and error rate over the metrics window, p95, and any open incident. For example: "swift-harbor-a3f2: degraded, 1,204 calls, 6.1% errors, p95 840 ms, incident open since 03:12 on `search`".
4. **Flag what needs a look:** an open incident, a failed latest deploy, or an error rate or p95 that rose after the latest deploy. Then offer the next step: debug-server for errors, deploy-and-verify for a failed deploy.

Keep it short: a server that is fine gets one line and no commentary. If every server is fine, say so in one sentence.
