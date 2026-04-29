---
layout: page
title: Extensions
eyebrow: TaskWarrior + TimeWarrior Ecosystem
subtitle: Workwarrior ships with a registry scanner that finds, indexes, and installs community extensions for both tools.
permalink: /extensions
---

TaskWarrior and TimeWarrior both support community-built extensions — hooks, reports, themes, integrations. The `ww extensions` service maintains a live registry of these and makes them installable from the CLI.

```bash
ww extensions taskwarrior list      # Browse TaskWarrior extensions
ww extensions taskwarrior search <term>
ww extensions taskwarrior info <name>
ww extensions taskwarrior refresh   # Update registry from upstream
```

---

## What Extensions Do

**TaskWarrior hooks** run at specific lifecycle events:

| Event | When It Fires |
|-------|--------------|
| `on-launch` | Every `task` invocation |
| `on-add` | New task creation |
| `on-modify` | Task change |
| `on-exit` | After `task` finishes |

Workwarrior itself uses `on-modify` hooks to auto-start TimeWarrior when a task is started.

**TimeWarrior extensions** are Python scripts that run against time data — custom reports, summaries, integrations.

---

## Building a Service

Workwarrior's own service architecture is designed for extension. A new service is an executable script in `services/<category>/` that:

- Responds to `--help` / `-h`
- Uses exit codes 0/1/2
- Logs via `lib/logging.sh`
- Doesn't write to profile directories directly

```bash
# Minimal service skeleton
#!/usr/bin/env bash
set -euo pipefail
source "$WORKWARRIOR_BASE/lib/logging.sh"

case "${1:-}" in
  --help|-h)
    echo "ww myservice — description"
    echo "Usage: ww myservice <subcommand>"
    ;;
  list)
    # implementation
    ;;
  *)
    log_error "Unknown subcommand: ${1:-}"
    exit 1
    ;;
esac
```

Profile-level services at `profiles/<name>/services/<category>/` shadow global services with the same name — useful for per-profile customizations.

---

## Planned Weapons

The weapons system (task manipulation tools) has a roadmap of planned additions:

| Weapon | Status | What It Will Do |
|--------|--------|----------------|
| Gun | Active | Bulk task series generator (wraps taskgun) |
| Sword | Active | Task splitting into sequential subtasks |
| Next | Active | CFS-inspired next-task recommendation |
| Schedule | Active | Auto-scheduler (wraps taskcheck) |
| Bat | Planned | — |
| Fire | Planned | — |
| Slingshot | Planned | — |

---

## Contributing

Workwarrior is open source. The service architecture is designed so new services can be added without touching core files. Extensions, weapons, and new service categories are all welcome via the ww-standard GitHub repository.

<p style="margin-top:28px">
  <a href="https://github.com/babbworks/ww-standard" class="btn btn-primary">GitHub: ww-standard →</a>
</p>
