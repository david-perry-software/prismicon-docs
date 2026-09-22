```yaml
status: in-review
branch: feature/wright-variant-react-demo-docs
last-updated: 2026-09-22
next-step: "2.3"
artifact-pr: "#9"
initiative: "frank-lloyd-wright-variant"
```

## Phase 1: Type declarations and React-facing selection

- [x] 1.1 Audit `src/react.js` and `src/variants/index.js` to confirm `wright` is already registered and resolves through `Prismicon`'s `variant` prop and `listVariants()` with no per-variant code, and record the finding — verify: `node -e "import('./src/index.js').then(m=>console.log(m.listVariants().map(v=>v.id).join(',')))"` prints `wright` and `node --test test/renderer-dispatch.test.js test/react-variant.test.js` exits 0
- [x] 1.2 Add `'wright'` to `BuiltInVariantId`, add `WrightFamily`/`WrightPaletteFamily` unions and a `WrightParams` interface (derived and prepared fields exactly as listed in plan.md `## Approach`), and widen `GlyphHandle.params` to include `WrightParams` in `index.d.ts` — verify: `npx tsc --noEmit --strict --target es2020 --lib es2020,dom index.d.ts`

## Phase 2: README documentation

- [x] 2.1 Add a `### Wright` subsection under `## Variants` documenting the id/label/spec, the four composition families and hybrid rule, the three palette families with the 3:1 contrast floor and 10% red-accent ceiling, the motion-model table, small-size legibility, and the frozen `wright-geometry-v1` derivation — verify: `grep -n "### Wright" README.md` and `npm run check:variants`
- [x] 2.2 Update the variant intro sentence, the `listVariants()` example, and the `handle.params` narrowing note so `wright` appears alongside the other three built-ins — verify: `grep -n "wright" README.md` shows the updated example and `npm run verify`
- [ ] 2.3 Correct the two factual inaccuracies in the `### Wright` README section found in review: (a) the aria-label example for `maya` must read `5 planes` (not `3 planes`) — `deriveWright('maya').planeCount` is 5; (b) the motion-model sentence "Every state starts from the seed's rest pose … so the first mounted frame equals the static portrait" must be qualified, because `poseWright(params, 'working')` deliberately returns the first working-frame pose (not rest) and `mountGlyph` uses `variant.pose(p, 'working')` when mounted with `state: 'working'` — so only non-`working` mounts start on the static portrait (added 2026-09-22) — verify: `node -e "import('./src/variants/wright.js').then(m=>console.log(m.describeWright(m.deriveWright('maya'))))"` prints `prairie Wright composition with art-glass detail, 5 planes` and `grep -n "5 planes" README.md` matches inside the Wright section; `grep -c "3 planes" README.md` prints 0; and `npm run verify`

## Phase 3: Demo coverage

- [x] 3.1 Add a dedicated Wright section to `demo/index.html` mirroring the Orbit section (seed input, `STATES` buttons, one `wright` cell via `mountGlyph(box, seed, { kind: 'agent', size: 72, state, dark, variant: 'wright' })`), and confirm the existing variant `<select>` already lists `wright` — verify: serve the repository root at `local:3163` (`python3 -m http.server 3163` from the product worktree root — `demo/index.html` imports `../src/index.js`, so `--directory demo` cannot serve it) and open `http://localhost:3163/demo/`, drive the page in the browser, and capture `evidence/step-3-1-wright-demo.png` (verify command corrected in review 2026-09-22; the ticked check was performed against `/demo/` served from the root, as the evidence URL shows)

## Phase 4: Final gate

- [x] 4.1 Integrate `origin/main` in both halves and recheck concurrent-delivery overlap (open PRs and their changed files) — verify: `git merge origin/main` in the product and companion halves and `gh pr list --state open --json number,headRefName,title`
- [x] 4.2 Confirm existing variants are byte-identical and run the full maintainer gate — verify: `git diff --exit-code origin/main -- src/variants src/core.js test/fixtures src/react.js && npm run verify`
