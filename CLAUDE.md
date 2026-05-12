# Claude Code Game Studios — Indie Template

Solo/small-team game development, simplified. 13 agents, 21 skills, 4 phases.

## Technology Stack

- **Engine**: Godot 4.6
- **Language**: GDScript
- **Version Control**: Git
- **Build System**: Godot CLI headless builds
- **Asset Pipeline**: Godot import system

## Project Structure

@.claude/docs/directory-structure.md

## Development Phases

1. **Concept** — Define game idea, choose engine, set pillars, visual style
2. **Prototype** — Build, test, validate core mechanic (fast iteration)
3. **Production** — Implement features, code review, iterate
4. **Polish & Release** — QA, bug fixes, ship

## Workflow

**Prototype-first. Iterate fast. Defer QA until you have a playable game.**

- Run `/start` for first-time setup
- Run `/help` at any time for context-aware guidance
- Run `/prototype` early and often — validate before building
- QA only activates in Phase 4 (Polish & Release)

## Engine Version Reference

@docs/engine-reference/godot/VERSION.md

## Core Skills

/start, /help, /brainstorm, /setup-engine, /project-stage-detect,
/quick-design, /art-bible, /design-system, /prototype, /gate-check,
/dev-story, /code-review, /story-done, /create-epics, /create-stories,
/sprint-plan, /bug-report, /qa-plan, /smoke-check, /playtest-report, /changelog

## Available Agents

@.claude/docs/agent-roster.md

## Coding Standards

@.claude/docs/coding-standards.md

## Context Management

@.claude/docs/context-management.md

## Collaboration Protocol

**User-driven collaboration, not autonomous execution.**
Every task follows: **Question -> Options -> Decision -> Draft -> Approval**

- Agents MUST ask before using Write/Edit tools
- Agents MUST show drafts or summaries before requesting approval
- No commits without user instruction
