```yaml
status: in-progress
branch: feature/wright-variant-regression-suite
last-updated: 2026-09-21
next-step: "4.1 integrate origin/main and run the full maintainer gate"
artifact-pr: "#8"
initiative: "frank-lloyd-wright-variant"
```

## Phase 1: Performance measurement

- [x] 1.1 Add `scripts/measure-wright.mjs` mirroring `scripts/measure-orbit.mjs`: static SVG byte size and median `paint()` time over 200 calls across the five generic seeds plus the representative Wright seeds (all four geometry families, all three palette families, one verified hybrid), then an animated engine-loop table (working frames at 30 fps, a send burst, settling to rest) reporting median/p95 `animate + paint` ms and the settling frame count, gated `frame gate: pass|fail` — verify: `node scripts/measure-wright.mjs`
- [x] 1.2 Run the measure script and record its output to `evidence/wright-benchmark.txt` in this directory; confirm the frame gate reports `pass` for every representative seed — verify: `node scripts/measure-wright.mjs | tee evidence/wright-benchmark.txt` and `grep -q 'frame gate: pass' evidence/wright-benchmark.txt`

## Phase 2: Representative golden seed sets

- [x] 2.1 Extend `test/helpers/golden.js` with a Wright-only seed extension so `captureGolden({ variant: 'wright' })` additionally captures the representative static seeds and mounted scenarios (one per geometry family, one per palette family, one concrete hybrid with `deriveWright(seed).secondaryFamily !== null`), leaving the shared `STATIC_SEEDS`/`MOUNTED_SCENARIOS` and every other family untouched — verify: `node --test test/golden-v1.test.js test/golden-ncube.test.js test/golden-orbit.test.js test/wright.test.js`
- [x] 2.2 Add `test/golden-wright.test.js` mirroring `test/golden-ncube.test.js`: assert the fixture keys equal the registered Wright ids and that every static key and mounted frame hash re-captures byte-identically — verify: `node --test test/golden-wright.test.js`
- [x] 2.3 Regenerate goldens and confirm only `test/fixtures/golden-wright-v1.json` changes; the default, n-cube, and orbit fixtures stay byte-identical to `origin/main` — verify: `node scripts/generate-golden.mjs && git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json`

## Phase 3: Event-frame stability

- [x] 3.1 Add a dedicated event-frame stability test in `test/wright.test.js`: for the representative Wright seeds and every `PROBE_STATES` state, run `animateWright` for a bounded number of frames, assert the frame sequence is deterministic across two runs, settles to the exact rest pose by identity within a bounded frame count, and paints finite (`NaN`/`Infinity` absent) geometry inside `WRIGHT_VIEWBOX_BOUNDS` — verify: `node --test --test-name-pattern="wright" test/wright.test.js`
- [x] 3.2 Confirm the representative Wright seeds are included in the mounted golden scenarios so the golden gate locks their event-frame hashes — verify: `node --test test/golden-wright.test.js && npm run check:variants`

## Phase 4: Final gate

- [ ] 4.1 Integrate `origin/main` (product and companion), recheck concurrent-delivery overlap, and run the full maintainer gate — verify: `npm run verify`
- [ ] 4.2 Confirm CI parity and non-Wright fixture isolation one last time — verify: `npm test && npm run check:variants && git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json`
