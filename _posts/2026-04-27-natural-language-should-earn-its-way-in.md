---
layout: post
title: "Natural Language Should Earn Its Way In"
date: 2026-04-27
categories: [architecture, technical]
---

There's a temptation when building a CLI tool in 2024 to make AI the primary interface. Type anything, the model figures it out. No documentation needed. Natural language all the way down.

Workwarrior went the other direction. The heuristic engine is the primary interface. AI is optional, disabled by default, and explicitly opt-in. If you want to type natural language commands, you can — but they go through 627 regex rules first, and only reach an LLM if nothing matches.

This is a deliberate philosophy, not a cost-cutting measure.

## The Arguments for Heuristics First

**Determinism.** A heuristic rule either matches or it doesn't. The output for a given input is always the same. An LLM produces different outputs for the same input on different runs, different days, different model versions. For a system that writes to your task database and time tracking records, determinism matters.

**Latency.** A regex match takes microseconds. An LLM API call takes seconds, plus the network round trip. For someone running `ww browser` as a persistent dashboard, heuristic commands feel instant. LLM commands feel like waiting.

**Privacy.** The heuristic engine processes everything locally. A task description that goes to an LLM API leaves your machine. For professional or client work, that's a meaningful distinction.

**Reliability.** The heuristic engine works offline. It works when your LLM provider is down. It works when you're in a coffee shop with unreliable internet. It works always.

## The Self-Improvement Loop

The most interesting part of the heuristic/AI relationship is the feedback mechanism. Every CMD submission is logged as JSONL: input, route (heuristic or AI), output, success status. Running `ww compile-heuristics --digest` reads that log and converts successful AI translations into new heuristic rules.

This creates a loop: the AI handles the edge cases the heuristics can't, those edge cases become heuristic rules over time, the AI handles fewer things, the heuristics handle more. The AI dependency decreases through use.

It's a case where AI earns its place by making itself less necessary. The value isn't persistent dependence — it's seeding the rule set with patterns that cover real usage.

## What AI Is Good For

The cases where AI adds genuine value are the cases where the heuristic coverage is thin: unusual phrasings, compound commands with unusual structure, commands that combine semantics in ways the compiler didn't anticipate.

```
"Flag the server migration task as needing review before the deadline"
```

That's not a standard imperative, declarative, or shorthand phrasing. It's an unusual sentence that a human would understand immediately. The heuristic engine probably misses it. The LLM handles it.

After the `--digest` run, that phrasing becomes a rule. The next person who types something similar gets heuristic speed.

## The Configuration Model

```yaml
# config/ai.yaml
mode: off              # Default
# mode: local-only    # Ollama only — no data leaves machine
# mode: local+remote  # Local first, remote fallback
```

Three modes. Default is `off`. `local-only` is for people who have ollama running locally and want AI fallback without external API dependencies. `local+remote` for people who want the full fallback chain.

Per-profile overrides in `profiles/<name>/ai.yaml`. Work profile might have AI off (deterministic commands for professional tasks). Personal profile might have AI on (more relaxed about sending journal entries to an API).

The controls are available from `ww ctrl` and from the browser CTRL panel. No restart required. Toggle AI mode mid-session.

The point is: you decide. The heuristics work without AI. AI helps when you want it to. The system doesn't require a subscription.
