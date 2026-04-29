---
layout: post
title: "The Control Plane: What ww/system Actually Is"
date: 2026-04-25
categories: [internals, architecture]
---

`ww/system` is the directory that doesn't ship. It's the project's internal coordination layer — planning documents, task boards, fragility registers, agent role definitions, gate contracts. Users never see it. But it's the reason the codebase is coherent.

Describing it as "documentation" is underselling it. It's an operating model for a codebase that's complex enough to need one.

## Why a Control Plane at All

A bash codebase with 24 library files, a bidirectional sync engine, a Python web server, and a compiler that generates regex rules has failure modes that aren't obvious from the code. The sync engine can corrupt data in two places at once. The shell integration injects functions into every shell session — a bug there affects every command a user runs. The CLI dispatcher routes everything — a routing error silently misdirects commands.

Without a way to classify risk and enforce process around high-risk changes, the natural tendency is to edit files and see what breaks. In a project with irreversible failure modes, that's not acceptable.

The fragility register classifies every major subsystem. HIGH fragility means changes require an extended risk brief, explicit write scope, and adversarial Verifier sign-off before merge. MEDIUM means standard process. LOW means standard process with lighter documentation.

## The Four Roles

The agent model defines four always-active roles:

**Orchestrator** — owns the task board, writes contracts, makes merge decisions. Never writes production code. The constraint is critical: if the Orchestrator writes code, it loses the perspective needed to evaluate that code.

**Builder** — works within explicit write scope defined by the Orchestrator. Produces a risk brief before touching any file in the write scope. The risk brief is required — it forces the Builder to think through failure modes before editing.

**Verifier** — adversarial test execution. Runs the test suite, looks for regressions, produces a signed checklist. Never implements. The Verifier's job is to find problems, not fix them.

**Docs** — updates CLAUDE.md files, user documentation, and help strings after merge. Runs last, after the implementation is confirmed correct. Documentation written before code is verified tends to describe intended behavior rather than actual behavior.

No agent self-approves. The Orchestrator's task card is a contract — the Builder implements against it, the Verifier validates against it, the Docs agent updates to reflect it.

## The Gate Model

Gate A: Spec complete. Contract written, scope agreed.  
Gate B: Implementation ready. Risk brief produced.  
Gate C: Tests pass. Verifier sign-off.  
Gate D: Docs updated. Task closed.

No skipping gates. A change that skips Gate C and goes straight to Gate D is a change that's not verified. A change that skips Gate B is a change without a risk brief — meaning nobody thought through what could go wrong before touching the code.

The gate model isn't bureaucracy for its own sake. It's the minimum process needed to keep irreversible failure modes from reaching production.

## It Ships as MCP Tooling

The same agent model — Orchestrator, Builder, Verifier, Docs — is available to users via `ww mcp install`. The MCP server exposes TaskWarrior data to AI agents. The control plane architecture you'd use to build a project with these roles is the same one that builds Workwarrior.

This means the system is self-demonstrating. The development process produces a tool that implements the development process. Users who want to bring the same coordination model to their own projects can deploy it directly.

It's a strange loop. The cleaner we keep `ww/system`, the better the tool that comes out of it.
