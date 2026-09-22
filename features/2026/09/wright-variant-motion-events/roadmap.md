```yaml
status: planned
branch: feature/wright-variant-motion-events
last-updated: 2026-09-21
next-step: "1.1"
initiative: "frank-lloyd-wright-variant"
```

## Phase 1: Motion language

- [ ] 1.1 Replace the placeholder `{ lift, shear, pulse }` pose and its generic `poseWright`/`animateWright` motion in `src/variants/wright.js` with the frozen Wright motion pose (`{ illuminate, panelPulse, settle }`) and per-state motifs: working = illumination sweep + panel pulse, waiting = slow illumination drift, thinking = sequential panel cascade, sleeping = slow dim breath, sending/receiving = transient illumination burst, settling = structural settle/reconstruct. Keep `poseWright(params, 'idle')` the all-neutral rest pose so static rendering stays byte-identical — verify: `node --test --test-name-pattern="wright" test/wright.test.js`
- [ ] 1.2 Derive tunable motion traits (sweep direction, phase offsets, speed bases) in `prepareWright` from disjoint bit-ranges of `params.hash` with no new PRNG draws, leaving `deriveWright` output and `WRIGHT_SPEC_VERSION` unchanged — verify: `node --test --test-name-pattern="wright.*derive|wright.*motion" test/wright.test.js`

## Phase 2: Painting and invariants

- [ ] 2.1 Extend `paintWright` to interpret the new pose (bounded illumination lightening, per-module panel-pulse modulation, structural settle offset) while preserving layer order, `data-wright-layer` attributes, finite SVG, and byte-identical rest rendering — verify: `node --test --test-name-pattern="wright.*paint|wright.*palette" test/wright.test.js`
- [ ] 2.2 Enforce the invariants at every animated frame: geometry inside the 100×100 viewBox stroke bounds, red accent ≤10% painted area, ≥3:1 contrast on final emitted colors, and reduced-motion output fully static — verify: `node --test test/wright.test.js`

## Phase 3: Determinism and golden

- [ ] 3.1 Add event-frame determinism, per-state distinctness, and settle-to-rest identity tests over the full `PROBE_STATES` list, and update the existing scaffold animate test to the new pose vocabulary — verify: `node --test test/wright.test.js`
- [ ] 3.2 Regenerate goldens and commit only the Wright `mounted` change, keeping the Wright `static` section and all non-Wright fixtures byte-identical to `origin/main` — verify: `node scripts/generate-golden.mjs && git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json && node --input-type=commonjs -e "const fs=require('fs'),cp=require('child_process');const n=JSON.parse(fs.readFileSync('test/fixtures/golden-wright-v1.json','utf8'));const w=JSON.parse(cp.execSync('git show origin/main:test/fixtures/golden-wright-v1.json').toString());if(JSON.stringify(n.static)!==JSON.stringify(w.static)){console.error('wright static section changed');process.exit(1)}console.log('wright static section unchanged')"`

## Phase 4: Visual verification

- [ ] 4.1 Drive `demo/index.html` at `local:3108`, select the Wright variant, sweep all states, and capture frame screenshots under this directory's `evidence/`; confirm motion is restrained, legible, architecture-consistent, and the idle portrait matches the pre-change portrait — verify: local:3108 shows distinct restrained Wright motion per state with an unchanged idle portrait (evidence captured)

## Phase 5: Final gate

- [ ] 5.1 Integrate `origin/main`, recheck concurrent delivery overlap, and confirm the final file boundary and full gates — verify: `node --test test/wright.test.js && npm run verify && test -z "$(git diff --name-only origin/main...HEAD -- . ':(exclude)src/variants/wright.js' ':(exclude)test/wright.test.js' ':(exclude)test/fixtures/golden-wright-v1.json')"`
