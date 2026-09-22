# Review: wright-variant-react-demo-docs

Verdict: request-changes

Reviewed 2026-09-22 against product `feature/wright-variant-react-demo-docs` @ `8f0be54`
(code PR [#24](https://github.com/david-perry-software/prismicon/pull/24), draft) and
companion `feature/wright-variant-react-demo-docs` @ `b1df296` (artifact PR
[#9](https://github.com/david-perry-software/prismicon-docs/pull/9), draft). `origin/main`
is an ancestor of `HEAD` in both halves; both halves were in sync with their remote and the
companion was `dirty: false`, `ahead: 0` at review start.

Skills consulted: modern-javascript-patterns (demo `<script type="module">` code — const/let
discipline, arrow callbacks, mirror of the existing Orbit block); vercel-react-best-practices
(the React binding was inspected: `src/react.js` is unchanged and resolves `wright` through
`ctx.registry.resolve`, so no React-specific findings apply).

The implementation is tight and does exactly what the plan scopes: three product files
(`index.d.ts`, `README.md`, `demo/index.html`), zero runtime changes, every existing variant
byte-identical, full gate green. The verdict is `request-changes` solely because the README
section — the primary deliverable of a documentation feature — contains two verifiably false
statements about the variant it documents (Findings 1–2). Both are one-line corrections.

## Acceptance checklist results

1. **`index.d.ts` lists `'wright'` in `BuiltInVariantId`, defines `WrightParams` (+ `WrightFamily`/`WrightPaletteFamily`), `GlyphHandle.params` widened** — **pass.**
   `git diff origin/main...HEAD -- index.d.ts`: `BuiltInVariantId = 'polyhedron' | 'ncube' | 'orbit' | 'wright' | \`ncube-${number}\`` (L37); `WrightFamily` (L103), `WrightPaletteFamily` (L106), `WrightParams` (L109–160); `readonly params: GlyphParams | NcubeParams | OrbitParams | WrightParams` (L182).
   Field audit against `src/variants/wright.js`: all 23 `deriveWright` output keys (`spec … hue2`, L223–246) and all 6 `prepareWright` additions (`strokeWidth`, `lightStroke`, `small`, `sweepDir`, `panelPhase`, `illumSpeed`, L271–280) are present, prepared fields optional + `readonly`, mirroring `OrbitParams`.
   `npx tsc --noEmit --strict --target es2020 --lib es2020,dom index.d.ts` → exit 0 (run by reviewer).

2. **`src/react.js` and `src/variants/index.js` unchanged; `wright` resolves through `Prismicon` and `listVariants()`** — **pass.**
   `git diff --exit-code origin/main -- src/variants src/core.js test/fixtures src/react.js src/authoring.js src/index.js` → exit 0.
   `node -e "import('./src/index.js').then(m=>console.log(m.listVariants().map(v=>v.id).join(',')))"` → `polyhedron,ncube,ncube-3,ncube-4,ncube-5,ncube-6,orbit,wright`.
   `node --test test/renderer-dispatch.test.js test/react-variant.test.js` → 30 pass / 0 fail. `evidence/step-1-1-audit.md` records the same.

3. **`README.md` documents the `wright` variant (id/label/spec, composition families + hybrids, palette families, motion model, small-size legibility, determinism) and the `listVariants()` example includes `wright`** — **pass with defects (see Findings 1–2).**
   `grep -n "### Wright" README.md` → L292. Intro sentence (L110), `listVariants()` example (L124: `{ id: 'wright', label: 'Wright Scaffold', spec: 'wright-geometry-v1' }`), and the `handle.params` narrowing note (L135–138) are updated. Every required topic is present.
   Reviewer cross-checked the section's numeric and structural claims against `src/variants/wright.js`: the 12-draw `WRIGHT_DRAW_ORDER` and every range (`52+⌊r·17⌋`, `38+⌊r·17⌋`, `−8+⌊r·17⌋`, `3+⌊r·3⌋`, `6+⌊r·5⌋`, `2+⌊r·4⌋`×2, `⌊r·3⌋`, `⌊r·4⌋`) match L203–214; `phase`, `paletteFamily = ⌊hash/2^40⌋ % 3`, `hue`/`hue2`, `lineCount`/`inset`/`horizon`/`cantilever`/`emphasis` match L215–245; hybrid compatibility table matches L26–31; `WRIGHT_CONTRAST_MIN = 3`, `WRIGHT_RED_AREA_CEILING = 0.10`, `WRIGHT_SMALL_SIZE = 28`, 2×2 grid / 2 decorations match L44–53; motion trait bit ranges 42/43–48/49–52 match L263–269; working/waiting/thinking/sleeping/sending/receiving/settling formulas match L384–456. **Two statements do not match the code** — Findings 1 and 2. `npm run check:variants` → all five checks ✓.

4. **`demo/index.html` has a dedicated Wright section mirroring Orbit; the variant `<select>` lists `wright`** — **pass.**
   Reviewer served the product root on `local:3163` and drove `http://localhost:3163/demo/` in the browser: `<h2>Wright</h2>` present; `#wright-seed` = `maya`; `#wright-buttons` = the 9 `STATES`; `#wright svg` aria-label `maya: prairie Wright composition with art-glass detail, 5 planes, idle`; `<select>` options include `wright` (label `Wright Scaffold`); selecting it re-mounts the top agent as `demo-agent: textile-block Wright composition, 5 planes, …`; clicking `working` in the Wright section sets the aria-label to `…, working`. Reviewer screenshots: [evidence/review-step-3-1-wright-demo.png](evidence/review-step-3-1-wright-demo.png), [evidence/review-step-3-1-wright-picker.png](evidence/review-step-3-1-wright-picker.png). Builder evidence [evidence/step-3-1-wright-demo.png](evidence/step-3-1-wright-demo.png) and [evidence/step-3-1-wright-picker.png](evidence/step-3-1-wright-picker.png) agree.

5. **Existing variants byte-identical, package surface unchanged** — **pass.**
   `git diff --exit-code origin/main -- src/variants src/core.js test/fixtures src/react.js` → exit 0; `npm run check:variants` → `✓ contract ✓ exports ✓ types ✓ pack ✓ goldens`. Product diff touches only `README.md`, `demo/index.html`, `index.d.ts` (194 insertions, 11 deletions).

6. **`npm run verify` exits 0** — **pass.** Reviewer run: 191 tests / 191 pass / 0 fail, all `check:variants` gates ✓, exit 0.

## Plan vs implementation

- Scope held exactly: the three files named in plan.md `## Approach` are the only product changes; every file listed under `## Out of scope` is untouched (verified by `git diff --exit-code` above). No new exports, options, or demo module.
- `WrightParams` matches the field list pinned in plan.md `## Approach` item 1 verbatim, including `emphasis: 'vertical' | 'horizontal'` and `sweepDir?: 1 | -1`.
- Decision 5 (leave `label: 'Wright Scaffold'` untouched) was honored; the label therefore surfaces in the README example and the demo picker as documented.
- Minor deviation, not a defect: the README Wright section goes further than the "mirror the n-cube/orbit sections" brief by including a full numbered derivation spec and the motion formulas. This is consistent with the Orbit section's depth and is welcome — but it is exactly what makes the two inaccuracies below matter.
- Deviation in the step 3.1 verify wording: the roadmap said `python3 -m http.server 3163 --directory demo`, which cannot serve `demo/index.html`'s `../src/index.js` import. The Builder actually served the repository root and drove `/demo/` (visible in the evidence URL); repaired in the roadmap audit below.

## Roadmap audit

Spot-checked all seven ticked boxes against the code — every tick is genuine:

- 1.1 — `evidence/step-1-1-audit.md` plus reviewer re-run of both verify commands (above).
- 1.2 — `index.d.ts` diff + `tsc` exit 0.
- 2.1 / 2.2 — `README.md` L110, L124, L135–138, L292–370.
- 3.1 — `demo/index.html` L63–70 (markup) and L227–256 (`mountWright`, buttons, seed listener); evidence PNGs present.
- 4.1 — `origin/main` is an ancestor of `HEAD` in both halves; `gh pr list --state open` shows only #24 / #9.
- 4.2 — byte-identical diff exit 0 and `npm run verify` exit 0 re-run by reviewer.

Repairs made:

- Added unticked step **2.3 (added 2026-09-22)** for the two README corrections in Findings 1–2, with a CLI-executable verify.
- Corrected the **3.1 verify command** to serve the repository root and open `/demo/` (the form that actually works), annotating that the ticked check was performed that way.
- Set `next-step: "2.3"` and `last-updated: 2026-09-22`. `status` left at `in-review` for the Builder fix handoff.

No falsely ticked boxes remain. No `(manual)` or `(manual, post-ship)` steps exist.

## Findings

1. **Moderate — README aria-label example is wrong for its own seed.** Product `README.md` Wright section, paragraph 1: "`maya: prairie Wright composition with art-glass detail, 3 planes`". `deriveWright('maya').planeCount` is **5**, and `describeWright` returns `prairie Wright composition with art-glass detail, 5 planes` (reviewer: `node -e "import('./src/variants/wright.js').then(m=>console.log(m.describeWright(m.deriveWright('maya'))))"`; the live demo aria-label shows the same). The family/hybrid part of the example is correct. Fix: `3 planes` → `5 planes`. Product-only, one token.

2. **Minor — motion-model claim is false for the `working` mount case.** README Wright section, "Motion model": "Every state starts from the seed's rest pose (`illuminate: -1`, `panelPulse: 0`, `settle: 0`), so the first mounted frame equals the static portrait". `poseWright(params, 'working')` (product `src/variants/wright.js` L371–380) deliberately returns the t = 0 working-frame pose, not rest, and `mountGlyph` (product `src/core.js` L102) uses `variant.pose(p, 'working')` when mounted with `state: 'working'` — so a Wright glyph mounted in `working` does **not** start on the static portrait (this differs from Orbit, whose `poseOrbit` is state-independent and for which the identical sentence is true). Fix: qualify the sentence, e.g. "Every state except `working` starts from the seed's rest pose (…); a glyph mounted in `working` starts on the first working frame so there is no jump onto the trajectory."

3. **Minor — `--directory demo` verify command could not have served the page.** Roadmap step 3.1 as planned. Repaired in place (see Roadmap audit); no product change needed.

4. **Note (no action) — `label: 'Wright Scaffold'`.** Surfaced in the demo picker and README exactly as Decision 5 accepted. Listed under Follow-ups.

No security-relevant surface is touched: the diff is type declarations, Markdown, and a static demo page that mirrors the existing Orbit block (no new inputs beyond the seed text field already used by the other sections; `mountGlyph` output is variant-painted SVG, same as Orbit).

## Follow-ups

- Rename the registered Wright label from `'Wright Scaffold'` to a shipping name (e.g. `'Wright'`). Requires touching `src/variants/wright.js` L526 and regenerating any golden/test that pins `listVariants()` labels, so it is its own issue rather than part of this docs feature (plan.md Decision 5).
- Consider aligning `poseWright('working')` documentation across the initiative: the `wright-variant-motion-events` plan introduced the working-pose exception; a short note in the Custom-variants `pose` hook row of the README (L398) that a variant *may* return a non-rest pose for `working` would make Finding 2 a documented convention rather than a per-variant caveat.
