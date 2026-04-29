---
layout: post
title: "Five Tools, One Profile, Zero Leakage"
date: 2026-04-20
categories: [architecture, technical]
---

The design decision that made everything else possible was profile isolation. It sounds simple and it is — one directory per work context, environment variables to activate it, every tool reads those variables automatically. But getting that right was the thing that unlocked the rest of the system.

The problem we were solving was context leakage. Work tasks showing up in personal task lists. Client time tracking mixed with personal project hours. A journal search for one project surfacing entries from another. These aren't hypothetical problems. They happen immediately the moment you use the same tools for more than one work context.

## The Directory Model

A profile is a directory:

```
profiles/work/
  .taskrc              TaskWarrior config
  .task/               Task database
  .timewarrior/        Time tracking database
  journals/            Journal files
  ledgers/             Hledger ledger files
  jrnl.yaml            Journal → file mapping
  ledgers.yaml         Ledger → file mapping
```

Activating a profile means exporting five environment variables:

```bash
export WARRIOR_PROFILE=work
export WORKWARRIOR_BASE=~/ww/profiles/work
export TASKRC=~/ww/profiles/work/.taskrc
export TASKDATA=~/ww/profiles/work/.task
export TIMEWARRIORDB=~/ww/profiles/work/.timewarrior
```

That's it. TaskWarrior reads `TASKRC` and `TASKDATA`. TimeWarrior reads `TIMEWARRIORDB`. JRNL reads its config from the path set in `jrnl.yaml`. Hledger reads from whatever file `l` points at. None of them know Workwarrior exists.

## What This Enables

Backup is `tar`. Restore is `untar`. No database to dump. No sync to configure. No cloud dependency. A profile is files.

Multiple profiles can exist simultaneously. Switching between them is instant — just env var changes. If you have a `work` profile and a `personal` profile and a `client-a` profile, `p-work` puts you in work context in milliseconds.

Multiple named resources per profile give you further isolation within a context. One profile can have journals called `strategy`, `engineering`, and `standup`, each pointing at a different file. Same for ledgers. The infrastructure anticipates multiple task lists and time tracking instances per profile, too.

Groups let you operate across profiles: `ww group backup all` backs up every profile in one command.

## What It Costs

The activation step. You have to remember to run `p-work` before working. If you don't, you're in the last active profile (or no profile if you haven't set a default).

This is a feature, not a bug. Explicit context switching means you always know which context your data is going into. The accidental cross-context write is impossible by design.

The `p-<name>` aliases are shell functions injected at init by `ww-init.sh`. They take under a millisecond. There's no meaningful latency in switching.

## The Key Principle

Nothing in the system hardcodes paths. Everything resolves through the five environment variables. Every service, every lib function, every shell wrapper reads `$WORKWARRIOR_BASE` or the tool-specific variables. A service that hardcodes a path is a bug.

This is also why profile removal works cleanly — `ww remove work` scrubs the profile from config, state, aliases, and templates. There are no hardcoded references to hunt down.
