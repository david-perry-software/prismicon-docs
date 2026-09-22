# Review: wright-variant-legibility-tuning

Verdict: approve

## Acceptance checklist results

All seven checklist items pass; each was re-verified independently on 2026-09-21.

- [x] `prepareWright` exposes a `small` flag at `size < 28` and `buildWright` reduces grid modules to at most `2×2` and decorations to at most `2` below that threshold, while `deriveWright`, `WRIGHT_DRAW_ORDER`, and `WRIGHT_SPEC_VERSION` stay frozen — **pass**. `src/variants/wright.js` adds `WRIGHT_SMALL_SIZE`/`WRIGHT_SMALL_GRID`/`WRIGHT_SMALL_DECORATIONS`, sets `small` in `prepareWright`, and `buildWright` caps `columns`/`rows` and `decorationCount`. `WRIGHT_DRAW_ORDER` and `WRIGHT_SPEC_VERSION = 'wright-geometry-v1'` are untouched (no diff on those lines); `node --test test/wright.test.js` passes 19/19.
- [x] At `size >= 28` (64/72/140) geometry is byte-identical to pre-feature output and the existing derive identities are unchanged — **pass**. Independent golden walk of `test/fixtures/golden-wright-v1.json` vs `origin/main` found **6 changed size-24 keys and 0 unexpected non-size-24 changes**; `git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json` passed. The derive-freeze tests (draw order, `maya`/`wright-family-5` identities) pass.
- [x] Every emitted layer stays within `WRIGHT_VIEWBOX_BOUNDS` (5–95), strokes remain `>= 1.3`, grid modules keep a minimum pane footprint, and output is free of `NaN`/`Infinity` at sizes 24/64/72/140 — **pass**. The reworked test `wright legibility reduces detail at size 24 and keeps full detail and invariants at 64, 72, and 140` asserts all four invariants and passes.
- [x] WCAG structural contrast stays `>= 3:1` and the red painted-area ratio stays `<= 0.10` at small sizes after reduction — **pass**. `wright contrast enforces … 3:1` and `wright red painted area stays within the 10 percent ceiling …` both pass in the independent `npm run verify` run; the red-area sweep covers size 24 post-reduction.
- [x] `test/fixtures/golden-wright-v1.json` is regenerated and only the six `{"size":24}` keys change; all other Wright entries and every non-Wright golden fixture are byte-identical — **pass**. `node scripts/generate-golden.mjs` then `npm run check:variants` (all five checks ✓), `git diff --exit-code` on the three non-Wright fixtures passed, and the independent key walk reported 6 size-24 changes / 0 others.
- [x] The Wright variant remains recognizable at small sizes and retains architectural character at larger sizes at `local:3184` — **pass**. Re-drove independently: served `python3 -m http.server 3184`, selected Wright in the variant `<select>` (hero aria-label `demo-agent: textile-block Wright composition, 5 planes, working` at size 140), and injected `renderStaticSVG` at sizes 24 and 64; screenshots at `evidence/review-step-3-2-wright-large-140.png` and `evidence/review-step-3-2-wright-small-vs-full.png` show simpler reduced geometry at 24 vs full grid detail at 64/140.
- [x] Final diff changes only `src/variants/wright.js`, `test/wright.test.js`, `test/fixtures/golden-wright-v1.json`, and delivery artifacts; no demo/docs/React/type declarations/other variants change; `npm run verify` passes — **pass**. `git diff origin/main...HEAD --name-only` returns exactly those three product files; `npm run verify` → 180/180 tests, `check:variants` all ✓ (no lint configured).

## Plan vs implementation

The implementation matches the plan and the four recorded decisions (defaults): single `size < 28` threshold, `2×2` grid / `2` decoration caps only (planes and Usonian second mass untouched), large-size byte-identity, and a frozen derivation spec with no new PRNG draws. The accent stroke was left at `2.2` because the red-area ceiling was not breached after reduction — consistent with the plan's "narrow only if breached" escape hatch, which correctly was not needed.

One benign in-flight roadmap adjustment: steps 1.2 and 2.1 were reworded during the build to split "code + count-assertion rework" (1.2) from "remaining invariants" (2.1), because the build step's full-file verify could not pass until the stale "retains detail" assertion was replaced. No step was added or deleted; total remained 7. This is recorded, not a deviation from plan scope.

## Roadmap audit

- All 7 ticked boxes spot-checked against the codebase and confirmed: the constants/`small` flag (1.1), the `buildWright` caps + test rework (1.2), the invariant assertions (2.1), the contrast/red-area confirmation (2.2), the regenerated golden with only size-24 changes (3.1), the browser evidence with linked screenshots and completion date (3.2), and the integrated full gate (3.3) are each present and correct.
- No falsely ticked boxes; no `(manual)` or `(manual, post-ship)` steps present. No missing-work steps added and no repairs required by the Reviewer.
- The two build-time roadmap commits that reworded 1.2/2.1 and attached evidence to 3.2 were pushed and are consistent with the final artifact state.

## Findings

1. **Minor — unused import in `test/wright.test.js`.** `WRIGHT_SMALL_SIZE` is imported (line 12) but never referenced in the test body; the legibility test uses the literal `size === 24` branch rather than `size < WRIGHT_SMALL_SIZE`. No lint tool is configured so this is not caught automatically, and it has no behavioral impact. Cleanup is optional; the test could either use the constant in the branch or drop the import.
2. **Informational — concurrent merge coordination.** `#21` `wright-variant-motion-events` is still open and touches the same three files (`src/variants/wright.js`, `test/wright.test.js`, `test/fixtures/golden-wright-v1.json`). The edits are in disjoint regions and this PR is standalone-clean, but whichever of `#21`/`#22` merges second must integrate the other's `origin/main` and re-run `node --test test/wright.test.js && npm run verify` (already the plan's stated mitigation).

## Follow-ups

None. No work items need to become new issues.
