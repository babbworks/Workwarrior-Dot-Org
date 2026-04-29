---
layout: post
title: "Weapons: Gun, Sword, and the Philosophy of Task Manipulation"
date: 2026-04-23
categories: [technical]
---

Weapons are tools that manipulate profile data in special ways — creating, slicing, and packaging tasks beyond what direct TaskWarrior commands produce. They're called weapons because they're more powerful than a standard command and require more care.

The current set: Gun, Sword, Next, Schedule. The roadmap: Bat, Fire, Slingshot (purposes TBD). They appear in the browser sidebar as a weapons bar.

## Sword

Sword is native to ww — no external binary. It takes a single task and splits it into N sequential subtasks with dependency chains.

```bash
ww sword 5 -p 3                    # Split task 5 into 3 parts
ww sword 5 -p 4 --interval 2d     # 2-day intervals between parts
ww sword 12 -p 2 --prefix "Phase" # Custom prefix
```

Each subtask gets:
- Description: "Part N of: {original description}"
- The parent task's project and tags
- A due date offset by N × interval from now
- A dependency on the previous subtask

The dependency chain is the key. TaskWarrior's dependency model prevents a task from being completed if it has uncompleted dependencies. Sword creates a strictly sequential chain — you can't mark "Part 3 of: Ship API" done until "Part 2 of: Ship API" is done. This is the right behavior for a task that genuinely has ordered phases.

The parent task gets archived (done) after the split, since it's now represented by the subtask chain. The context — project, tags, urgency tuning — carries forward.

## Gun

Gun wraps [taskgun](https://github.com/hamzamohdzubair/taskgun), a Rust binary for bulk task series generation. Give it a series definition and it creates multiple related tasks with deadline spacing.

```bash
ww gun <args>
```

The wrapping is thin: Gun respects the active profile (reads TASKRC/TASKDATA from env), passes arguments through to taskgun, and handles the TaskWarrior output. It appears as `ww gun` rather than requiring the user to know taskgun's command syntax.

## Next

Next wraps the `next` binary — a CFS (Completely Fair Scheduler) inspired algorithm that recommends the optimal next task based on urgency scores, deadline proximity, and current context.

```bash
ww next
```

One command, one recommendation. The recommendation factors in TaskWarrior's urgency score plus context signals. When you don't know what to work on, this is the answer.

## Schedule

Schedule wraps taskcheck, an auto-scheduler that assigns time blocks to tasks across your calendar.

```bash
ww schedule
```

## Design Principles

All weapons follow the same rules:
- Read TASKRC/TASKDATA from environment (profile isolation is respected)
- Work through `POST /cmd` in the browser UI — weapons are available from the weapons bar
- Never modify data outside the active profile's scope
- Respond to `--help`

The weapons bar in the browser UI shows icons for each weapon. Clicking opens a panel that exposes the weapon's parameters as a form, so you don't have to remember the CLI syntax.

The planned weapons — Bat, Fire, Slingshot — don't have specified purposes yet. The naming scheme is intentional: tools that cut, split, and shape task data. What they shape is still being figured out.
