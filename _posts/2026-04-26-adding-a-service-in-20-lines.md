---
layout: post
title: "Adding a Service in 20 Lines"
date: 2026-04-26
categories: [technical, open-source]
---

The service architecture is designed to be extended. A new service is an executable script in `services/<category>/`. The `ww` dispatcher discovers it by scanning for executables. No registration step. No config update. Write the script, make it executable, it's available as `ww <category>`.

Here's a minimal service in full:

```bash
#!/usr/bin/env bash
set -euo pipefail

source "$WORKWARRIOR_BASE/lib/logging.sh"

USAGE="ww myservice — one-line description
Usage: ww myservice <list|info|add>
"

case "${1:-}" in
  --help|-h)
    echo "$USAGE"
    exit 0
    ;;
  list)
    # implementation
    log_info "Listing things..."
    ;;
  info)
    [[ -z "${2:-}" ]] && { log_error "info requires a name"; exit 1; }
    log_info "Info for: $2"
    ;;
  *)
    log_error "Unknown subcommand: '${1:-}'"
    echo "$USAGE"
    exit 1
    ;;
esac
```

That's the contract. Five requirements:
1. `#!/usr/bin/env bash` + `set -euo pipefail`
2. Responds to `--help` / `-h`
3. Exit codes: 0 success, 1 user error, 2 system error
4. Logs via `lib/logging.sh`
5. Doesn't write to profile directories directly — calls lib functions

The dispatcher (`bin/ww`) routes `ww myservice list` to the `list` case branch. The help output appears in `ww help`. The service is available from the browser UI CMD panel immediately.

## Profile-Level Services

Services at `profiles/<name>/services/<category>/` shadow global services. This means you can have a per-profile version of any service. `ww myservice` in the `work` profile runs the work-profile version; in the `personal` profile it runs the global version.

This is useful for per-profile customizations that shouldn't affect other contexts. A work profile might have a specialized export service. A client profile might have a service that integrates with the client's specific tooling.

## Template Tiers

`docs/guides/services/service-development.md` describes three service tiers:

**Tier 1 — Simple** — single-file script, direct TaskWarrior/TimeWarrior calls. Appropriate for services that wrap a single tool command or do simple data reads.

**Tier 2 — Compound** — multiple subcommands, uses lib functions for profile management. Appropriate for services with lifecycle (create/list/delete/backup patterns).

**Tier 3 — Complex** — state management, external APIs, background processes. Appropriate for services like github-sync that maintain persistent state across invocations.

Most new services start at Tier 1 and grow if needed. The pattern is identical across tiers — the difference is how much of the lib you reach into.

## Why This Works

The architecture is a consequence of the profile model. Because everything resolves through environment variables, a service doesn't need to know which profile it's running against. It reads `$WORKWARRIOR_BASE`, calls lib functions with the right paths, and the isolation is handled automatically.

This is also why services are safe to contribute. A new service in `services/my-extension/` touches nothing that already exists. It can't break the sync engine. It can't break the browser UI. It can't break the CLI dispatcher (unless it introduces a naming conflict, which the dispatcher checks for at startup).

The extension surface is designed to be safe. Build services. Build weapons. Build profile-level customizations. The architecture gets out of the way.
