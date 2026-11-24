---
name: build-app-view
description: Build the interface (an MCP Apps view) for an MCP server the user deploys on foro, in their own repository, so it renders in ChatGPT, Claude and foro's Apps tab. Use for "give my tool a UI", "build a view or widget for my server", "make it work in ChatGPT and Claude", "my app renders blank". Not for deploying it (use deploy-and-verify) or for errors in the server's tools (use debug-server).
---

# Build an app view

The user's explicit instructions take priority over these steps.

This is written in the user's repository, so it needs a client that can edit their files. It follows foro's MCP Apps docs (https://foro.sh/docs/apps); `example-view.md` beside this file holds a complete view to adapt.

## The model

An app is one `ui://` resource plus every tool that points at it. It is not one tool with a UI attached: one view drives a whole workflow, the way the `todo-mcp` template (`list_templates`) puts five tools behind a single list. Start by asking which tools the view serves and what the user should see and do in it.

- The server registers the resource with MIME type `text/html;profile=mcp-app` and returns the view's HTML from it.
- Each tool the view renders sets `_meta.ui.resourceUri` to that URI (FastMCP: `@mcp.tool(app=AppConfig(resource_uri="ui://views/card"))`). ChatGPT also reads `_meta["openai/outputTemplate"]`; set both to the same URI.
- The view gets the tool's result as `structuredContent`, so the tool returns the data the view draws, not only text.

Nothing else changes on foro: a server that declares an app deploys exactly like one that doesn't.

## Hand-written HTML, or React with @foro/app

A view is one self-contained HTML document either way. Hand-written HTML (the todo template is one) is fine for a few interactions. Past that, write React with `@foro/app`, the runtime that lets one build run in the Apps tab, Claude and ChatGPT: the `design-mcp-view` skill in the foro SDK (https://github.com/foro-sh/foro) produces this shape: component, build, resource and the `_meta` that points a tool at it. Set the runtime up the way that skill does: `@foro/app` is not published to the public npm registry, so never install a package by that name from it.

- `views/<name>.tsx`: one default-exported component.
- `views/dist/<name>.html`: generated, with React, the runtime and the view inlined into one file. The server returns this file as the resource.

Hooks are the whole surface; nothing in the component branches on which host it runs in:

- `useToolInfo()`: the `input` and `structuredContent` of the call that mounted the view, its `_meta`, and `pending` while the result is on its way. Only MCP hosts report `pending`, so also treat a missing `structuredContent` as still loading.
- `useCallTool(name)`: `{ call, pending, error }` to call another tool on the same server. `call(args)` resolves with the tool result; check its `isError`.
- `useViewState()`: `[state, setState]`, view state that survives a re-render, since the sandbox has no `localStorage`. It is also what the host tells the model about the view, so keep it small and never put secrets in it. In MCP hosts it does not survive a remount; ChatGPT restores it.
- `useLayout()`: `theme` (light or dark), `maxHeight` and `safeArea` as the host reports them.
- `useDisplayMode()`: `[mode, request]` to read the display mode and ask for another (inline, fullscreen, pip). The host decides: the Apps tab grants all three and logs the request, chat stays inline, so read `mode` back rather than assuming the ask worked.
- `useOpenExternal()`: open a link through the host; the frame can't navigate itself.

Before the host has sent its context, the hooks answer with a light theme, inline display, no height limit and zero safe-area insets.

## Build, then commit the output

Build the view on the user's machine or in their CI and commit `views/dist/<name>.html` with the change, before pushing. A Python server's deploy on foro has no Node step, so a forgotten build silently deploys the previous view and the deploy still succeeds. (A Node server's deploy does run its package's `build` script, so there the view can be built as part of it instead.)

## The sandbox and its CSP

The view runs in an iframe that allows scripts and forms and nothing else: no same-origin access, an opaque origin, no `localStorage`, `sessionStorage` or `IndexedDB`. Keep state on the server, where its tools can reach it. An OAuth callback or an API key allowlisted to an origin can't work from inside the view, and a form submission can't leave the frame.

Every origin the view loads from or talks to must be declared in the resource's CSP, or the frame stays blank. This is the most common reason an app renders nothing:

- `resourceDomains`: scripts, styles, images, fonts and media. A bundled React view usually needs none, until it pulls a font or image from somewhere.
- `connectDomains`: `fetch`, XHR and WebSockets. Empty by default, and best left empty: draw from `structuredContent` and get fresh data through `useCallTool`, not by fetching it again from the view.

The Apps tab shows what a view declares under its frame. The Apps tab refuses a UI resource over roughly 1 MB rather than truncating it, so a larger view can't be checked there; a bundled React view starts near half of that, so watch large inlined images and libraries.

## Look and behaviour

- Follow the host: `theme` for colours, and stay within `maxHeight` (scroll inside the view rather than growing past it).

## Light and dark mode

A view that ignores the host's theme looks fine until a user in the other theme opens it. Full guide: https://foro.sh/docs/apps#light-and-dark-mode.

- Set the root `color-scheme` from the host's theme, on first render and whenever it changes: in React, from `useLayout().theme`; in hand-written HTML, from `hostContext.theme` on `ui/initialize` and on `ui/notifications/host-context-changed`. The ext-apps SDK has `applyDocumentTheme`, `applyHostStyleVariables` and `applyHostFonts` for this. `@foro/app` reports the theme but does not set `color-scheme` for you.
- Leave `html` and `body` transparent so the host's surface shows through, and declare `<meta name="color-scheme" content="light dark">`.
- Use the host's `--color-*` variables from `hostContext.styles.variables`, and ship fallback colours for hosts that send none. Claude's are `light-dark()` pairs that resolve once `color-scheme` is set.
- Never theme with `@media (prefers-color-scheme)` alone. Inside the frame it follows the visitor's operating system, not the host.
- A stray `*/` inside a CSS comment closes it early and drops the next rule, and the view falls back to white.
- Every state has a screen: loading while `pending` or before `structuredContent` arrives, an empty state when the result has no items, and an error message when a call fails, never a blank frame.
- Read the data from `structuredContent`; the view mounts with the result of the call that triggered it.

## What works where

The hooks work in every host. A few adaptor methods reached through `getAdaptor()`, the runtime's escape hatch, are ChatGPT-only (`openModal`, `uploadFile`, `selectFiles`, `getFileDownloadUrl`, `setOpenInAppUrl`): elsewhere they throw `NotSupportedError`. `download` is the reverse: it throws `NotSupportedError` in ChatGPT, and an error in a host that doesn't offer downloads. Catch these and fall back (hide the button, show a link) so the view degrades instead of breaking.

## Verify

1. Deploy with `deploy_project` and follow it with the deploy-and-verify skill.
2. Open the server's Apps tab in the foro dashboard. The tab mounts the view the way a host does, with the result of its first read-only (`readOnlyHint: true`) tool that takes no required arguments and is visible to the model (not `_meta.ui.visibility: ["app"]`); otherwise use "Try other data" on the Data tab to pick a tool yourself. Mounted cold, a view that renders from its tool result looks empty, so give a tool that lists what the view shows the read-only hint. The tab lights up each tool as the view calls it and logs every call with its arguments, result and timing, plus `ui/update-model-context`: what the view tells the model, usually the reason an agent seems not to know what the user just did.
3. Press Check theming on the Apps tab, with the Host switch on both Claude and Bare host. It runs the view in light and in dark and reports unreadable text, an opaque body background, or a missing `color-scheme`.
4. Then try it in a real host: `get_connect_instructions` gives the steps for ChatGPT, Claude and Cursor.

Tool calls the view makes go through foro's data protection and rate limit like any other call; the view's HTML is served as written.
