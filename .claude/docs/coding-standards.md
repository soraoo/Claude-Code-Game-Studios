# Coding Standards

## Prototype Phase (Phase 2)

- Hardcode values freely — optimize for speed, not polish
- Skip tests — validate by playing
- Placeholder assets are fine
- Code can be thrown away after validation

## Production Phase (Phase 3+)

- Gameplay values must be data-driven (external config), never hardcoded
- All public methods should be unit-testable (dependency injection over singletons)
- Use doc comments on public APIs

## Design Document Standards

- All design docs use Markdown
- GDD files should include these 4 core sections:
  1. **Overview** — what the system does
  2. **Player Experience** — how it feels
  3. **Rules & Formulas** — mechanics and math
  4. **Acceptance Criteria** — testable success conditions
- Add Dependencies, Tuning Knobs, and Edge Cases sections as needed

## Testing Standards

- **Tests are deferred to Phase 4** — write automated tests only after you have a playable demo
- In Production (Phase 3), test by playing — smoke check style verification
- In Polish (Phase 4), add automated tests for Logic and Integration stories
- Test naming: `[system]_[feature]_test.[ext]` for files; `test_[scenario]_[expected]` for functions
- Tests must be deterministic, isolated, and independent

## What NOT to Automate

- Visual fidelity (shader output, VFX appearance, animation curves)
- "Feel" qualities (input responsiveness, perceived weight, timing)
- Full gameplay sessions (covered by playtesting)
