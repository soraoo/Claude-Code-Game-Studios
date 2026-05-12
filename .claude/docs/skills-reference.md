# Available Skills (Slash Commands)

21 slash commands organized by phase. Type `/` in Claude Code to access any of them.

## Phase 1 — Concept

| Command | Purpose |
|---------|---------|
| `/start` | First-time onboarding — asks where you are, then guides you to the right workflow |
| `/help` | Context-aware "what do I do next?" — reads current stage and surfaces the required next step |
| `/brainstorm` | Guided ideation using professional studio methods (MDA, SDT, Bartle, verb-first) |
| `/setup-engine` | Configure engine + version, detect knowledge gaps, populate version-aware reference docs |
| `/project-stage-detect` | Full project audit — detect phase, identify gaps, recommend next steps |
| `/art-bible` | Define visual style, color palette, and asset standards for your game |

## Phase 2 — Prototype

| Command | Purpose |
|---------|---------|
| `/prototype` | Rapid prototyping workflow — validate core mechanic with throwaway code |
| `/quick-design` | Lightweight design spec for small changes — tuning, tweaks, minor additions |
| `/gate-check` | Validate readiness to advance between phases |

## Phase 3 — Production

| Command | Purpose |
|---------|---------|
| `/design-system` | Guided, section-by-section GDD authoring for a single game system |
| `/create-epics` | Break GDDs into implementation epics |
| `/create-stories` | Break an epic into implementable story files |
| `/sprint-plan` | Plan the current sprint — lightweight, single-file |
| `/dev-story` | Read a story file and implement it — the core build loop |
| `/code-review` | Architectural and quality code review |
| `/story-done` | Verify story meets acceptance criteria |
| `/bug-report` | Create a structured bug report |

## Phase 4 — Polish & Release

| Command | Purpose |
|---------|---------|
| `/qa-plan` | Generate QA test plan from GDDs and stories |
| `/smoke-check` | Critical path verification — quick check that nothing is broken |
| `/playtest-report` | Collect and analyze playtest feedback |
| `/changelog` | Auto-generate changelog from git history |

## Usage Notes

- **Phase 1-3**: QA skills are intentionally unavailable — defer testing until you have a playable demo
- **Phase 4**: QA activates —  become available
- `/gate-check` is your decision tool — use it at phase boundaries
- `/prototype` is the most important Phase 2 skill — validate before you build
