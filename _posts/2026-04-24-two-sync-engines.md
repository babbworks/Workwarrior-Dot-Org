---
layout: post
title: "Two Sync Engines, Two Different Jobs"
date: 2026-04-24
categories: [technical, internals]
---

Workwarrior ships two GitHub sync engines. They're complementary, not redundant, and understanding the distinction matters for how you configure each one.

**Bugwarrior** — one-way pull from 20+ issue tracking services into TaskWarrior.  
**ww github-sync** — two-way bidirectional sync between individual tasks and GitHub issues.

Use both. They handle different parts of the problem.

## Bugwarrior: Pull Everything

Bugwarrior is a pull tool. Configure it with API credentials for your services — GitHub, GitLab, Jira, Trello, Bitbucket, and 17 more — and `i pull` brings all your open issues into TaskWarrior as tasks.

```bash
ww issues custom          # Configure services interactively
i pull                    # Pull from all configured services
i status                  # Show what was pulled
```

The tasks it creates carry injected UDAs: `githubissue`, `githuburl`, `githubrepo`, `githubauthor`. These are tagged `[github]` in the UDA source registry — you can see them with `ww profile uda list`.

Bugwarrior is strictly one-way. It reads from the service and writes to TaskWarrior. Changes you make to a bugwarrior-created task in TaskWarrior don't flow back to GitHub. That's where github-sync comes in.

## ww github-sync: Two-Way on Selected Tasks

The two-way sync engine links individual TaskWarrior tasks to individual GitHub issues. Once linked, changes flow in both directions: update the task locally, it updates the issue. Comment on the issue, the annotation appears on the task.

```bash
ww issues enable <task> <issue#> <org/repo>   # Link
ww issues sync                                 # Two-way sync all linked
ww issues push                                 # Push local → GitHub only
```

**What syncs bidirectionally:**
- Description ↔ Title
- Status ↔ State (pending/started → OPEN, completed/deleted → CLOSED)
- Priority ↔ Labels (H/M/L → priority:high/medium/low)
- Tags ↔ Labels
- Annotations ↔ Comments (with prefixes to prevent sync loops)

**What pulls only (GitHub → TaskWarrior):**
- Issue number, URL, repo, author, created date, closed date

Conflict resolution is last-write-wins with a configurable conflict window. If both sides changed within the window, the sync reports the conflict rather than silently overwriting.

## The Fragility Register

The sync engine is the highest-risk part of the codebase. Ten files in `lib/`. Classified HIGH FRAGILITY.

Getting it wrong means data loss in two places simultaneously: a task annotation deleted that was actually a comment on a critical issue, a status change propagated incorrectly that closes an issue that shouldn't be closed. These failures are hard to detect and harder to reverse.

The development protocol for these files: extended risk brief before any change, explicit write scope, Verifier sign-off before merge, integration tests required. This isn't excessive — it's proportionate to the failure modes.

## Using Them Together

The intended workflow:

1. Configure Bugwarrior to pull from all your services. Run `i pull` to get all your issues as tasks.
2. Identify the tasks you're actively working and want to keep in sync. Run `ww issues enable` on those.
3. Work normally. `i pull` refreshes the full issue list. `ww issues sync` keeps your linked tasks bidirectionally current.

The Bugwarrior-created tasks and the github-sync-linked tasks can coexist in the same TaskWarrior profile. Bugwarrior stamps its tasks with the `[github]` source badge on the UDA. github-sync manages its own state file. They don't interfere with each other.
