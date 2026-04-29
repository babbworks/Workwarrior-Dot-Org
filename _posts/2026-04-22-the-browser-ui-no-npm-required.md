---
layout: post
title: "The Browser UI: No npm Required"
date: 2026-04-22
categories: [technical, internals]
---

The browser UI is Python 3 stdlib only. No npm. No node_modules. No build step. No webpack, no bundler, no transpiler. If you can run Python 3, you can run `ww browser`.

That constraint — no external dependencies — shaped everything. It's not a limitation we worked around; it's a design requirement we built from.

## Why It Matters

A UI with an npm build pipeline has install friction. `npm install` fails for reasons unrelated to your code. Build steps break. Dependencies go stale. Someone with a fresh machine needs to debug package-lock.json before they see anything in the browser.

The alternative: a Python `ThreadingHTTPServer` that serves static HTML/CSS/JS from disk, a REST API of 20 endpoints, and a Server-Sent Events channel for real-time updates. The whole server is a single Python file and a directory of static assets.

Start it with `ww browser`. It's running on `localhost:7777` in under a second.

## The Architecture

```
services/browser/
  server.py         ThreadingHTTPServer, REST API, SSE
  static/
    index.html      Single-page app shell
    style.css       Dark terminal aesthetic, 15+ panel layouts
    app.js          Panel management, command routing, SSE client
```

`ThreadingHTTPServer` handles SSE connections without blocking. An SSE connection holds a socket open — the server pushes profile change events down that socket whenever something changes. Concurrent POST /cmd requests are handled in separate threads without queuing behind the SSE socket.

Static files are served directly from disk on each request. No caching, no preprocessing. Change a CSS file and refresh the browser — the change is visible immediately without a server restart.

## The Security Boundary

The biggest risk in a browser UI that executes system commands is remote code execution. `POST /cmd` executes a ww subcommand — that has to be constrained.

The constraint: an `ALLOWED_SUBCOMMANDS` frozenset. Every POST /cmd request is validated: the first token must appear in the frozenset. No `sh -c`. No eval. Unknown subcommands return 400.

This means the attack surface is bounded. An XSS in the UI can only execute valid ww subcommands. Valid ww subcommands can't exec arbitrary shell commands. The security model is explicit and auditable.

## What the Panels Show

15+ panels covering everything in the active profile:

- **Tasks** — full task list, inline editing, UDA display, start/stop/done buttons, annotation management
- **Time** — today's total, weekly breakdown, recent intervals, start/stop controls
- **Journals** — entry list with expand/collapse, new entry form, multi-journal dropdown
- **Ledgers** — account balances, recent transactions, income statement, balance sheet, new transaction form
- **CMD** — natural language + direct command input, route indicator (⚡ AI · ⚙ heuristic)
- **CTRL** — AI mode toggle, prompt settings
- **Sync** — GitHub sync dashboard
- **Weapons bar** — Gun, Sword, Next, Schedule icons

The SSE channel pushes profile switch events. Switch profiles from the CLI — the browser updates in real time.

## The Tradeoff

No npm means no React, no Vue, no TypeScript. The app.js is vanilla JavaScript. For 15+ panels with real-time updates and a command routing layer, that's manageable. It's not a framework — it's a single-page app written carefully in the language that runs in every browser without installation.

The performance is fine. The bundle size is zero. The install step for the browser UI is "you already have Python."
