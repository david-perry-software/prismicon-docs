# Wright variant React, demo, and documentation

## Problem

The Frank Lloyd Wright variant is fully implemented and merged: `wright-variant-spec-scaffold`, `wright-variant-geometry-grammar`, `wright-variant-palette-system`, `wright-variant-motion-events`, `wright-variant-legibility-tuning`, and `wright-variant-regression-suite` are all `status: complete`, and `src/variants/wright.js` ships the complete frozen pipeline. But the companion-facing finish line is still open: the `wright` family is invisible to anyone discovering the package.

- `README.md` documents only `polyhedron` (default), the n-cube family, and `orbit`. Its `listVariants()` example, its "Variants" intro sentence, and its `handle.params` narrowing note all omit `wright`.
- `index.d.ts` does not list `'wright'` in `BuiltInVariantId`, has no `WrightParams` interface, and `GlyphHandle.params` is `GlyphParams | NcubeParams | OrbitParams` — so TypeScript callers cannot autocomplete or narrow the new family.
- `demo/index.html` has per-family sections for the n-cube dimensions, lifecycle, and `orbit`, but no Wright section. The variant `<select>` is populated from `listVariants()` so `wright` already appears there, but there is no visual showcase of the family, its composition/palette families, or its motion.

This plan implements the `wright-variant-react-demo-docs` member of the [Frank Lloyd Wright variant breakdown](../../../../initiatives/2026/09/frank-lloyd-wright-variant/breakdown.md#wright-variant-react-demo-docs), whose brief is: "Ensure all companion-facing updates are shipped with the implementation and aligned with existing variant authoring/selection conventions." It closes the initiative by making the new family discoverable, documented, and typed — without reopening its frozen derivation, geometry, motion, palette, or legibility behavior and without changing any existing variant output.

## Decisions

No blocking ambiguity remained after the user supplied recommended defaults for every open decision, so no clarifying questions were asked. Defaults adopted (each matches the user's recommended option):

1. **README depth (recommended):** add a `### Wright` subsection under `## Variants` mirroring the n-cube/orbit family sections, and update the variant intro sentence, the `listVariants()` example, and the `handle.params` narrowing note so `wright` appears alongside the other built-ins.
2. **Demo (recommended):** extend `demo/index.html` with a dedicated Wright section mirroring the existing Orbit section (seed input + `STATES` buttons + one rendered `wright` cell). No new `demo/wright-variant.js` module — `wright` is a built-in and is referenced by id exactly like `variant: 'orbit'`. The existing variant `<select>` already lists `wright` automatically via `listVariants()`.
3. **React/type declarations (recommended):** `src/react.js` needs no change (it resolves any registered id through `ctx.registry.resolve(variant)`); only `index.d.ts` changes — add `'wright'` to `BuiltInVariantId`, add `WrightParams`, and widen the `GlyphHandle.params` union. `src/variants/index.js` already registers/exports `wright`.
4. **Visual evidence (recommended):** the demo step is browser-driven by the Builder itself (delivery-policy §1) against `local:3163` with a screenshot captured to `evidence/`; the machine gate remains `npm run verify`. No `(manual)` steps and no post-ship exception — the static demo is fully browser-drivable locally.
5. **`src/variants/wright.js` stays untouched**, including its registered `label: 'Wright Scaffold'` (a cosmetic scaffold leftover). Renaming it would change a frozen file and `listVariants()` output, so it is recorded as out of scope and as a follow-up risk rather than silently changed.

## Research

Skills consulted: modern-javascript-patterns (Node.js ESM, demo module conventions, const/pure-function style); vercel-react-best-practices (React binding review — `src/react.js` inspected for the selection path; no React code change needed).

- `src/variants/index.js` already registers `wright` as a built-in: `BUILT_IN_VARIANTS = createVariantRegistry([polyhedron, ...ncubeVariants, orbit, wright], …)` and exports `wright` plus `WRIGHT_SPEC_VERSION`. `listVariants()` and `resolveVariant()` therefore already return/resolve `wright`; the built-in id list is `polyhedron, ncube, ncube-3 … ncube-6, orbit, wright`.
- `src/variants/wright.js` owns the complete frozen pipeline and must not be touched: `WRIGHT_SPEC_VERSION = 'wright-geometry-v1'`; `WRIGHT_FAMILIES` = `prairie | art-glass | textile-block | usonian` with `WRIGHT_HYBRID_COMPATIBILITY` secondary families; `WRIGHT_PALETTE_FAMILIES` = `textile | stained-glass | concrete-wood` with light/dark role sets, `WRIGHT_CONTRAST_MIN = 3` and `WRIGHT_RED_AREA_CEILING = 0.10`; motion traits (`sweepDir`, `panelPhase`, `illumSpeed`) derived from disjoint hash bits (no extra PRNG draws); small-size legibility via `WRIGHT_SMALL_SIZE = 28`. Its registered label is `'Wright Scaffold'`.
- `src/react.js` `Prismicon` resolves `variant` through `ctx.registry.resolve(variant)` with no per-variant branches, and `PrismiconProvider` accepts any `VariantRegistry` — so `wright` already works through the React binding with no code change. `test/renderer-dispatch.test.js` and `test/react-variant.test.js` exercise this dispatch.
- `index.d.ts` defines `BuiltInVariantId = 'polyhedron' | 'ncube' | 'orbit' | \`ncube-${number}\`` (no `wright`) and `GlyphHandle.params: GlyphParams | NcubeParams | OrbitParams` (no `WrightParams`). `NcubeParams`/`OrbitParams` are the exact pattern to mirror for `WrightParams`.
- `README.md` documents the built-ins under `## Variants` with an `### N-cube family` and `### Orbit` section but no Wright section; the `listVariants()` example ends at `orbit`, and the `handle.params` note names only `GlyphParams | NcubeParams | OrbitParams`.
- `demo/index.html` builds the variant `<select>` from `listVariants()` (so `wright` already appears once selected by the picker) and has dedicated sections for "Dimensions (n-cube family)", "Lifecycle (n-cube family)", "Orbit", and "Custom variant". The Orbit section is the mirror pattern: `mountGlyph(box, seed, { kind: 'agent', size: 72, state, dark, variant: 'orbit' })`.
- Verification baseline: `npm ci` (46 packages, 0 vulnerabilities) and `npm run verify` at `origin/main` HEAD `f64913c` exit 0 — `npm test` runs 191 tests (0 fail) and `npm run check:variants` reports `✓ contract`, `✓ exports`, `✓ types`, `✓ pack`, `✓ goldens`.
- **Lint baseline (§5):** command `none configured` — AGENTS.md declares `Lint: none` and `package.json` has no lint script (scripts are `test`, `check:variants`, `verify`). Exit status `N/A`; findings: none. Overlap decision: with no lint findings there is nothing to overlap and no scoped gate is required; the gate for the touched files (`index.d.ts`, `README.md`, `demo/index.html`) is `npm run verify` (the `types` check covers `index.d.ts`; the `exports`/`pack` checks enforce the package surface stays unchanged; the demo is browser-driven at `local:3163`).
- Concurrent-delivery check: `gh pr list --state open --json number,headRefName,title` returned `[]` on 2026-09-21 in both `prismicon` and `prismicon-docs` — no open delivery branch overlaps `index.d.ts`, `README.md`, or `demo/index.html`. Recheck before push and merge `origin/main` in both halves.

## Approach

Documentation/demo/type-only feature: three product files change, nothing else. Existing variant output stays byte-identical.

1. **`index.d.ts`** — add `'wright'` to `BuiltInVariantId`; add `WrightFamily` (`'prairie' | 'art-glass' | 'textile-block' | 'usonian'`) and `WrightPaletteFamily` (`'textile' | 'stained-glass' | 'concrete-wood'`) string-literal unions; add a `WrightParams` interface mirroring `OrbitParams` with the derived fields from `deriveWright` (`spec`, `seed`, `hash`, `paletteFamily`, `dominantFamily`, `secondaryFamily: WrightFamily | null`, `massWidth`, `massHeight`, `massOffset`, `planeCount`, `planeSpread`, `gridColumns`, `gridRows`, `decoration`, `accent`, `lineCount`, `inset`, `horizon`, `cantilever`, `emphasis: 'vertical' | 'horizontal'`, `phase`, `hue`, `hue2`) and the prepared, non-identity fields from `prepareWright` (`strokeWidth`, `lightStroke`, `small`, `sweepDir: 1 | -1`, `panelPhase`, `illumSpeed`); widen `GlyphHandle.params` to `GlyphParams | NcubeParams | OrbitParams | WrightParams` and update its narrowing comment to mention `'dominantFamily' in params`.
2. **`README.md`** — add a `### Wright` subsection documenting the id/label/spec, the four composition families and hybrid rule, the three palette families with the 3:1 contrast floor and 10% red-accent ceiling, the motion-model table, small-size legibility, determinism, and the frozen `wright-geometry-v1` derivation; update the "Variants" intro sentence ("…`orbit` is the third" → name `wright` as the fourth), the `listVariants()` example (append `{ id: 'wright', label: 'Wright Scaffold', spec: 'wright-geometry-v1' }`), and the `handle.params` narrowing note.
3. **`demo/index.html`** — add a `## Wright` section mirroring the Orbit section: a seed input, a `STATES` button row, and one rendered `wright` cell via `mountGlyph(box, seed, { kind: 'agent', size: 72, state, dark, variant: 'wright' })`. The existing `<select>` needs no change (it already lists `wright` from `listVariants()`).

Unchanged (enforced by the final gate): `src/variants/wright.js` (including its `'Wright Scaffold'` label), `src/variants/index.js`, `src/react.js`, `src/core.js`, `src/authoring.js`, all `src/variants/*.js`, and all `test/fixtures/*.json`.

## Risks

- The `'Wright Scaffold'` label is a cosmetic scaffold leftover that the demo picker and `listVariants()` will surface. Mitigation: leave it untouched (renaming touches the frozen `src/variants/wright.js`); document it as a follow-up rather than change it here.
- The `WrightParams` field list could drift from `deriveWright`/`prepareWright` output. Mitigation: the field list above is copied from the source and pinned in this plan; the Builder copies it verbatim and `npm run check:variants` runs the `types` gate.
- `BuiltInVariantId` widening is compile-time only; runtime behavior is unchanged. Mitigation: no runtime code changes; `npm run verify` confirms the surface.
- README/demo edits are documentation-only, but a demo script syntax error would only surface in-browser. Mitigation: the demo step is browser-driven at `local:3163` with a captured screenshot; `check:variants`' `pack` gate keeps `demo/` unpublished.
- A concurrent delivery may start after planning and touch `index.d.ts`, `README.md`, or `demo/index.html`. Mitigation: re-list open PR files and merge `origin/main` in both halves before every push; sequence after any newly overlapping slug.
- No dev server or preview system is configured (AGENTS.md). Mitigation: the demo is served locally on the per-slug port 3163 (`python3 -m http.server 3163 --directory demo`) and stopped after verification — a `local:<ports>` target, not a post-ship exception, so no post-ship exception is requested.

## Out of scope

- Any change to `src/variants/wright.js` — its frozen constants, `WRIGHT_SPEC_VERSION`, derivation, geometry, motion, palette, legibility, or its registered `label: 'Wright Scaffold'`.
- Any change to `src/variants/index.js`, `src/react.js`, `src/core.js`, `src/authoring.js`, `src/variants/registry.js`, `src/variants/validate.js`, `src/variants/seed.js`, or any non-Wright variant.
- Any change to existing golden fixtures or any existing variant SVG output (byte-identical requirement).
- New public APIs, exports, or option keys; a new `demo/wright-variant.js` module (the family is built-in and referenced by id).
- Regression/performance coverage for the Wright family (owned by `wright-variant-regression-suite`, already complete).

## Acceptance checklist

- [ ] `index.d.ts` lists `'wright'` in `BuiltInVariantId`, defines `WrightParams` (plus `WrightFamily`/`WrightPaletteFamily`), and `GlyphHandle.params` is `GlyphParams | NcubeParams | OrbitParams | WrightParams` — verify: `npx tsc --noEmit --strict --target es2020 --lib es2020,dom index.d.ts` exits 0.
- [ ] `src/react.js` and `src/variants/index.js` are unchanged and `wright` resolves through `Prismicon`'s `variant` prop and `listVariants()` — verify: `git diff --exit-code origin/main -- src/react.js src/variants/index.js` and `node -e "import('./src/index.js').then(m=>{if(!m.listVariants().map(v=>v.id).includes('wright'))process.exit(1);console.log('wright present')})"`.
- [ ] `README.md` documents the `wright` variant (id/label/spec, composition families + hybrids, palette families, motion model, small-size legibility, determinism) and its `listVariants()` example includes `wright` — verify: `grep -n "wright" README.md` shows the new `### Wright` section and the updated example, and `npm run check:variants` still passes.
- [ ] `demo/index.html` has a dedicated Wright section mirroring the Orbit section and the existing variant `<select>` lists `wright` — verify: browser-driven at `local:3163` with a screenshot at `evidence/step-3-1-wright-demo.png` showing the Wright section rendering and the picker including Wright.
- [ ] Existing variants are byte-identical and the package surface is unchanged — verify: `git diff --exit-code origin/main -- src/variants src/core.js test/fixtures src/react.js` and `npm run check:variants`.
- [ ] Full maintainer gate is green with the docs/demo/types changes — verify: `npm run verify` exits 0.
