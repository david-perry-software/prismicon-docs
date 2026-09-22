# Review: wright-variant-legibility-tuning

Verdict: approve

## Acceptance checklist results

All seven checklist items pass; re-verified independently on 2026-09-21 after the post-approval integration of `origin/main` (see `## Roadmap audit`).

- [x] `prepareWright` exposes a `small` flag at `size < 28` and `buildWright` reduces grid modules to at most `2×2` and decorations to at most `2` below that threshold, while `deriveWright`, `WRIGHT_DRAW_ORDER`, and `WRIGHT_SPEC_VERSION` stay frozen — **pass**. `src/variants/wright.js` adds `WRIGHT_SMALL_SIZE` (28) / `WRIGHT_SMALL_GRID` (2×2, frozen) / `WRIGHT_SMALL_DECORATIONS` (2), sets `small` in `prepareWright`, and `buildWright` caps `columns`/`rows` and `decorationCount` when `params.small`. `git diff origin/main...HEAD -- src/variants/wright.js` shows no edit to `WRIGHT_DRAW_ORDER`/`WRIGHT_SPEC_VERSION` (only a comment line naming them); `npm test` → 188/188.
- [x] At `size >= 28` (64/72/140) geometry is byte-identical to the pre-feature baseline and derive identities are unchanged — **pass**. Independent golden walk of `test/fixtures/golden-wright-v1.json` vs `origin/main` found **6 changed keys, all `{"size":24}`, 0 non-size-24 changes**; injected `renderStaticSVG` renders at 64 and 140 produce identical counts (12 rects / 9 lines) while size 24 produces 10 rects / 3 lines.
- [x] Every emitted layer stays within `WRIGHT_VIEWBOX_BOUNDS` (5–95), strokes remain `>= 1.3`, grid modules keep a minimum pane footprint, and output is free of `NaN`/`Infinity` at sizes 24/64/72/140 — **pass**. The test `wright legibility reduces detail at size 24 and keeps full detail and invariants at 64, 72, and 140` asserts all four invariants and passes in `npm test`.
- [x] WCAG structural contrast stays `>= 3:1` and the red painted-area ratio stays `<= 0.10` at small sizes after reduction — **pass**. `wright contrast … 3:1` and `wright red painted area … 10 percent ceiling` pass in `npm test`; the red-area sweep covers size 24 post-reduction.
- [x] `test/fixtures/golden-wright-v1.json` is regenerated and only the six `{"size":24}` keys change; all other Wright entries and every non-Wright golden fixture are byte-identical — **pass**. `git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json` is empty; the independent key walk reports 6 size-24 changes / 0 others; `check:variants` `goldens` ✓.
- [x] The Wright variant remains recognizable at small sizes and retains architectural character at larger sizes at `local:3184` — **pass**. Re-drove independently: served `python3 -m http.server 3184`, selected Wright in the variant `<select>` (hero `demo-agent: textile-block Wright composition, 5 planes, working`), injected `renderStaticSVG` at 24/64/140, and captured fresh screenshots `evidence/review-step-3-2-wright-small-vs-full.png` and `evidence/review-step-3-2-wright-large-140.png` — size 24 shows a simplified compact frame while 64/140 show the full grid with a restrained red accent.
- [x] Final diff changes only `src/variants/wright.js`, `test/wright.test.js`, `test/fixtures/golden-wright-v1.json`, and delivery artifacts; no demo/docs/React/type declarations/other variants change; `npm run verify` passes with no lint configured — **pass**. `git diff origin/main...HEAD --name-only` returns exactly those three product files; `npm test` → 188/188 and `npm run check:variants` → contract/exports/types/pack/goldens all ✓ (no lint script in `package.json`).

## Plan vs implementation

The implementation matches the plan and the four recorded decisions (defaults): single `size < 28` threshold, `2×2` grid / `2` decoration caps only (horizontal planes and the Usonian second mass untouched), large-size byte-identity, and a frozen derivation spec with no new PRNG draws. The accent stroke stays at `2.2` because the red ceiling was not breached — consistent with the plan's "narrow only if breached" escape hatch, which was not needed.

Post-approval deviation from the original plan scope, fully documented in the roadmap: the concurrent `#21` `wright-variant-motion-events` merged into `origin/main` after the first review, so this branch had to integrate it. Step 5.1 (added 2026-09-21) was completed: `origin/main` merged into both halves, the `prepareWright` conflict resolved by keeping both features' disjoint changes (`small` flag + stroke swap from legibility, `...wrightMotionTraits(params)` from motion), the golden regenerated from the combined code, and the full gate re-run. The net product diff vs `origin/main` is unchanged in scope (still the same three files) — `#21`'s code now lives in the baseline, so this feature's own change is still purely the legibility reduction.

## Roadmap audit

- All 10 ticked boxes spot-checked against the codebase and confirmed: 1.1 constants/flag; 1.2 `buildWright` caps + test rework; 2.1 invariants; 2.2 contrast/red ceiling; 3.1 regenerated golden (only size-24 keys); 3.2 browser evidence (linked screenshots + completion date, re-verified with fresh screenshots); 3.3 integrated full gate; 4.1 single-sourced `WRIGHT_SMALL_SIZE` threshold; 4.2 historical concurrent-coordination reconfirmation; 5.1 post-merge integration of `#21`.
- No falsely ticked boxes. No `(manual)` or `(manual, post-ship)` steps. Step 5.1 is the missing-work step added for the `#21` integration; no further repairs required.
- Step 4.2's description ("`#21` has not merged into `origin/main`") was true when ticked; it is now superseded by `#21`'s merge and covered by step 5.1 — recorded, not a false tick.
- Companion half ends clean: `dirty: false`, `ahead: 0`, `behind: 0`; `origin/main` is an ancestor of both halves' HEAD; product PR `#22` and companion PR `#7` are both `CLEAN`.

## Findings

None above minor severity. The conflict resolution in `prepareWright` correctly unions the two features: `strokeWidth`/`lightStroke` use the single-sourced `small` flag, and `...wrightMotionTraits(params)` is preserved. The regenerated golden reflects the combined code and differs from `origin/main` only in the six size-24 keys, so large-size byte-identity still holds.

## Follow-ups

None. No work items need to become new issues.
