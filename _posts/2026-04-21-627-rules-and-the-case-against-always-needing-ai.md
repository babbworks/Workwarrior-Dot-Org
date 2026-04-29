---
layout: post
title: "627 Rules and the Case Against Always Needing AI"
date: 2026-04-21
categories: [technical, architecture]
---

The heuristic engine came from a simple question: how many natural language command phrasings can you handle with regex before you need a language model? The answer turned out to be: most of them.

627 compiled rules. 19 command domains. 6 phrasing variations per command. No network. No latency. No API key. You type "add a task to review the budget due friday" and the rule for imperative task creation with a date expression fires in microseconds.

## Why This Matters

If AI is required for basic commands, the system has an external dependency for routine operations. That's fine when internet is reliable and latency is acceptable. It's a problem when you're on a plane, when your LLM provider is down, when you don't want to send your task descriptions to a third party.

The heuristic engine makes the CMD service useful offline and without configuration. Install workwarrior, run `ww browser`, type commands in plain English — it works before you've set up a single LLM provider.

## What the Engine Covers

Six phrasing variations per command, with different confidence scores:

| Variation | Confidence | Example |
|-----------|-----------|---------|
| Passthrough | 1.0 | `task add review budget` |
| Imperative | 0.95 | `add a task to review the budget` |
| Declarative | 0.90 | `I need a task for reviewing the budget` |
| Interrogative | 0.90 | `can you create a task to review the budget` |
| Shorthand | 0.90 | `task: review budget due friday` |
| Verbose | 0.85 | `I would like to add a new task for reviewing the budget` |

Date expressions are normalized: "tomorrow", "next week", "friday", "end of month", "in 3 days" all map to the correct TaskWarrior date format.

Compound commands are split on conjunctions and matched independently:

```
"finish task 5 and stop tracking"
  → task 5 done
  → timew stop

"add task review and start tracking time"
  → task add review
  → timew start review
```

If any segment fails to match above the confidence threshold (0.8), the entire compound goes to AI. This is the right behavior — a half-translated compound command is worse than no translation.

## The Compiler

Rules aren't written by hand. `ww compile-heuristics` scans the codebase — `bin/ww` case branches, `config/shortcuts.yaml`, `config/command-syntax.yaml` — generates regex patterns, validates them against a synthetic corpus, resolves conflicts, fills coverage gaps, and writes the output.

The `--verbose` flag shows every rule with test results. The `--digest` flag reads `services/cmd/cmd.log` — every CMD submission is logged as JSONL — and converts successful AI translations into new heuristic rules.

That last part is the self-improvement loop. Every time the AI handles a command the heuristics couldn't, that's a potential new rule. Run `--digest` regularly and the AI dependency shrinks over time.

## The Fallback Chain

```
Input
  → Compound split (if "and"/"then"/"also"/"plus")
  → Each segment: heuristic match at confidence > 0.8
  → No match? → AI fallback if configured
  → AI not configured? → Clear error message
```

The error message on a heuristic miss is explicit: "No matching rule. Try the full ww command or enable AI mode." That's better than silently failing or silently sending data to an API.

AI is an option, not a requirement. Most commands go through heuristics. The engine gets better over time. The AI dependency decreases.
