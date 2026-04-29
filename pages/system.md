---
layout: page
title: System
eyebrow: Control Plane
subtitle: The internal coordination layer that builds Workwarrior — and ships as optional MCP tooling for users.
permalink: /system
---

`ww/system` is the project's control plane. It's not shipped to users — it's the directory where all planning, contracts, fragility analysis, gate decisions, and agent coordination live during development.

It's also the source of the agent model that ships as `ww mcp install` — the same four-role system that builds Workwarrior can be deployed against any TaskWarrior-backed project via the MCP server.

---

## Agent Model

Four always-active roles. Two conditional roles deployed by the Orchestrator as needed.

| Role | Constraint |
|------|-----------|
| **Orchestrator** | Owns backlog, contracts, merge decisions. Never writes production code. Never self-approves. |
| **Builder** | Works within explicit write scope. Produces risk brief before touching any file. |
| **Verifier** | Adversarial test execution. Produces signed checklist. Never implements. |
| **Docs** | Updates CLAUDE.md files, docs/, help strings. Runs after merge. |
| **Explorer** | Read-only. Produces risk briefs for audit phases and cross-cutting analysis. |
| **Simplifier** | Large diffs or high-risk edits — embedded in Verifier's checklist. |

Handoff sequence: **Orchestrator** (contract) → **Builder** (implement) → **Verifier** (validate) → **Docs** (close).

---

## Fragility Register

Not every part of the codebase is the same risk. The fragility register classifies the highest-risk subsystems:

| Subsystem | Files | Classification | Why |
|-----------|-------|---------------|-----|
| GitHub sync engine | `lib/sync-*.sh`, `lib/github-*.sh` (10 files) | HIGH | Bidirectional state, conflict resolution, auth |
| Shell integration | `lib/shell-integration.sh` | HIGH | Injected into every shell session |
| CLI dispatcher | `bin/ww` | HIGH | All commands route through here |
| Profile manager | `lib/profile-manager.sh` | MEDIUM | Writes profile directories |
| Browser server | `services/browser/` | MEDIUM | External-facing, SSE state |

Changes to HIGH classification files require an extended risk brief before implementation begins.

---

## Gate Model

Every implementation task passes four gates before close:

- **Gate A** — Spec complete. Contract document written, scope agreed.
- **Gate B** — Implementation ready. Risk brief produced, write scope explicit.
- **Gate C** — Tests pass. BATS suites and/or pytest pass. Verifier sign-off.
- **Gate D** — Docs updated. CLAUDE.md, user docs, help strings reflect the change.

No skipping gates. No self-approval across roles.

---

## Shell Standards

Every script in the project follows these rules — violations are Gate B failures:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

- Bash, not sh. `#!/usr/bin/env bash` on every script.
- `set -euo pipefail` is the second line. No exceptions.
- Logging via `lib/logging.sh` — never raw `echo` in lib or services.
- Absolute paths always. `$WORKWARRIOR_BASE/...`, never relative.
- Quote all variable expansions. `"$var"`, `"${array[@]}"`.
- Functions in snake_case. All local variables declared with `local`.
- Exit codes: 0 = success, 1 = user error, 2 = system/internal error.

---

## Environment Variables

Set automatically when a profile is active:

| Variable | Value |
|----------|-------|
| `WARRIOR_PROFILE` | Active profile name (e.g. `work`) |
| `WORKWARRIOR_BASE` | Profile base directory |
| `TASKRC` | TaskWarrior config path |
| `TASKDATA` | TaskWarrior data directory |
| `TIMEWARRIORDB` | TimeWarrior database path |

Nothing in the system hardcodes paths. Everything resolves through these variables.

---

## MCP Server

The agent model ships as `ww mcp install` — an MCP (Model Context Protocol) server that exposes TaskWarrior data to AI agents. The same roles and gate model that builds Workwarrior can be deployed against any project that runs TaskWarrior.

```bash
ww mcp install     # Install the MCP server
ww mcp status      # Show running state
```

<p style="margin-top:28px">
  <a href="https://github.com/babbworks/ww-standard" class="btn btn-ghost">ww-standard on GitHub →</a>
</p>
