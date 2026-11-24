---
name: create-server
description: Put a new MCP server on foro from the library, a template, a GitHub repository, a published npm package or a folder on the user's disk, set its secrets, and deploy it. Use for "deploy my repo to foro", "host this npm server", "start a new server from a template", "upload this project", "deploy a ready-made server". Not for redeploying or fixing a server that already exists (use deploy-and-verify or debug-server).
---

# Create a server on foro

The user's explicit instructions take priority over these steps.

1. **Pick the workspace.** Tools take the workspace per call. With several, `list_workspaces` and ask which one.
2. **Ask where the server comes from**, unless the user already said:
   - **The library:** ready-made servers, no GitHub, GitLab or Codeberg account. `list_library`, let the user pick, then `create_project_from_library`. It is created without deploying. If the reply names env vars, `deploy_project` sends the user to set a required one; never ask for a value in chat.
   - **A template:** `list_templates`, let the user pick, ask for a name (3-48 lowercase letters, digits or dashes), then `create_project_from_template`. It copies the template into GitHub (the user's account, or the workspace's GitHub App) and its first deploy is already running: skip to step 4. If that deploy fails for a missing key, `request_secrets` with the name its log shows, then deploy again.
   - **A repository:** `list_repos` and let the user pick. Call `create_project_from_repo` with the repo's default branch unless they name another, and `path` set to the directory holding `pyproject.toml`, `package.json` or `Cargo.toml` when it isn't the root. A repo with none of them is not something foro can build: say so.
   - **A published npm package:** `preview_npm_package` with the bare package name (never a pasted install line with keys in it). Show the user the version, the command it would run and the env var names, then `create_project_from_npm` with the exact version the preview returned. No GitHub or git account is needed.
   - **A folder on the user's disk with no repository:** only when you can run shell commands on that machine (Claude Code, Cursor, an IDE agent). Call `create_upload`, run the command it returns from the project root, and read the new project's `slug` from the JSON it prints. The URL works once and expires after ten minutes; env files are never uploaded. In a chat-only client, ask the user to push the project to a repository instead.
3. **Secrets.** Never ask for a value in chat or accept one pasted there: `request_secrets` sends the user to a page where they type it.
   - **An npm package:** call `get_project`. When missing_env lists names, call `request_secrets` with exactly those names and wait until the user says they have saved, then `deploy_project`; it refuses and names any key still unset.
   - **A repository or upload:** foro only knows the env vars the project declares in its `env` config table (missing_env lists the unset required ones; names from an example env file are optional and never block). Code that reads others is not stopped. Ask the user which names the server needs, or take them from the project's README, call `request_secrets` with those names, and wait until the user says they have saved before deploying. A key missed here shows up as a failed deploy whose log names it. Declaring them in the `env` table makes the next deploy refuse until they are set.
4. **Deploy.** `deploy_project` (not for a template project, which is already deploying). A library project deploys the catalog's pin. Then continue with the deploy-and-verify skill to follow it to live, explain a failure and connect the server to a client.
5. **Offer auto-deploy** for a GitHub project (a GitHub repository or a template) on the Starter plan or above: `set_auto_deploy` makes every push to its branch deploy. Only turn it on when the user wants it.

A creation that is refused names the reason; pass it on rather than retrying. The common ones are the workspace's plan limit on servers (offer to free a slot: ask which of their projects to delete, then `delete_project` only on their explicit go-ahead, or point to the dashboard's billing page; never change the plan yourself) and a repository foro can't reach (the user connects git access in the dashboard).
