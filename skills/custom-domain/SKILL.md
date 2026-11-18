---
name: custom-domain
description: Put one of the user's foro-hosted MCP servers on a domain they own, explain why one isn't working yet, or remove it. Use for "use my own domain", "mcp.mycompany.com", "my custom domain is still pending", "the domain doesn't load", "remove the domain". Not for connecting a server to ChatGPT, Claude or Cursor (use deploy-and-verify) or for a server that is failing on its foro.sh URL too (use debug-server).
---

# Custom domain

The user's explicit instructions take priority over these steps.

A custom domain is an extra hostname for a server: its `https://<slug>.foro.sh` URL keeps working. A public or token endpoint takes the same credentials on both. An OAuth endpoint names its foro.sh URL as the resource clients get tokens for, so connect OAuth clients through that URL rather than the custom domain. It needs the Starter plan or above.

1. **Find the server** with `list_projects` if it isn't named, and call `get_project` to see whether it already has a domain and in what state.
2. **Set it.** Call `set_custom_domain` with a plain hostname (no scheme, port, path or wildcard). Setting a different domain replaces the current one and restarts verification; say so before replacing a verified one.
3. **Show the DNS records exactly as returned**, as a table of type, name and value: a TXT record that proves the user controls the domain, and a CNAME pointing the domain at the server. An apex domain (`example.com` itself) cannot take a CNAME: the user needs their DNS provider's ALIAS or CNAME flattening, or an A record as the docs link in the result describes. Then wait until the user says they have added them.
4. **Verify.** Call `verify_custom_domain`.
   - **Still pending or failed:** the TXT record wasn't found yet. The usual causes, in order: DNS hasn't propagated (minutes, sometimes longer: retry later), the record was added under the wrong name (many DNS panels append the domain, so `_foro-challenge.mcp.example.com` becomes `_foro-challenge.mcp.example.com.example.com`), or it was added as the wrong type. Check those with the user, then verify again.
   - **Verified:** it still doesn't serve until the next deploy, and only once the CNAME resolves to foro: a CNAME behind a proxy (Cloudflare's orange cloud, for example) resolves to the proxy instead, and the certificate can't be issued. `get_project` and `verify_custom_domain` report whether the domain is serving; if it is verified but not serving, offer `deploy_project`, then follow it with the deploy-and-verify skill. After that deploy, `https://<the domain>/mcp` serves the server with its own certificate.
5. **Remove** only when the user asks: confirm first, then `remove_custom_domain`. A verified domain keeps serving until the next deploy; offer `deploy_project` to stop routing it. The foro.sh URL is unaffected.

"Domain is already in use" means another project has that hostname; it has to be removed there first. Never ask for DNS provider credentials: the user adds the records themselves.
