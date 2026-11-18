---
name: usage-and-clients
description: Say who uses one of the user's foro-hosted MCP servers, from which clients, how agents behave in their sessions, and whether the workspace is heading past its plan's monthly tool-call limit. Use for "who uses my server", "which clients connect", "how many users", "am I near my limits", "will I run out of calls". Not for errors or slowness (use debug-server) or a health overview of every server (use fleet-check).
---

# Usage and clients

The user's explicit instructions take priority over these steps.

1. **Find the server** with `list_projects` if it isn't named.
2. **Who and from where.** Call `get_usage`: calls per client (ChatGPT, Claude, Cursor, ...), how many distinct signed-in callers it had over the detail window, and the workspace's tool-call count against its plan's monthly limit. Distinct callers are counts, never identities, and only an OAuth endpoint names its callers: a token or public endpoint shows clients but no callers, which is not the same as nobody using it.
3. **How agents use it.** Call `get_sessions` for the recent sessions: client, calls, errors, and the flags loop (the same call over and over), retry (a failed call straight away repeated) and ended_on_error (its last call failed). Open one flagged session with `get_sessions` and its session id to show the calls in order. A session id starting `syn_` was inferred by foro, not sent by the client.
4. **The trend.** Call `get_metrics` for call volume over its window, to say whether usage is growing, flat or falling.
5. **Answer, then the limit.**
   - One or two lines on who uses it: the main clients with their share, the number of distinct callers, and anything a flagged session shows (an agent looping on one tool is usually a tool description or error message it misreads).
   - **The limit.** Give used of limit, and project the month: if this month's pace (calls so far divided by days elapsed, times the days in the month) passes the limit, say roughly when. What crossing it means depends on the plan (`list_workspaces` shows it): on Free it stops every server in the workspace except foro's example project, and they stay stopped until the user upgrades, or redeploys them once the count resets on the 1st, so a projected overrun is worth acting on now; on Starter and Team it is a fair-use figure, and traffic keeps running past it. The remedies are a larger plan through the dashboard's billing page, or fewer calls (fixing a looping agent is often the cheapest saving). Never change the plan yourself.

The figure is per workspace, so it counts every server in it, and `get_usage` on any of them shows the same total. foro's example project never reports one: when `get_usage` says there is no cap, check a server of the user's own before telling them their plan has none.
