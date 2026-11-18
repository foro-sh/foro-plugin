# foro

Deploy, debug and monitor the MCP servers you host on [foro.sh](https://foro.sh).

foro hosts MCP servers in the EU: you sign in with GitHub, pick a repository, and get a stable `https://<name>.foro.sh` URL. This plugin lets you run that from Claude. List your servers, deploy or roll back, follow a deploy to live and see why one failed, read and analyze runtime logs, check calls, errors and latency, see who uses a server, and check whether it is ready for a store listing.

It is for developers who already host MCP servers on foro.sh, or who want to start. It acts with your own foro account and only in the workspaces you are a member of.

## Skills

| Skill | What it does |
| --- | --- |
| `get-started` | Shows your workspaces and servers right after you connect, and suggests a first step. |
| `create-server` | Puts a new server on foro from the library, a template, a repository, an npm package or a folder, sets its secrets and deploys it. |
| `deploy-and-verify` | Deploys, rolls back or cancels a deploy, follows it to live, and explains one that failed. |
| `debug-server` | Finds out why a server is failing, erroring or slow, and proposes a fix. |
| `fleet-check` | Gives a short health overview of all your servers. |
| `usage-and-clients` | Says who uses a server and from which clients, and whether you are heading past your plan's monthly tool-call limit. |
| `security-check` | Checks your servers' dependencies for known vulnerabilities and says how to fix them. |
| `custom-domain` | Puts a server on a domain you own, or explains why one isn't working yet. |
| `prepare-for-store` | Checks whether a server is ready for the Claude, ChatGPT, MCP registry, Cursor and other listings, and writes the fixes. |
| `build-app-view` | Builds the interface for a server's tools (an MCP Apps view) in your own repository. |

## Connections and data

- **One destination.** The plugin connects only to the foro MCP server at `https://mcp.foro.sh/mcp`. It makes no other network connection of its own, and it ships no scripts, hooks or commands.
- **Sign-in.** You sign in with OAuth under your own foro account the first time a tool runs. The plugin never sees your password, and you can disconnect at any time from your foro account.
- **Permissions.** The connection asks for the `read`, `deploy` and `write` scopes. Everything the tools do, foro's API decides under your account: workspace membership, plan limits and seats apply exactly as in the dashboard.
- **Secrets stay out of chat.** The plugin never reads, asks for or repeats secrets, tokens or passwords. When a server needs a secret, it sends you to a foro page where you set it yourself.
- **It asks first.** Creating a project, deploying, rolling back, cancelling a deploy and deleting a project all ask you to confirm before they run.
- **Limits.** foro's tools cannot edit your repository's code. The `build-app-view` skill writes files only through your client's own file tools, in a checkout you have open. Billing and team management stay in the foro dashboard.

## Learn more

- Tool reference: [docs.foro.sh/foro-mcp](https://docs.foro.sh/foro-mcp)
- Privacy policy: [foro.sh/privacy](https://foro.sh/privacy)
- Terms of service: [foro.sh/terms](https://foro.sh/terms)
- Support: [foro.sh/support](https://foro.sh/support)

## License

MIT, see [LICENSE](LICENSE).
