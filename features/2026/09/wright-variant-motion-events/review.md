# Review: wright-variant-motion-events

Verdict: approve

## Acceptance checklist results

- **PASS — distinct per-state motion.** Every public state (`working`, `waiting`, `thinking`, `sleeping`, `sending`, `receiving`) gets a distinct, deterministic, architecture-consistent motif in `src/variants/wright.js` (`animateWright` per-state branches), while `done`/`error` remain `flashWright`-driven (unchanged). Evidence: `test/wright.test.js` "wright motion event frames are deterministic across every probed state", "wright motion paints a distinct frame sequence per state", "wright motion settles back to rest by identity for every animated state".
- **PASS — static identity frozen.** `deriveWright` and `WRIGHT_SPEC_VERSION` (`wright-geometry-v1`) are untouched in the diff; `git show origin/main:test/fixtures/golden-wright-v1.json` vs the regenerated fixture's `static` section are byte-identical (verified with the 3.2 node check). The lock-seed geometry values are asserted unchanged in "wright motion traits derive deterministically…".
- **PASS — hash-derived motion traits.** `wrightMotionTraits` reads only disjoint bit-ranges 42–52 of `params.hash` (`sweepDir`, `panelPhase`, `illumSpeed`) in `prepareWright`; no new PRNG draws, no `WRIGHT_DRAW_ORDER` or other-variant change. Evidence: diff touches only `wright.js`/`wright.test.js`/`golden-wright-v1.json`.
- **PASS — settle to rest.** `animateWright` in `settling` eases all three fields to `rest` and returns `ctx.rest` by identity within the epsilon. Evidence: "wright motion animates deterministically and settles back to rest by identity" and the per-state settle loop test.
- **PASS — animated-frame invariants.** ViewBox bounds, ≤10% red painted area, ≥3:1 contrast on emitted colors, and neutral reduced-motion/static rendering are asserted across animated frames. Evidence: "wright motion keeps every animated frame inside the viewBox…", "wright illumination keeps structural contrast >= 3:1…", "wright rest pose is neutral…".
- **PASS — golden scope.** Only `golden-wright-v1.json` changed among fixtures (168/168 `mounted` hashes); `golden-v1.json`, `golden-ncube-v1.json`, `golden-orbit-v1.json` are byte-identical to `origin/main`, and the Wright `static` section is unchanged.
- **PASS — visual verification.** The demo at `local:3108` shows the Wright variant across all states; 9 Builder screenshots plus my 4 independent re-drive screenshots (`review-idle`, `review-working`, `review-thinking`, `review-done`) confirm a restrained, legible, architecture-consistent composition and an unchanged idle portrait.
- **PASS — file boundary and gates.** `git diff --name-only origin/main...HEAD` lists exactly `src/variants/wright.js`, `test/wright.test.js`, `test/fixtures/golden-wright-v1.json`. `npm run verify` passes: 188 tests, plus `contract`, `exports`, `types`, `pack`, `goldens` all ✓. No lint command exists (recorded in plan `## Research`).

## Plan vs implementation

The implementation follows the plan's approach exactly: the recommended `{ illuminate, panelPulse, settle }` pose shape, the per-state motif mapping, disjoint-hash motion traits (bits 42–52, above the `phase`/`paletteFamily` ranges), bounded illumination lightening re-checked against `WRIGHT_CONTRAST_MIN`, per-module panel-pulse stroke modulation, and a structural settle offset on the horizontal planes. No undocumented changes; `flashWright`, `deriveWright`, `paintWright`'s layer order, and the public variant contract are preserved.

The two findings from the prior review are resolved: `sending`/`receiving` now ease `settle` to rest instead of hard-setting it, and `poseWright(params, 'working')` now seeds the mount pose from `params.phase` so it equals the first `animateWright` frame at `t = 0` (no mount jump, `params` used).

## Roadmap audit

All 11 steps are ticked and each tick checks out against the codebase:
- 1.1 pose/animate replacement, 1.2 hash traits, 2.1 paint interpretation, 2.2 invariant tests, 3.1 determinism/distinctness/settle tests, 3.2 golden regeneration (static byte-identical), 4.1 visual evidence (9 linked `evidence/` screenshots), 5.1 final gate with clean file boundary — all verified.
- 6.1 settle-easing fix, 6.2 working-pose seeding fix, 6.3 golden regeneration + full gate — each implemented, verified, and committed.
- Step 4.1 is an ordinary browser-driven step (not `(manual)`), and its linked evidence files exist.

No falsely ticked boxes, no missing-work steps, no repairs required.

## Findings

None above minor severity. The two prior minor findings (`sending`/`receiving` settle pop; unused `params` in `poseWright`) are fixed and verified by the 6.1–6.3 steps and the full gate.

## Follow-ups

- When `feature/wright-variant-legibility-tuning` (PR #22) merges, this branch must integrate `origin/main` before `/agento ship` — both touch `src/variants/wright.js` and `test/wright.test.js`.
