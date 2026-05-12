# Design Directory

When authoring or editing files in this directory, follow these standards.

## GDD Files (`design/gdd/`)

Every GDD should include these **4 core sections**:
1. **Overview** — what the system does, one paragraph
2. **Player Experience** — how it feels to interact with this system
3. **Rules & Formulas** — mechanics and any math, clearly defined
4. **Acceptance Criteria** — testable success conditions

Additional sections (add as needed):
- **Dependencies** — other systems this depends on
- **Tuning Knobs** — configurable values
- **Edge Cases** — unusual situations handled

**File naming:** `[system-slug].md` (e.g. `movement-system.md`, `combat-system.md`)

**Systems index:** `design/gdd/systems-index.md` — update when adding a new GDD.

## Quick Specs (`design/quick-specs/`)

Lightweight specs for tuning changes, minor mechanics, or balance adjustments.
Use `/quick-design` to author.
