---
name: security-check
description: Check the user's foro-hosted MCP servers for known vulnerabilities in their dependencies and say how to fix them. Use for "is my server safe", "any vulnerabilities", "CVEs", "security scan", "why was my deploy blocked by the scan". Not for runtime errors (use debug-server) or store-listing requirements (use prepare-for-store).
---

# Security check

The user's explicit instructions take priority over these steps.

1. **Pick the servers.** One named server, or all of them: `list_projects` and check each that is `live` or `degraded`, or `building` (a redeploy in flight: the previous image is still serving).
   - **"Why was my deploy blocked?"** is about the new image, which never went live, while `get_image_scan` reports the live one. Call `get_deployment` for the blocked deploy instead: its error names the open findings at or above the blocking severity, or says the scan failed or didn't finish (deploy again for those), and after a completed scan of a repository or upload project it carries a ready-made fix prompt. Then give the fix as in step 5.
2. **Read the scan.** Call `get_image_scan` per server. It covers the server's own dependencies only: the foro base image and runtime are not part of this scan, so never present its result as a verdict on them.
   - **pending:** the scan of the live image is still running; check again in a minute. The result always covers the live image: for a `building` server, the deploy in flight is checked through `get_deployment` once it finishes.
   - **failed** or **none:** there is no result to judge by (none means the live deployment has no scanned image yet). Say so instead of calling the server clean.
3. **Report worst first.** Lead with critical and high findings: the CVE id, the package, the installed version and the version that fixes it, or that no fix exists yet. Group medium and low into counts unless the user asks for them. Leave out findings the result lists as dismissed: the user already decided on them.
4. **Say whether it blocks deploys.** The result names the severity at or above which an open finding blocks a deploy (off blocks nothing). With blocking on, a deploy is also refused when its own scan failed or didn't finish.
5. **Give the concrete fix.**
   - **A repository or upload project:** bump each package to its fixed version in the project's manifest and lockfile. If you are working in the user's repository, make the change there with their agreement; otherwise hand over the ready-made prompt for their coding agent when the result carries one (it lists the same findings). Then the change needs a deploy: `deploy_project` once it is pushed (a push deploys it by itself with auto-deploy on), or for an upload project a new `create_upload` with the project's slug first. Check `get_image_scan` again once the deploy is live.
   - **An npm package project:** the vulnerable dependency ships inside the published package, so the fix is a newer release from its publisher. Say that plainly.
   - **No fix available:** say so, and that dismissing it with a reason, or changing the blocking severity, is done on the dashboard's security page the result links to.

A clean scan means no known vulnerabilities in the scanned dependencies, nothing more: it says nothing about the server's own code, how it handles secrets, or what its tools let a caller do. Never call a server "safe" on the strength of it.
