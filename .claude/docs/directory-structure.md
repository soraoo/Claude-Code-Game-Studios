# Directory Structure

```
/
├── CLAUDE.md                    # Master configuration
├── .claude/                     # Agent definitions, skills, hooks, rules, docs
├── src/                         # Game source code
├── assets/                      # Game assets (art, audio, vfx, shaders, data)
├── design/                      # Game design documents (gdd, quick-specs)
├── docs/                        # Technical documentation
│   └── engine-reference/        # Curated engine API snapshots (version-pinned)
├── tests/                       # Test suites (deferred to Phase 4)
├── prototypes/                  # Throwaway prototypes (isolated from src/)
└── production/                  # Production management
    ├── session-state/           # Ephemeral session state (active.md)
    └── session-logs/            # Session audit trail
```
