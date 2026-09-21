```yaml
status: planned
branch: feature/wright-variant-palette-system
last-updated: 2026-09-21
next-step: "1.1 Define deterministic Wright material palette families"
artifact-pr: "#5"
initiative: "frank-lloyd-wright-variant"
```

## Phase 1: Palette contract

- [ ] 1.1 Define deeply frozen textile, stained-glass, and concrete/wood semantic palette families in `src/variants/wright.js`, select them deterministically without consuming or reordering geometry draws, and add focused reachability/stability tests — verify: `node --test --test-name-pattern="wright.*palette|wright.*derive" test/wright.test.js`
- [ ] 1.2 Integrate semantic palette roles into Wright painting for light and dark contexts, preserving finite deterministic SVG, layer order, geometry, and current effect semantics — verify: `node --test --test-name-pattern="wright.*paint|wright.*palette|wright.*flash" test/wright.test.js`

## Phase 2: Quantitative constraints

- [ ] 2.1 Enforce unrounded WCAG relative-luminance contrast of at least `3:1` for meaningful structural roles against adjacent colors in light, dark, and current flash/lighten states — verify: `node --test --test-name-pattern="wright.*contrast" test/wright.test.js`
- [ ] 2.2 Enforce a `10%` maximum red painted-area ratio using rendered fill area and stroke footprint across representative seeds, sizes, and pulse states — verify: `node --test --test-name-pattern="wright.*red.*painted area" test/wright.test.js`

## Phase 3: Verification

- [ ] 3.1 Inspect representative seeds for all three palette families in light and dark mode through the unchanged existing demo and capture evidence under this delivery artifact directory — verify: local:3141 shows visually distinct, architectural, non-neon palettes with legible structural layers and restrained red
- [ ] 3.2 Run focused and full repository gates, confirm the final file boundary, and recheck concurrent delivery overlap; no lint command is configured — verify: `node --test test/wright.test.js && npm run verify && test -z "$(git diff --name-only origin/main...HEAD -- . ':(exclude)src/variants/wright.js' ':(exclude)test/wright.test.js')"`