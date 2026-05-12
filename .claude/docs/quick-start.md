# Indie Game Development — Quick Start Guide

## What Is This?

A simplified Claude Code agent architecture for indie game development.
13 specialized agents, 21 skills, 4 phases — designed for solo devs and
small teams who need to prototype fast and iterate quickly.

## The 4 Phases

1. **Concept** — Define your game idea, choose engine, set pillars and visual style
2. **Prototype** — Build, test, validate the core mechanic (the most important phase)
3. **Production** — Implement features, code review, iterate
4. **Polish & Release** — QA, bug fixes, polish, ship

## How to Start

### First Time

Run `/start` — it will detect your project state and guide you to the right place.

### Have an idea?

Run `/brainstorm` to explore and document your game concept.

### Ready to build?

Run `/prototype` to validate your core mechanic with throwaway code.
**Prototype early and often.** Don't build systems before proving they're fun.

### Prototype validated?

Run `/gate-check` to confirm you're ready for production, then:
1. `/design-system` for each major system
2. `/dev-story` to implement
3. `/code-review` before merging

### Have a playable demo?

Run `/gate-check` to enter Polish & Release phase, then:
1. `/qa-plan` to create test plan
2. `/smoke-check` to verify nothing is broken
3. `/playtest-report` to analyze feedback
4. `/changelog` to generate release notes

## Available Agents

| I need to... | Use this agent |
|-------------|---------------|
| Design a mechanic | `game-designer` |
| Write gameplay code | `gameplay-programmer` |
| Create a shader | `technical-artist` |
| Design a level | `level-designer` |
| Review code | `lead-programmer` |
| Write dialogue | `writer` |
| Build UI | `ui-programmer` |
| Define visual style | `art-director` |
| Godot-specific help | `godot-specialist` |
| Architecture decision | `technical-director` |
| Test my game | `qa-tester` (Phase 4 only) |

## Key Principles

- **Prototype first** — validate fun before building systems
- **QA is deferred** — don't test until you have a playable demo
- **You're the director** — agents suggest, you decide
- **Flat structure** — no hierarchy, just pick the right agent for the job
