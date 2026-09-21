# Wright variant geometry grammar

## Problem

Implement the `### wright-variant-geometry-grammar` member block from [initiatives/2026/09/frank-lloyd-wright-variant/breakdown.md](../../../../initiatives/2026/09/frank-lloyd-wright-variant/breakdown.md):

- Summary: implement deterministic Wright composition families and hierarchical geometry grammar (primary mass, horizontal planes, grid/decorative layers, accents) from seed-derived parameters.
- Brief: translate Prairie, art-glass, textile-block, and Usonian design language into constrained procedural geometry rather than random rectangles.

The current Wright scaffold renders one rectangular frame, three to five horizontal bands, and one vertical accent from a small hash-derived parameter set in `src/variants/wright.js`. It is deterministic and contract-valid, but it does not yet express the layered architectural hierarchy needed for immediate Wright-family recognition across seeds.

User-visible effect: selecting the existing `wright` variant produces recognizable, seed-stable compositions drawn from all four requested design families, including controlled hybrids, while preserving the existing prismicon renderer contract.

## Decisions

Clarifying questions and answers are retained verbatim:

1. Which composition families should this feature implement: Prairie, art-glass, textile-block, Usonian, or all four?
   - `all`
2. Should seeds select one family exclusively, or allow controlled hybrid compositions?
   - `hybrid`
3. What matters more: immediate Wright-family recognizability or greater seed-to-seed diversity?
   - `immediate wright family recognizability`
4. Should small-size detail reduction remain entirely deferred, apart from preventing clipping or invalid geometry?
   - `Ye`
5. Which icon sizes should the Builder use for local visual review?
   - `Whatever is consistent with what we have done in the past`

For question 4, `Ye` is interpreted as yes. Historical delivery artifacts use size 64 for deterministic geometry checks, size 24 for compact renderer parity, and the demo uses size 72 for variant comparisons, so this plan uses 24, 64, and 72. At size 24, this feature checks only valid bounded output; aesthetic detail reduction remains assigned to `wright-variant-legibility-tuning`.

## Research

Skills consulted: modern-javascript-patterns

- `src/variants/wright.js` owns the complete Wright pipeline. `deriveWright` currently derives `lineCount`, `inset`, `horizon`, `cantilever`, and `emphasis` directly from `cyrb53`; `buildWright` returns a frozen `{ frame, bands }` model; and `paintWright` renders that model as a frame, bands, and accent line. This is the controlling implementation surface for the geometry feature.
- `src/variants/registry.js` defines the stable hook boundary (`derive`, `prepare`, `geometry`, `pose`, `animate`, `paint`, and `flash`). Geometry is computed once per instance and treated as immutable, so the richer grammar should remain inside the existing Wright hooks rather than changing the registry or renderer.
- `src/variants/orbit.js` demonstrates fixed-budget PRNG derivation, frozen nested geometry, direct 100-by-100 viewBox construction, and stable painting order. `src/variants/ncube.js` and `src/variants/polyhedron.js` reinforce that identity-affecting draw order is documented and spec-versioned.
- `src/variants/seed.js` provides the shared `cyrb53` and `mulberry32` primitives. Using a fixed draw budget independent of selected family and layer counts prevents conditional PRNG consumption from destabilizing identities when grammar branches differ.
- `test/wright.test.js` already covers normalized deterministic derivation, frozen scaffold geometry, deterministic motion/settling, bounded paint hooks, and descriptor validation. It is the focused test surface to expand with family coverage, hierarchy invariants, bounds checks, and deterministic SVG assertions.
- `demo/index.html` derives its built-in comparison list from the registry and renders variants at size 72, so no demo source change is required for local visual review. Existing artifacts also use size 64 for geometry measurement and size 24 for compact renderer parity.
- Palette inputs (`hue`, `hue2`), pose/animation behavior, and flash behavior already live in `src/variants/wright.js`. This feature may adapt painting to the richer geometry model, but must preserve those existing color and motion semantics so `wright-variant-palette-system` and `wright-variant-motion-events` retain clear ownership.

Lint baseline (policy section 5): `AGENTS.md` declares `Lint: none`, and `package.json` defines no `lint` script. The explicit probe `npm run lint` exited 1 with `Missing script: "lint"`; there are no lint findings to clean up or scope around. The repository's documented replacement gate is `npm run verify` (`npm test && npm run check:variants`). In the fresh planning worktree, both constituent commands exited 1 before source changes because dependencies were not installed and Node could not resolve `jsdom`. The roadmap therefore begins with `npm ci` and requires a clean `npm run verify`; no failing source assertion was observed.

Overlap decision:

- `gh pr list --state open --json number,headRefName` returned `[]` for `david-perry-software/prismicon`.
- The equivalent open-PR query returned `[]` for `david-perry-software/prismicon-docs`.
- There is no active delivery overlap at planning time. The ready sibling `wright-variant-palette-system` is expected to touch `src/variants/wright.js`, so it should be sequenced after this geometry branch ships or integrate `origin/main` before each push if work begins concurrently.

## Approach

- Replace the scaffold derivation with a documented, spec-versioned, fixed-budget grammar. Derive a dominant family from Prairie, art-glass, textile-block, and Usonian; optionally derive one compatible secondary family; and cap hybrid influence so every result retains a clear primary hierarchy.
- Represent composition as explicit semantic layers: primary masses establish silhouette and focal weight; horizontal planes establish datum and cantilever rhythm; grid/decorative modules add family-specific subdivision; accents provide a restrained final emphasis. Keep family and layer quotas bounded rather than emitting arbitrary rectangles.
- Build and deeply freeze geometry in the existing 100-by-100 viewBox. Clamp extents and stroke-aware bounds so all tested seeds remain finite and unclipped at sizes 24, 64, and 72 without introducing the downstream feature's detail-suppression policy.
- Update `paintWright` to render semantic layers in a stable order while continuing to consume the scaffold's existing `hue`, `hue2`, pose, and effect values. Do not redesign palette mapping, event motion, or flash semantics.
- Expand `test/wright.test.js` with a deterministic seed corpus that covers every dominant family and controlled hybrids; assert fixed derivation, frozen nested structures, required layer hierarchy, quota and bounds invariants, no `NaN`/`Infinity`, and byte-stable repeated painting.
- Use the existing demo through a temporary local static server on the allocated feature port 3172. Review representative pure-family and hybrid seeds at 24, 64, and 72 in light and dark modes, capturing browser evidence without modifying demo source.
- Keep the lint decision explicit: no lint command exists, so changed JavaScript is gated by the focused Wright test plus the complete documented `npm run verify` suite.

Primary files expected in scope:

- `src/variants/wright.js`
- `test/wright.test.js`

## Risks

- Hybrid composition can read as random decoration rather than Wright-family structure. Mitigation: one dominant family always owns the primary mass and datum; a compatible secondary family may affect only bounded decorative or accent slots.
- Four families can cause conditional random draws and unstable identities. Mitigation: document and test a fixed PRNG draw order and consume the maximum draw budget regardless of selected family or layer count.
- Rich geometry can escape the viewBox or produce degenerate paths. Mitigation: centralize clamping/minimum dimensions and test every generated coordinate for finiteness, positive area where required, and stroke-aware bounds.
- The feature could absorb palette, motion, or small-size tuning owned by later initiative members. Mitigation: preserve current hue/pose/flash semantics and limit size 24 work to validity and clipping prevention.
- `wright-variant-palette-system` is ready and will likely overlap `src/variants/wright.js`. Mitigation: sequence that feature after this one ships; if it starts concurrently, merge `origin/main` before every push and resolve against the finalized geometry model.
- Visual recognizability is partly qualitative. Mitigation: combine semantic structural assertions with local browser review of a representative family/hybrid seed matrix in both themes, retaining screenshots as evidence.
- Planning verification was initially blocked by absent dependencies. Mitigation: make `npm ci` the first roadmap step and stop if `npm run verify` reveals a source baseline failure after installation.

## Out of scope

- Wright-specific material palette families, contrast policy, or accent-color budgets.
- New state/event animation motifs or changes to existing pose, animate, and flash semantics.
- Aesthetic small-size simplification, ornament thresholds, or production legibility tuning beyond preventing invalid or clipped output.
- Wright golden fixtures, performance benchmarking, and event-frame regression expansion assigned to `wright-variant-regression-suite`.
- Demo, README, React, type declaration, registry, or public API changes.
- Reproducing any specific Frank Lloyd Wright building, window, textile block, logo, or historical ornament.

## Acceptance checklist

- [ ] Deterministic derivation covers Prairie, art-glass, textile-block, and Usonian as dominant families and produces controlled hybrids without conditional draw-order drift; verify with named cases and repeated/deep-equality assertions in `node --test test/wright.test.js`.
- [ ] Every generated composition has a recognizable hierarchy of primary mass, horizontal plane, grid/decorative, and restrained accent layers within documented quotas; verify with semantic invariant assertions in `test/wright.test.js` and browser evidence at `local:3172`.
- [ ] Geometry and SVG output are immutable, finite, deterministic, and stroke-aware within the 100-by-100 viewBox at sizes 24, 64, and 72; verify with focused bounds/output tests and no `NaN` or `Infinity` in rendered markup.
- [ ] Existing Wright palette inputs and pose/animate/flash semantics remain unchanged while painting consumes the richer geometry model; verify with retained motion/flash assertions plus focused paint tests in `test/wright.test.js`.
- [ ] Existing variants, public exports, package contents, and golden fixtures remain unchanged; verify with `npm run check:variants`.
- [ ] The repository's no-lint configuration remains accurate and the complete documented replacement gate passes after dependency installation; verify `package.json` still has no `lint` script and `npm run verify` exits 0.