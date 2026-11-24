---
name: get-started
description: Set up foro right after the user connects it, by showing their workspaces and servers and suggesting a first step. Use for "get started with foro", "set up foro", "what can foro do", or right after install. Not for creating, deploying or debugging a specific server (use create-server, deploy-and-verify or debug-server).
---

# Get started with foro

The user's explicit instructions take priority over these steps.

1. **Check the connection.** Call `list_workspaces`. If the user has no workspace yet, suggest a name (the login it gives, or their team or company), wait for them to confirm it, then call `create_workspace`. It also adds an example project, which is a demo and not their server. If it says foro isn't enabled for the user's workspaces yet, pass that on and stop.
2. **Pick the workspace.** With one workspace, use it. With several, list them by name and ask which to look at. Tools take the workspace per request, so tell the user they can name another one at any time.
3. **Show what's there.** Call `list_projects` for that workspace.
4. **Suggest one next step** based on what you found:
   - **No servers yet:** offer to create one from the library (no git account needed), a template, one of their repositories, a published npm package, or a folder on their disk, and continue with the create-server skill. Creating a project asks for confirmation.
   - **A server whose last deploy failed:** offer the deploy-and-verify skill to explain it.
   - **Live servers:** offer the fleet-check skill for an overview, or `get_connect_instructions` to add one to ChatGPT, Claude or Cursor.
5. **Close with three things they can ask next**, phrased as prompts, e.g. "Why did my last deploy fail?", "Why is my server throwing errors this week?", "Is my server ready for a store listing?".

Never ask for secrets, tokens or passwords: the user sets a secret in the foro dashboard, never in chat.
