# Wright variant palette system

## Problem

The Wright variant currently derives two generic hues from the shared polyhedron palette and paints every structural layer from those hues. It therefore lacks the constrained architectural material families, semantic color roles, measurable contrast guarantees, and restrained red usage required by the `wright-variant-palette-system` member of the [Frank Lloyd Wright variant breakdown](../../../../initiatives/2026/09/frank-lloyd-wright-variant/breakdown.md#wright-variant-palette-system).

This feature must make Wright output select among textile, stained-glass, and concrete/wood palettes deterministically while preserving the existing variant API, geometry identity, and later initiative ownership of motion, legibility, golden fixtures, demo changes, and documentation.

## Decisions

1. Palette families: Three material palettes (textile, stained-glass, and concrete/wood families).
2. Contrast metric: WCAG contrast ratio.
3. Red accent budget: Painted area (not element count).
4. Feature boundary: Code and focused tests only; leave golden fixtures and demo/docs to later initiative features.

## Research

Skills consulted: modern-javascript-patterns

- `src/variants/wright.js` owns the complete Wright derive/prepare/geometry/paint pipeline. `deriveWright` currently consumes the frozen `WRIGHT_DRAW_ORDER`, then maps the seed hash to `PALETTE`-derived `hue` and `hue2`; `paintWright` turns those generic hues into all layer colors. Palette selection can therefore remain in the owning variant module and derive independent values from the existing stable hash without consuming or reordering geometry random draws.
- `src/variants/wright.js` paints the semantic layers in stable order (`primary-mass`, `horizontal-plane`, `grid-module`, `decoration`, `accent`) and receives light/dark and transient effect state in `paintWright`. Those existing boundaries support semantic palette roles and contrast enforcement without a new subsystem or public option.
- `test/wright.test.js` already verifies normalized deterministic derivation, all four geometry families, frozen structures, bounded geometry, layer order, stable painting, and flash behavior. It is the focused location for palette-family reachability, WCAG calculations, painted-area budgeting, and preservation of geometry/motion contracts.
- The initiative breakdown requires a constrained architectural mapper integrated with the existing color-generation path and explicitly defers golden fixture ownership to `wright-variant-regression-suite` and demo/documentation ownership to `wright-variant-react-demo-docs`.
- WCAG 2.2 Success Criterion 1.4.11 defines a `3:1` minimum contrast ratio for meaningful graphical objects against adjacent colors, evaluated without rounding. This plan applies that threshold to Wright structural roles in both light and dark contexts: <https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html>.
- Full-repository lint baseline: command `none configured` (AGENTS.md declares `Lint: none`, and `package.json` has no lint script); exit status `N/A`; findings: no lint baseline exists. Because there are no lint findings to overlap, changed JavaScript is gated by `node --test test/wright.test.js` and the repository maintainer gate `npm run verify` (`npm test` plus `npm run check:variants`). Baseline runs on 2026-09-21 passed: `npm test -- test/wright.test.js` exited 0 and `npm run check:variants` exited 0.
- Concurrent-delivery check: `gh pr list --state open --json number,headRefName,title` returned an empty list on 2026-09-21, so no open delivery branch overlaps the planned files. Recheck before push and integrate `origin/main` as required if new overlapping work appears.

## Approach

- In `src/variants/wright.js`, define deeply frozen textile, stained-glass, and concrete/wood palette-family records with semantic roles for canvas/background reference, primary structure, secondary material, line/grid, and accent. Use architectural, non-neon sRGB colors and keep all palette behavior local to the Wright variant.
- Select the palette family and any bounded within-family variation deterministically from the existing normalized seed/hash without adding to or reordering `WRIGHT_DRAW_ORDER`. Retain the current geometry parameters and public variant descriptor contract; advance the Wright internal spec only if the established spec-version convention requires it for changed rendered identity.
- Resolve light and dark semantic roles through WCAG relative luminance and contrast-ratio helpers. Ensure every structural role required to perceive the composition has an unrounded contrast ratio of at least `3:1` against its actual adjacent canvas or material role, including current flash/lighten states.
- Integrate semantic colors directly in `paintWright`, preserving layer order, valid finite SVG, and existing geometry, pose, animate, and flash ownership. Do not introduce a package-level color API or alter unrelated variants.
- Define a strict red painted-area ceiling of `10%`. Compute the red contribution from rendered fill area and stroke footprint, compare it with total painted geometry area, and use or constrain the red role so every tested composition stays at or below the ceiling; element counts are not an acceptable proxy.
- Extend `test/wright.test.js` with deterministic seed cases reaching all three palette families, frozen-role and repeatability assertions, independent geometry-derivation checks, standards-based sRGB contrast calculations for light/dark and transient states, and painted-area checks over a representative seed/size corpus.
- Keep the lint decision explicit: no lint command exists, so the focused Wright test is the per-file behavior gate and `npm run verify` is the full repository gate. No golden fixture, demo, README, type declaration, React, or other product file changes are permitted by this feature boundary.

## Risks

- Adding palette entropy could accidentally perturb frozen geometry identities. Mitigation: derive palette choices independently from the existing hash, preserve `WRIGHT_DRAW_ORDER`, and assert geometry-relevant values for representative seeds.
- Contrast can pass in the idle light theme but fail in dark mode or after flash/lighten effects. Mitigation: calculate WCAG relative luminance from final emitted colors and exercise both themes plus every current non-null flash state without rounding ratios.
- A red line can appear restrained by element count while occupying too much visual area. Mitigation: enforce and test a `10%` ceiling using fill area and stroke footprint across varied geometries, sizes, and pulse states.
- Material palettes may become visually interchangeable or overly saturated despite satisfying numeric checks. Mitigation: use distinct frozen semantic-role tables, cover all families with named seeds, and inspect the existing demo at `local:3141` without changing demo source.
- A concurrent delivery may begin after planning and touch `src/variants/wright.js` or `test/wright.test.js`. Mitigation: re-list open PR files and merge `origin/main` before every push; sequence after any newly overlapping slug if integration would combine unstable Wright behavior.
- The repository has no lint command. Mitigation: require focused Node tests for every palette invariant and a final `npm run verify`; any newly introduced lint configuration is outside this feature and cannot be silently substituted.

## Out of scope

- Golden fixture creation or updates, including Wright golden baselines.
- Changes to `demo/`, README or other documentation, React integration, or type declarations.
- Motion-event redesign, small-icon geometry/LOD tuning, or changes to geometry grammar.
- New public options, exports, generalized palette APIs, or behavior changes to polyhedron, ncube, and orbit variants.
- Reproduction of a specific historical artwork, building, textile block, or stained-glass design.

## Acceptance checklist

- [ ] Deterministic normalized seeds select all three deeply frozen palette families (textile, stained-glass, and concrete/wood) with stable semantic roles while existing geometry draw order and geometry-derived values remain unchanged; verify with named and corpus assertions in `node --test test/wright.test.js`.
- [ ] Wright paint output uses the selected architectural palette in light and dark contexts, remains deterministic and finite, preserves semantic SVG layer order, and does not change the public variant API; verify with focused paint/contract assertions in `test/wright.test.js`.
- [ ] Every meaningful structural role meets an unrounded WCAG contrast ratio of at least `3:1` against its adjacent role in light, dark, and current flash/lighten states; verify with standards-based sRGB relative-luminance assertions in `test/wright.test.js`.
- [ ] Taliesin-like red occupies no more than `10%` of estimated painted geometry area for every representative seed, tested size, and pulse state, with fill area and stroke footprint rather than element count used by the assertion; verify in `node --test test/wright.test.js`.
- [ ] Representative outputs for all three palette families are visually distinct, architectural, non-neon, and legible in light and dark mode at `local:3141`; verify through the existing demo without modifying demo source and capture delivery evidence.
- [ ] The final diff changes only `src/variants/wright.js`, `test/wright.test.js`, and delivery artifacts; no golden fixtures, demo/docs, React, type declarations, or unrelated variants change, and `npm run verify` passes with no lint command available.