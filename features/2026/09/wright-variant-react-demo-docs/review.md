# Review: wright-variant-react-demo-docs

Verdict: approve

**Round 2 of 3** (re-review, 2026-09-22) against product `feature/wright-variant-react-demo-docs`
@ `4f0d243` (code PR [#24](https://github.com/david-perry-software/prismicon/pull/24), draft,
`mergeStateStatus: CLEAN`) and companion `feature/wright-variant-react-demo-docs` @ `49ce8cd`
(artifact PR [#9](https://github.com/david-perry-software/prismicon-docs/pull/9), draft).
`origin/main` is an ancestor of `HEAD` in both halves; both halves were in sync with their
remote and the companion was `dirty: false`, `ahead: 0` at review start. `doctor --for
review-feature` → `ok`.

**What changed since round 1.** Round 1 (product `8f0be54`) returned `request-changes` for two
factual errors in the new `README.md` `### Wright` section and added roadmap step 2.3 for them.
The Builder executed 2.3 with a single product commit `4f0d243` `docs(readme): correct wright
aria-label example and working-mount note` touching only `README.md` (+6/−4): the `maya`
aria-label example now reads `5 planes`, and the motion-model sentence is qualified for the
`working` mount case. `demo/index.html` and `index.d.ts` are unchanged since round 1, and the
round-1 roadmap repair to the step 3.1 verify command stands. Both fixes are verified below
against the source; every gate was re-run by the reviewer and is green.

Skills consulted: modern-javascript-patterns (demo `<script type="module">` block —
const/let discipline, arrow callbacks, `replaceChildren`, mirror of the existing Orbit block;
no findings); vercel-react-best-practices (React binding inspected — `src/react.js` is
byte-identical to `origin/main` and resolves `wright` through `ctx.registry.resolve`, so no
React-specific rules are engaged).

## Acceptance checklist results

1. **`index.d.ts` lists `'wright'` in `BuiltInVariantId`, defines `WrightParams` (+ `WrightFamily`/`WrightPaletteFamily`), `GlyphHandle.params` widened** — **pass.**
   `git diff origin/main...HEAD -- index.d.ts`: `BuiltInVariantId = 'polyhedron' | 'ncube' | 'orbit' | 'wright' | \`ncube-${number}\`` (L37); `WrightFamily` (L103), `WrightPaletteFamily` (L106), `WrightParams` (L109–160) with all 23 `deriveWright` keys and the 6 optional `readonly` `prepareWright` fields (`strokeWidth`, `lightStroke`, `small`, `sweepDir?: 1 | -1`, `panelPhase`, `illumSpeed`); `readonly params: GlyphParams | NcubeParams | OrbitParams | WrightParams` (L182). Unchanged since round 1.
   Reviewer: `npx tsc --noEmit --strict --target es2020 --lib es2020,dom index.d.ts` → exit 0.

2. **`src/react.js` and `src/variants/index.js` unchanged; `wright` resolves through `Prismicon` and `listVariants()`** — **pass.**
   Reviewer: `git diff --exit-code origin/main -- src/variants src/core.js test/fixtures src/react.js src/authoring.js src/index.js` → exit 0.
   `node -e "import('./src/index.js').then(m=>{…includes('wright')…})"` → `wright present: polyhedron,ncube,ncube-3,ncube-4,ncube-5,ncube-6,orbit,wright`, exit 0.
   `node --test test/renderer-dispatch.test.js test/react-variant.test.js` → 30 tests / 30 pass / 0 fail. [evidence/step-1-1-audit.md](evidence/step-1-1-audit.md) records the same finding.

3. **`README.md` documents the `wright` variant (id/label/spec, composition families + hybrids, palette families, motion model, small-size legibility, determinism) and the `listVariants()` example includes `wright`** — **pass** (round-1 defects resolved).
   `grep -n "### Wright" README.md` → L292. Intro sentence (L110), `listVariants()` example (L124: `{ id: 'wright', label: 'Wright Scaffold', spec: 'wright-geometry-v1' }`), `handle.params` narrowing note (L135–138) all present. Section covers id/label/spec (L301), composition families + hybrid compatibility (L302–307), palette families with the 3:1 floor and 10% ceiling (L308–314), small-size legibility (L315–317), motion model table (L319–334), traits-without-draws and reduced motion (L336–344), the frozen 12-draw derivation spec (L346–371).
   **Round-1 Finding 1 resolved:** L299 reads `maya: prairie Wright composition with art-glass detail, 5 planes`; reviewer `node -e "import('./src/variants/wright.js').then(m=>console.log(m.describeWright(m.deriveWright('maya'))))"` → `prairie Wright composition with art-glass detail, 5 planes`; `grep -c "3 planes" README.md` → `0`; `grep -n "5 planes" README.md` → only L299, inside the Wright section (L292–371).
   **Round-1 Finding 2 resolved:** L319–323 now read "Every state except `working` starts from the seed's rest pose (`illuminate: -1`, `panelPulse: 0`, `settle: 0`), so the first mounted frame equals the static portrait; a glyph mounted in `working` starts on the first working frame (`poseWright(params, 'working')`) so it lands on the motion trajectory without a jump". Checked against product `src/variants/wright.js` L371–380 (`poseWright` returns the t = 0 working pose for `'working'`, rest otherwise) and `src/core.js` L101–102 (`initialPose = !eng.reduced && initial === 'working' ? variant.pose(p, 'working') : rest`). The `eng.reduced` branch is covered by the section's own "Reduced motion" bullet (L341–344: "a mounted Wright composition equals its static markup in every state"), so the sentence is accurate as written.
   The remaining numeric and structural claims were cross-checked in round 1 and the section text outside L299 and L319–323 is unchanged (`git show 4f0d243 -- README.md`). `npm run check:variants` → all five checks ✓.

4. **`demo/index.html` has a dedicated Wright section mirroring Orbit; the variant `<select>` lists `wright`** — **pass.**
   `demo/index.html` is unchanged since round 1 (L63–70 markup; L227–256 `mountWright`, `STATES` buttons, seed listener — a line-for-line mirror of the Orbit block with `variant: 'wright'`). Reviewer re-served the product root on `local:3163` (`python3 -m http.server 3163 --bind 127.0.0.1 --directory <product root>`), reloaded `http://localhost:3163/demo/` in the browser and drove it: `<h2>Wright</h2>` present; `#wright-seed` = `maya`; `#wright-buttons` = the 9 `STATES`; `#wright svg` aria-label `maya: prairie Wright composition with art-glass detail, 5 planes, idle`; clicking `working` → `…, 5 planes, working`; `<select>` options `polyhedron, ncube, ncube-3…6, orbit, wright (Wright Scaffold), square`; selecting `wright` re-mounts the top agent as `demo-agent: textile-block Wright composition, 5 planes, working`. The page renders identically to round 1, so the round-1 reviewer screenshots stand as evidence: [evidence/review-step-3-1-wright-demo.png](evidence/review-step-3-1-wright-demo.png), [evidence/review-step-3-1-wright-picker.png](evidence/review-step-3-1-wright-picker.png); Builder evidence [evidence/step-3-1-wright-demo.png](evidence/step-3-1-wright-demo.png), [evidence/step-3-1-wright-picker.png](evidence/step-3-1-wright-picker.png) agree. Server stopped and port 3163 released afterwards.

5. **Existing variants byte-identical, package surface unchanged** — **pass.**
   `git diff --exit-code origin/main -- src/variants src/core.js test/fixtures src/react.js` → exit 0 (the wider set in item 2 also exit 0). `npm run check:variants` → `✓ contract ✓ exports ✓ types ✓ pack ✓ goldens`. Product diff `origin/main...HEAD` touches only `README.md` (+91), `demo/index.html` (+39), `index.d.ts` (+77/−11): 196 insertions, 11 deletions, 3 files.

6. **`npm run verify` exits 0** — **pass.** Reviewer run at `4f0d243`: 191 tests / 191 pass / 0 fail; `check:variants` all five ✓; exit 0.

**Lint gate (§5):** AGENTS.md declares `Lint: none` and `package.json` has no lint script; plan.md `## Research` records the baseline as `none configured`, exit `N/A`, findings none. Nothing to compare; the gate for the touched files is `npm run verify`, which is green.

## Plan vs implementation

- Scope held exactly: the three files named in plan.md `## Approach` are the only product changes across all six branch commits; every file under `## Out of scope` is byte-identical to `origin/main`. No new exports, options, or demo module.
- `WrightParams` matches the field list pinned in plan.md `## Approach` item 1 verbatim, including `emphasis: 'vertical' | 'horizontal'` and `sweepDir?: 1 | -1`.
- Decision 5 (leave `label: 'Wright Scaffold'` untouched — `src/variants/wright.js` L526) honored; the label surfaces in the README example and the demo picker as documented.
- The step 2.3 fix commit `4f0d243` is scoped precisely to the two sentences the round-1 review named; no collateral README edits.
- Documented deviation carried from round 1: the README Wright section is deeper than "mirror the n-cube/orbit sections" (full derivation spec + motion formulas). Consistent with the Orbit section's depth and now fully accurate.
- Roadmap step 3.1's original `--directory demo` verify command was repaired in round 1; the Builder's evidence and both reviewer runs served the repository root.

## Roadmap audit

All eight ticked boxes spot-checked against the code — every tick is genuine:

- 1.1 — [evidence/step-1-1-audit.md](evidence/step-1-1-audit.md); reviewer re-ran both verify commands (item 2).
- 1.2 — `index.d.ts` diff + `tsc` exit 0.
- 2.1 / 2.2 — `README.md` L110, L124, L135–138, L292–371.
- 2.3 (added 2026-09-22) — product `4f0d243`; `describeWright(deriveWright('maya'))` → `…, 5 planes`; `grep -c "3 planes"` → 0; `grep -n "5 planes"` → L299 inside the section; `npm run verify` exit 0. All four verify clauses pass.
- 3.1 — `demo/index.html` L63–70, L227–256; browser re-driven on `local:3163`; evidence PNGs present.
- 4.1 — `origin/main` is an ancestor of `HEAD` in both halves; `gh pr list --state open` in both repositories shows only #24 / #9 (no overlapping delivery); PR #24 `mergeStateStatus: CLEAN`.
- 4.2 — byte-identical diff exit 0 and `npm run verify` exit 0 re-run by reviewer.

Header state is consistent: `status: in-review`, `next-step: ""`, `last-updated: 2026-09-22`, 8/8 ticked. No `(manual)` or `(manual, post-ship)` steps exist. **No repairs needed this round**; no falsely ticked boxes remain.

## Findings

No findings above minor severity remain.

1. **Resolved (was Moderate) — README aria-label example.** `3 planes` → `5 planes` at `README.md` L299; verified against `describeWright(deriveWright('maya'))`.
2. **Resolved (was Minor) — motion-model claim for the `working` mount.** `README.md` L319–323 now state the `working` exception and name `poseWright(params, 'working')`; verified against `src/variants/wright.js` L371–380 and `src/core.js` L101–102.
3. **Resolved (was Minor) — step 3.1 verify command.** Repaired in the roadmap in round 1; unchanged and correct.
4. **Note (no action) — `label: 'Wright Scaffold'`.** Surfaced in the demo picker, `listVariants()`, and the README example exactly as plan.md Decision 5 accepted. Carried under Follow-ups.

No security-relevant surface is touched: the diff is type declarations, Markdown, and a static demo page mirroring the existing Orbit block (the only input is the seed text field already used by the other sections; `mountGlyph` output is variant-painted SVG, same as Orbit).

## Follow-ups

- Rename the registered Wright label from `'Wright Scaffold'` to a shipping name (e.g. `'Wright'`). Requires touching `src/variants/wright.js` L526 and updating the README `listVariants()` example (L124) plus any test that pins `listVariants()` labels, so it is its own issue rather than part of this docs feature (plan.md Decision 5).
- Optionally document the `pose('working')` convention once in the Custom-variants `pose` hook row of the README (L400): a variant *may* return the first working frame (rather than rest) for `'working'` so a glyph mounted in `working` lands on its trajectory without a jump. That would make the Wright-section caveat (L319–323) an instance of a documented contract rather than a per-variant note.
