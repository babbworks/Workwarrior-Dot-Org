---
layout: post
title: "UDAs: TaskWarrior's Extensible Data Model and What We Did With It"
date: 2026-04-29
categories: [technical]
---

TaskWarrior's User Defined Attributes system is one of the most underused features in the tool. You can add arbitrary typed fields to every task — strings, numbers, dates, durations, booleans. Workwarrior treats UDAs as a first-class concept and exposes a full management layer on top of them.

The result: profiles can carry 100+ UDAs covering project metadata, financial fields, compliance tracking, people, equipment, and more — and the browser UI renders all of them in the task inline editor.

## What UDAs Are

A UDA is a field definition in `.taskrc`. You declare the field name, type, label, and optional default value. Once declared, every task can carry that field.

```
uda.client.type=string
uda.client.label=Client
uda.estimate.type=duration
uda.estimate.label=Estimated Duration
uda.billable.type=numeric
uda.billable.label=Billable Hours
```

These are now first-class fields on every task in the profile. `task add "Design review" client:acme estimate:2h billable:0` creates a task with those custom fields populated.

## The UDA Registry

Workwarrior maintains a UDA registry per profile with source classification:

- `[github]` — injected by Bugwarrior from issue tracking services
- `[extension]` — added by TaskWarrior extensions or hooks
- `[custom]` — defined by the user via `ww profile uda add`

```bash
ww profile uda list            # All UDAs with source badges
ww profile uda add goals       # Interactive creation wizard
ww profile uda remove <name>   # Remove with safety warnings
ww profile uda group work      # Group UDAs for batch operations
ww profile uda perm goals nosync  # Set sync permissions per-UDA
```

The source badges matter for sync. A `[github]` UDA is owned by Bugwarrior — you probably don't want to push changes to that field back to GitHub. A `[custom]` UDA is yours — sync permissions are yours to set.

## Sync Permissions

`ww profile uda perm <name> nosync` marks a UDA as excluded from the github-sync engine. This prevents custom internal fields from appearing in GitHub issue labels or comments during two-way sync.

This is the kind of detail that breaks integrations in the wild: you add a `priority-internal` UDA that maps to your team's internal priority scale, and suddenly it's showing up as a GitHub label because the sync engine doesn't know it's internal. The permission system prevents that.

## The Browser UI Integration

The browser UI's task inline editor renders all UDAs defined in the active profile. You don't configure this — it reads the profile's `.taskrc`, discovers all UDA definitions, and builds the edit form dynamically.

This means a profile with `client`, `estimate`, `billable`, `sprint`, and `component` UDAs gets a task editor with all five fields, correctly typed (text inputs for strings, numeric inputs for numbers, duration pickers for durations). The profile with no custom UDAs gets the standard task editor.

## Urgency Tuning

UDAs can contribute to TaskWarrior's urgency score. If `billable` hours is high, the task should float up in priority. If `sprint` is the current sprint number, active sprint tasks should be more urgent.

```bash
ww profile urgency             # Interactive urgency coefficient tuner
```

The tuner shows all available urgency coefficient inputs — due date weight, priority weight, age weight, tag weights, and custom UDA weights — and lets you adjust them with an interactive prompt. The result is written to the profile's `.taskrc`.

Per-profile urgency tuning is the right abstraction. Your work profile might weight `client` UDA heavily (client work is always more urgent). Your personal profile doesn't have a `client` UDA and weights due date proximity more.

## The Data Model Goes Deep

100+ UDAs covering: project metadata (phase, epic, component, sprint, story points), financial fields (estimate, actuals, billable, rate), compliance (owner, reviewer, approval date, compliance tag), people (assignee, stakeholder, reporter), and equipment (asset ID, location, maintenance date).

These aren't built-in. They're documented patterns in `docs/setups/` that users can apply to their profiles using `ww profile uda group <name>`. The templates are starting points, not requirements. The data model is yours.
