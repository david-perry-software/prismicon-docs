```yaml
status: planned
branch: feature/wright-variant-geometry-grammar
last-updated: 2026-09-20
next-step: "1.1 Install dependencies and establish a clean verification baseline"
initiative: "frank-lloyd-wright-variant"
```

## Phase 1: Baseline and grammar contract

- [ ] 1.1 Install dependencies and establish a clean verification baseline before source edits; verify: `npm ci && npm run verify` exits 0
- [ ] 1.2 Define and test the Wright geometry spec, fixed PRNG draw order, four dominant families, and controlled-hybrid compatibility rules in `test/wright.test.js`; verify: `node --test test/wright.test.js` exits 0 with named coverage for Prairie, art-glass, textile-block, Usonian, and hybrid cases

## Phase 2: Hierarchical geometry

- [ ] 2.1 Implement deterministic dominant-family and bounded secondary-family derivation in `src/variants/wright.js`, including the new spec version and descriptive anatomy; verify: `node --test test/wright.test.js` exits 0 with normalized repeated derivations deep-equal and all family cases reachable
- [ ] 2.2 Replace scaffold frame/bands geometry with deeply frozen primary-mass, horizontal-plane, grid/decorative, and accent layers; verify: `node --test test/wright.test.js` exits 0 with layer-presence, quota, positive-area, finiteness, and deep-freeze assertions
- [ ] 2.3 Render the semantic layers in stable hierarchy order while preserving existing hue/effects and pose transforms; verify: `node --test test/wright.test.js` exits 0 with repeated SVG byte equality, expected layer markup, and retained motion/flash assertions
- [ ] 2.4 Enforce stroke-aware 100-by-100 viewBox bounds at sizes 24, 64, and 72 without adding aesthetic detail reduction; verify: focused `test/wright.test.js` size-matrix assertions pass and rendered markup contains neither `NaN` nor `Infinity`

## Phase 3: Recognition and stability

- [ ] 3.1 Review representative pure-family and controlled-hybrid seeds in light and dark modes at sizes 24, 64, and 72 through the existing demo, and capture the comparison evidence; verify: local:3172 — a temporary static server renders each matrix entry unclipped, with immediate Wright-family hierarchy visible and size 24 assessed only for valid bounded output
- [ ] 3.2 Confirm no palette-system, motion-event, legibility-tuning, registry, demo, React, type, or public API scope leaked into the implementation; verify: `git diff --name-only origin/main...HEAD` lists only `src/variants/wright.js` and `test/wright.test.js` in the product repository
- [ ] 3.3 Run the complete no-lint replacement gate after integrating `origin/main`; verify: `node -e "const p=require('./package.json'); if ('lint' in p.scripts) process.exit(1)" && npm run verify` exits 0, including contract, exports, types, pack, and unchanged existing goldens