---
layout: post
title: "Services All the Way Down"
date: 2026-04-29
categories: [architecture, technical]
---

Everything in Workwarrior is a service. The CLI dispatcher (`bin/ww`) routes every command to a service script in `services/<category>/`. The browser UI is a service. The heuristic compiler is a service. Profile management is a service. Even the weapons are discovered and dispatched as services.

This architecture wasn't a design decision made upfront. It emerged from the requirement that the system be extensible — that adding a new capability shouldn't require touching core files.

## The Dispatcher

`bin/ww` is 709 lines. It's the single entry point for every command. Its job is to:

1. Check that a profile is active (or that the command is one of the profile-agnostic ones)
2. Find the service script for the command
3. Validate arguments at the routing level
4. Execute the service with the remaining args

The discovery mechanism: scan `services/` for executables matching the command name. Profile-level services at `profiles/<name>/services/<category>/` are checked first — they shadow global services. If no service is found, print an error and exit 1.

This means adding a service is a file operation, not a code change. The dispatcher doesn't have an explicit registry of known services. It discovers them from the filesystem.

## The Service Contract

Every service must:
- Respond to `--help` / `-h` with a one-line description and usage example
- Use exit codes: 0 success, 1 user error, 2 system/internal error
- Log via `lib/logging.sh`, not raw `echo`
- Not write to profile directories directly — call lib functions

The help output is the documentation. `ww help` shows all discovered services with their one-line descriptions, pulled from `--help` output at startup. A service that doesn't implement `--help` is invisible to the help system.

Exit codes matter for the browser UI. A `POST /cmd` request checks the exit code. Exit 1 means the user made an error — show the error message. Exit 2 means a system problem — show a different error and don't retry automatically.

## The Browser Service

`services/browser/` is the most complex service. It starts a Python HTTP server, manages its process, and shuts it down. It's also the only service that runs persistently — every other service runs, completes, and exits. The browser service starts a background process and returns immediately.

That asymmetry is handled by the service contract: `ww browser` returns 0 if the server started successfully. Status checks use `ww browser status`. Shutdown uses `ww browser stop`. The dispatcher doesn't need to know the browser service is different — it just routes the subcommands.

## Why This Matters for Extensions

The service architecture means Workwarrior is extensible at the seam that matters. You can add a service without touching the dispatcher. You can override a service per-profile without modifying the global service. You can inspect the full service inventory from `ww help` without reading source code.

New weapons are services. New data integrations are services. New export formats are services. The 25+ service domains in the current release grew this way — each one is a directory under `services/`, each one follows the same contract, each one was added without modifying the ones that existed before it.

The service architecture is not a framework. There's no base class to inherit, no decorator to register. It's a filesystem convention and a contract. Conventions are simpler than frameworks, break less often, and are easier to reason about when something goes wrong.

Everything is a service. When in doubt, make it a service.
