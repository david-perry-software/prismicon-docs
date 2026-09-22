```yaml
status: in-progress
branch: feature/wright-variant-react-demo-docs
last-updated: 2026-09-21
next-step: "2.1"
artifact-pr: "#9"
initiative: "frank-lloyd-wright-variant"
```

## Phase 1: Type declarations and React-facing selection

- [x] 1.1 Audit `src/react.js` and `src/variants/index.js` to confirm `wright` is already registered and resolves through `Prismicon`'s `variant` prop and `listVariants()` with no per-variant code, and record the finding — verify: `node -e "import('./src/index.js').then(m=>console.log(m.listVariants().map(v=>v.id).join(',')))"` prints `wright` and `node --test test/renderer-dispatch.test.js test/react-variant.test.js` exits 0
- [x] 1.2 Add `'wright'` to `BuiltInVariantId`, add `WrightFamily`/`WrightPaletteFamily` unions and a `WrightParams` interface (derived and prepared fields exactly as listed in plan.md `## Approach`), and widen `GlyphHandle.params` to include `WrightParams` in `index.d.ts` — verify: `npx tsc --noEmit --strict --target es2020 --lib es2020,dom index.d.ts`

## Phase 2: README documentation

- [ ] 2.1 Add a `### Wright` subsection under `## Variants` documenting the id/label/spec, the four composition families and hybrid rule, the three palette families with the 3:1 contrast floor and 10% red-accent ceiling, the motion-model table, small-size legibility, and the frozen `wright-geometry-v1` derivation — verify: `grep -n "### Wright" README.md` and `npm run check:variants`
- [ ] 2.2 Update the variant intro sentence, the `listVariants()` example, and the `handle.params` narrowing note so `wright` appears alongside the other three built-ins — verify: `grep -n "wright" README.md` shows the updated example and `npm run verify`

## Phase 3: Demo coverage

- [ ] 3.1 Add a dedicated Wright section to `demo/index.html` mirroring the Orbit section (seed input, `STATES` buttons, one `wright` cell via `mountGlyph(box, seed, { kind: 'agent', size: 72, state, dark, variant: 'wright' })`), and confirm the existing variant `<select>` already lists `wright` — verify: serve `demo/` at `local:3163` (`python3 -m http.server 3163 --directory demo`), drive the page in the browser, and capture `evidence/step-3-1-wright-demo.png`

## Phase 4: Final gate

- [ ] 4.1 Integrate `origin/main` in both halves and recheck concurrent-delivery overlap (open PRs and their changed files) — verify: `git merge origin/main` in the product and companion halves and `gh pr list --state open --json number,headRefName,title`
- [ ] 4.2 Confirm existing variants are byte-identical and run the full maintainer gate — verify: `git diff --exit-code origin/main -- src/variants src/core.js test/fixtures src/react.js && npm run verify`
