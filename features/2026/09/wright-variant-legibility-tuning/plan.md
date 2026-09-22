# Wright variant legibility tuning

## Problem

The Wright variant renders the same full geometry at every size. `prepareWright` (`src/variants/wright.js`) is the only size-aware chokepoint, and today it only *thickens* strokes below `size < 28` — it reduces no detail, so the 2–5 column/row grid, up to 8 decorations, and the full plane stack all survive at 24 px. Thin grid panes and decoration lines collapse into visual noise at small avatar sizes.

This feature implements the `wright-variant-legibility-tuning` member of the [Frank Lloyd Wright variant breakdown](../../../../initiatives/2026/09/frank-lloyd-wright-variant/breakdown.md#wright-variant-legibility-tuning): graceful, size-aware detail reduction using the existing `prepare`/`geometry` thresholds rather than a new level-of-detail framework, so icons stay recognizable at small sizes while retaining architectural character at larger sizes. It does not touch motion, demo/docs, the public API, or any other variant.

## Decisions

Clarifying questions were asked before writing (the structured question tool was unavailable, so the §10 fallback was used). The user answered every question with **"defaults"**. The defaults resolve as follows:

1. **Reduction scope** — A: "defaults" → minimal: stroke-width thresholds plus a modest count reduction (fewer grid modules and decorations). Horizontal-plane count and the Usonian second mass are not reduced.
2. **Size bands** — A: "defaults" → a single `size < 28` threshold, matching the existing polyhedron/ncube convention; canonical verification sizes are 24, 64, 72, and 140 (the sizes the existing tests and golden fixture already exercise).
3. **Large-size stability** — A: "defaults" → output at `size >= 28` stays byte-identical; only the size-24 golden entries change.
4. **Derivation freeze** — A: "defaults" → `WRIGHT_DRAW_ORDER` (the 12 sequential draws) and `WRIGHT_SPEC_VERSION = 'wright-geometry-v1'` stay frozen; all reduction is computed in `prepareWright`/`buildWright` from prepared params, with no new PRNG draws.
5. **Acceptance** — A: "defaults" → automated verification is sufficient; no `(manual)` step is required.

## Research

Skills consulted: modern-javascript-patterns (vercel-react-best-practices is listed in the project skills table but does not apply — no React/Next code is touched).

- `src/variants/wright.js` owns the complete Wright pipeline. `deriveWright` (lines 192–241) consumes the frozen `WRIGHT_DRAW_ORDER` (12 sequential `mulberry32` draws) and is never size-aware. `prepareWright` (lines 247–253) is the single size chokepoint and today only swaps stroke widths at `size < 28` (`3`/`1.8` vs `2.2`/`1.3`). `buildWright` (lines 255–322) reads prepared params but never a size flag: it derives grid modules from per-family `profiles` (prairie and usonian fixed at `3×2`; art-glass and textile-block from `params.gridColumns`/`params.gridRows`) and a `decorationCount = min(8, 2 + decoration*2 + (secondaryFamily ? 2 : 0))`.
- The established size-aware pattern in the other built-ins is `prepare` → `{ ...params, adjustedFields }` at the literal `size < 28` threshold: `polyhedron.js:206-209` and `ncube.js:189-192` downgrade wireframe to shaded; `orbit.js:106-108` ignores size. No variant yet reduces counts beyond the finish flip, so this feature is the first count reduction and should keep the same `prepare`-derived-field shape that `buildWright` then consumes — the same non-spec, prepare-derived technique ncube/orbit use for `motionTraits`.
- `paintWright` and `paintedAreaMetrics` both consume `params.strokeWidth`/`params.lightStroke`, and the red accent budget (`WRIGHT_RED_AREA_CEILING = 0.10`, `wright.js:46`) is measured as the accent line's stroke footprint against total painted area. Reducing grid/decoration counts shrinks the denominator, so the red ratio rises at small sizes; the plan keeps the accent unchanged and only narrows it at small sizes if the ceiling is actually breached.
- `test/wright.test.js` is the focused location. The size test at lines 149–183 currently asserts element counts are *identical* across 24/64/72 ("retains detail at size") — this is the assertion that must flip to expect reduction at 24. The red-area test (lines 329–353) already sweeps sizes 24/64/72/140 and will keep passing against reduced geometry; the contrast test uses size 64 only and is unaffected.
- Golden freezing: `test/helpers/golden.js` maps `wright` → `test/fixtures/golden-wright-v1.json`. `STATIC_OPTS` includes `{ size: 24 }` and `{ state: 'error', size: 140 }`; `MOUNTED_SCENARIOS` includes `{ seed: 'maya', opts: { size: 24 } }`. Under decision 3 (large sizes byte-identical), exactly the six `{"size":24}` keys change (five static + one mounted `maya`), and every other Wright entry plus all non-Wright fixtures stay byte-identical.
- Full-repository lint baseline: command `none configured` (AGENTS.md declares `Lint: none`; `package.json` has no lint script); exit status `N/A`; findings: none — no lint baseline exists. Because there are no lint findings to overlap, the gate is: changed files are linted-equivalent to nothing, so every changed JavaScript file (`src/variants/wright.js`, `test/wright.test.js`) is covered by the focused suite `node --test test/wright.test.js` plus the repository maintainer gate `npm run verify` (`npm test` + `npm run check:variants`). Baseline runs on 2026-09-21: `npm test` → 180 pass / 0 fail; `npm run check:variants` → all five checks pass.
- Concurrent-delivery check: `gh pr list --state open --json number,headRefName,title` returned an empty list in both the product and companion repositories on 2026-09-21, so no open delivery branch overlaps the planned files. Recheck before push and integrate `origin/main` as required if new overlapping work appears.

## Approach

- In `src/variants/wright.js`, add deeply frozen reduction constants following the existing `WRIGHT_*` naming: `WRIGHT_SMALL_SIZE = 28`, `WRIGHT_SMALL_GRID = Object.freeze({ columns: 2, rows: 2 })`, and `WRIGHT_SMALL_DECORATIONS = 2`. Keep the existing small/large stroke widths (all already at or above the `1.3` minimum stroke floor).
- Extend `prepareWright` to set `small: size < WRIGHT_SMALL_SIZE` alongside the existing stroke-width swap, without touching `deriveWright`, `WRIGHT_DRAW_ORDER`, or `WRIGHT_SPEC_VERSION`.
- Make `buildWright` consume the flag: when `params.small` is true, cap the effective grid columns/rows to `WRIGHT_SMALL_GRID` (applies uniformly across prairie, art-glass, textile-block, and usonian profiles, whose fixed `3×2` grids also reduce to `2×2`) and cap `decorationCount` to `WRIGHT_SMALL_DECORATIONS`. When `params.small` is false the computation is unchanged, so `size >= 28` output is byte-identical.
- Preserve the semantic layer order (`primary-mass`, `horizontal-plane`, `grid-module`, `decoration`, `accent`), finite valid SVG, and the existing pose/animate/flash behavior. No new public options, exports, or behavior changes to other variants.
- Update `test/wright.test.js`: replace the "retains detail at size" assertion with a legibility suite that asserts (a) reduced grid-module and decoration counts at size 24 versus full counts at 64/72, (b) every layer inside `WRIGHT_VIEWBOX_BOUNDS` (5–95) at sizes 24/64/72/140, (c) strokes `>= 1.3` and grid modules keeping a minimum pane footprint, (d) no `NaN`/`Infinity`, and (e) the existing derivation-freeze identities for `maya`/`wright-family-5` unchanged.
- Keep the contrast (`>= 3:1`) and red-area (`<= 0.10`) tests green across their existing size sweeps. If, after grid/decoration reduction, any tested seed at size 24 exceeds the red ceiling, narrow the accent stroke width at small sizes (floor `1.3`, documented as `WRIGHT_SMALL_ACCENT_WIDTH`) rather than restoring detail.
- Regenerate `test/fixtures/golden-wright-v1.json` with `node scripts/generate-golden.mjs` and confirm only the six `{"size":24}` keys change; all other Wright entries and every non-Wright golden fixture stay byte-identical.
- Keep the lint decision explicit: no lint command exists, so `node --test test/wright.test.js` is the per-file behavior gate and `npm run verify` is the full-repository gate.

## Risks

- Over-reduction could make small icons unrecognizable or under-reduction could leave them noisy. Mitigation: single `2×2` grid / `2` decoration caps chosen to match the codebase threshold convention; bounds, minimum-pane, and stroke-floor assertions; and a browser check at `local:3184` across small and large sizes.
- Golden churn could mask accidental large-size drift. Mitigation: decision 3 freezes `size >= 28`; the golden step regenerates once and verifies only `{"size":24}` keys differ while non-Wright fixtures stay byte-identical.
- Reduced detail shrinks the painted-area denominator and can raise the red ratio toward `WRIGHT_RED_AREA_CEILING`. Mitigation: keep the existing ratio test sweeping size 24; narrow the accent at small sizes only if the ceiling is breached.
- Derivation freeze could be violated by adding draws. Mitigation: all reduction is prepare-derived from existing params; no new PRNG draws; freeze tests assert `WRIGHT_DRAW_ORDER`/`WRIGHT_SPEC_VERSION` and representative derive identities.
- A concurrent delivery may begin after planning and touch `src/variants/wright.js`, `test/wright.test.js`, or `test/fixtures/golden-wright-v1.json`. Mitigation: re-list open PR files and merge `origin/main` (product) before every push; sequence after any newly overlapping slug if integration would combine unstable Wright behavior.
- The repository has no lint command. Mitigation: require focused Node tests for every legibility invariant plus a final `npm run verify`; no lint configuration may be silently introduced.

## Out of scope

- A generalized multi-variant LOD framework or any size tier beyond the single `size < 28` threshold.
- Changes to large-size output (`size >= 28`), the derivation draw order, or `WRIGHT_SPEC_VERSION`.
- Motion/event behavior (`wright-variant-motion-events`), the full regression suite (`wright-variant-regression-suite`), and demo/docs updates (`wright-variant-react-demo-docs`).
- Changes to `demo/`, README/docs, React integration, type declarations, the public API, or polyhedron/ncube/orbit output.
- Reproduction of any specific Wright building, window, textile block, logo, or ornament.

## Acceptance checklist

- [ ] `prepareWright` exposes a `small` flag at `size < 28` and `buildWright` reduces grid modules to at most `2×2` and decorations to at most `2` below that threshold, while `deriveWright`, `WRIGHT_DRAW_ORDER`, and `WRIGHT_SPEC_VERSION` stay frozen; verify in `node --test test/wright.test.js`.
- [ ] At `size >= 28` (64/72/140) geometry is byte-identical to pre-feature output — full grid counts and decoration counts — and the existing derive identities for representative seeds are unchanged; verify in `node --test test/wright.test.js`.
- [ ] Every emitted layer stays within `WRIGHT_VIEWBOX_BOUNDS` (5–95), strokes remain `>= 1.3`, grid modules keep a minimum pane footprint, and output is free of `NaN`/`Infinity` at sizes 24/64/72/140; verify in `node --test test/wright.test.js`.
- [ ] WCAG structural contrast stays `>= 3:1` and the red painted-area ratio stays `<= 0.10` at small sizes after reduction, across the existing seed/size/pulse sweeps; verify in `node --test test/wright.test.js`.
- [ ] `test/fixtures/golden-wright-v1.json` is regenerated and only the six `{"size":24}` keys change; all other Wright entries and every non-Wright golden fixture are byte-identical; verify with `node scripts/generate-golden.mjs`, `npm run check:variants`, and `git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json`.
- [ ] The Wright variant remains recognizable at small sizes and retains architectural character at larger sizes when driven through the unchanged demo at `local:3184`, with screenshots captured under this delivery's `evidence/`; verify via the browser at `http://localhost:3184/demo/index.html`.
- [ ] The final diff changes only `src/variants/wright.js`, `test/wright.test.js`, `test/fixtures/golden-wright-v1.json`, and delivery artifacts; no demo/docs, React, type declarations, or other variants change, and `npm run verify` passes with no lint command configured.
