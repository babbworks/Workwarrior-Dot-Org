---
layout: post
title: "Why We Wrapped Instead of Built"
date: 2026-04-28
categories: [open-source, architecture]
---

TaskWarrior is almost 20 years old. Hledger is over 15. JRNL has been actively maintained for over a decade. These aren't projects that emerged from a hackathon and might disappear next year. They have communities, changelogs, issue trackers with years of history, and maintainers who care.

Building replacements for any of these tools would mean rebuilding years of accumulated design decisions and then maintaining those decisions indefinitely. Wrapping them means the tools stay excellent, the communities keep them current, and Workwarrior builds only the layer that didn't exist.

## What That Layer Is

The thing that didn't exist was the connection layer. TaskWarrior doesn't know about TimeWarrior. Neither of them knows about JRNL. None of them have a concept of a "profile" — an isolated workspace that all four tools share. None of them have a unified command surface. None of them ship a local web UI.

The connection layer — profile isolation, unified commands, natural language translation, browser dashboard, GitHub sync, weapons — is thin relative to the underlying tools. The underlying tools do the heavy lifting. Workwarrior adds the wiring.

This is the right decomposition. The alternative, a monolithic productivity system that handles tasks, time, journaling, and accounting in a single codebase, exists (several of them). They're all trapped in a maintenance cycle that gets heavier as the feature surface grows. Workwarrior's feature surface is bounded by definition: it's the surface between tools, not the tools themselves.

## The Acknowledgement Model

There's an `tools/acknowledgements.md` in the repo. It lists every tool, the maintainers, the license, the upstream repo. This isn't a legal requirement — it's a design value. Workwarrior works because these projects exist. The people who built them deserve credit by name.

The extension registry (`ww extensions`) takes the same approach. TaskWarrior has a rich ecosystem of community hooks and extensions. Workwarrior indexes them, makes them searchable, and makes them installable from one command. The registry celebrates the ecosystem rather than hiding it.

When a user discovers that `ww gun` is powered by a Rust binary called taskgun written by hamzamohdzubair, and follows the link to the GitHub repo, that's a win. The attribution is explicit. The dependency is visible. The user can contribute to taskgun directly if they want to improve the gun weapon.

## When Wrapping Doesn't Work

There are cases where wrapping breaks down. The wrapper needs to stay thin — when it gets thick, it's usually a sign that a tool needed a patch rather than wrapping.

The two-way GitHub sync engine is the closest case. There's no off-the-shelf bidirectional TaskWarrior ↔ GitHub sync solution that fits the profile model. That layer had to be built. Ten library files, a conflict resolution algorithm, field mapping, annotation↔comment sync. It's the highest-fragility part of the codebase because it's the part that was actually built from scratch rather than wrapped.

Everything else wraps. The sync engine builds. The difference in fragility is instructive: the built parts are where the complexity lives.

## What Gets Contributed Back

When working at the interface layer reveals a limitation in an underlying tool, the right move is to contribute upstream rather than work around it. The profile model revealed several edge cases in how TaskWarrior handles hook execution with non-standard TASKRC paths. Those were reported upstream.

The wrapper relationship is bidirectional. Workwarrior depends on these tools and owes them the bugs found while using them in unusual ways. The upstream communities get better test coverage. The tools get better. Workwarrior benefits directly.

This is what "open source as it gets" means in practice. Not just a public repo and an MIT license. Using the ecosystem, attributing it, finding and reporting bugs, contributing back where possible, being explicit about what you depend on. The whole system works better when the connection layer takes those obligations seriously.
