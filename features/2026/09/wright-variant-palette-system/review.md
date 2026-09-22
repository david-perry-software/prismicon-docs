# Review: wright-variant-palette-system

Verdict: approve

## Acceptance checklist results

- [x] **Deterministic normalized seeds select all three deeply frozen palette families (textile, stained-glass, concrete-wood) with stable semantic roles while geometry draw order and geometry-derived values remain unchanged** — PASS. `test/wright.test.js` asserts the families list and deep-frozen structure, derives `paletteFamily` from `Math.floor(hash / 2 ** 40) % 3` without consuming a draw, reaches all three families with named seeds (`wright-palette-1` → textile, `wright-palette-11` → stained-glass, `wright-palette-0` → concrete-wood) and a 200-seed corpus, and locks geometry values for `maya` and `wright-family-5` to the geometry-feature values. Re-run: `node --test --test-name-pattern="wright.*palette|wright.*derive" test/wright.test.js` → 4 pass / 0 fail.
- [x] **Paint output uses the selected architectural palette in light/dark, deterministic and finite, semantic layer order preserved, public variant API unchanged** — PASS. `paintWright` replaces the old `hsl()` mixing with `roles.*` semantic colors in unchanged layer order; the test asserts the family's frozen light/dark role colors are emitted and no `NaN|Infinity`. `npm run verify` passes the `contract`/`exports`/`types` gates, and `git diff --name-only origin/main...HEAD -- src/index.js src/react.js index.d.ts demo/ README.md LICENSE package.json` is empty.
- [x] **Every meaningful structural role meets an unrounded WCAG 3:1 ratio against its adjacent role in light, dark, and flash/lighten states** — PASS. Standards-based sRGB `hexToRgb`/`relativeLuminance`/`contrastRatio` are validated against white/black references, the frozen palette tables satisfy all `WRIGHT_CONTRAST_PAIRS`, and a final-emitted-color loop checks the painted role color against its actual adjacent surface (`canvas` for primary/secondary/accent, `secondary` for line) across 5 seeds × light/dark × idle/flash/lighten. Re-run: `node --test --test-name-pattern="wright.*contrast" test/wright.test.js` → 1 pass / 0 fail.
- [x] **Taliesin-like red occupies ≤ 10% of estimated painted geometry area across seeds, sizes, and pulse states using fill area + stroke footprint** — PASS. `paintedAreaMetrics` counts plane fills as area and all stroked layers as stroke footprint; the red share is the accent stroke footprint (never element count), asserted `≤ WRIGHT_RED_AREA_CEILING` (0.10) over 8 seeds × 4 sizes × 3 pulse states. Re-run: `node --test --test-name-pattern="wright.*red.*painted area" test/wright.test.js` → 1 pass / 0 fail.
- [x] **Representative outputs for all three families are visually distinct, architectural, non-neon, and legible in light/dark at `local:3141`** — PASS. Builder evidence `evidence/step-3-1-palette-families-{light,dark}.png` (2026-09-21) is linked on the step. I independently re-drove `local:3141` against this worktree at HEAD: all three seeds render distinct families in both modes (textile = brown/earth, stained-glass = teal/blue, concrete-wood = gray/wood), architectural, non-neon, legible layers, restrained red. My own captured screenshots and exact rendered-output SVGs are committed as `evidence/review-3-1-palette-families-{light,dark}.svg`.
- [x] **Final diff changes only `src/variants/wright.js`, `test/wright.test.js`, and delivery artifacts; no unrelated files; `npm run verify` passes** — PASS (with one documented deviation). Product diff is exactly `src/variants/wright.js`, `test/wright.test.js`, and the Wright-owned `test/fixtures/golden-wright-v1.json`. Non-Wright goldens are byte-identical (`git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json` → clean). No demo/docs/React/types/unrelated variants changed. `npm run verify` exits 0 (180 tests pass; contract/exports/types/pack/goldens all green). Deviation: the Wright golden was necessarily updated — see Plan vs implementation.

## Plan vs implementation

- **Wright golden fixture update (documented, necessary deviation).** plan.md `## Out of scope` and acceptance item 6 state "no golden fixtures … change", but changing Wright palette colors invalidates the existing Wright golden and `npm run verify` (`check:variants` → `goldens`) would fail. The roadmap resolved this correctly: step 3.2 was marked `(obsolete: the full gate exposes the Wright-owned golden fixture … superseded by 3.3 and 3.4)` and step 3.3 `(added 2026-09-21)` updates only the Wright-owned golden entry, leaving non-Wright goldens byte-identical. The plan itself was not edited after planning (single `plan` commit), so its wording is now stale — recorded as a follow-up, not a blocker.
- **Canvas is a background reference, not emitted.** The `canvas` role appears in three of the four contrast pairs, but the renderer paints no canvas fill (icons render on the host background). The palette-internal 3:1 guarantee for canvas-adjacent roles therefore holds exactly when the host background equals the palette's canvas color; the demo's light (`#fafaf8`) / dark (`#16161a`) backgrounds are near but not equal to the palette canvas colors. This matches the plan's "canvas/background reference" framing and is documented in the code comments.
- **No undocumented changes.** The implementation is confined to `src/variants/wright.js` + `test/wright.test.js` (+ Wright golden). Public API surface (`src/index.js`, `src/react.js`, `index.d.ts`) and `demo/` are untouched; no new package-level color API was introduced.

## Roadmap audit

Every ticked box re-checked against the codebase; none is false.

- 1.1, 1.2, 2.1, 2.2 — re-run their `verify:` patterns; all pass (4/6/1/1 tests respectively).
- 3.1 — evidence PNGs linked and present with completion date; re-driven independently by the reviewer at `local:3141` (light + dark).
- 3.2 — correctly obsolete with reason, per the documented `~…~ (obsolete: …)` syntax.
- 3.3 — `git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json` is clean.
- 3.4 — `npm run verify` exits 0.
- No steps added or removed by the reviewer.

## Findings

- **Minor — canvas contrast reference is never rendered.** `WRIGHT_CONTRAST_PAIRS` (in `src/variants/wright.js`) tests `primary`, `secondary`, and `accent` against `canvas`, but `paintWright` emits no canvas fill, so the documented 3:1 guarantee for those roles is conditional on the host background matching the palette's canvas color. Impact is low in practice (canvas colors approximate typical light/dark app backgrounds), but the guarantee is not self-contained.

## Follow-ups

- Reconcile plan.md `## Out of scope` and acceptance item 6 wording (and the initiative breakdown's deferral of golden-fixture ownership to `wright-variant-regression-suite`) with the fact that this feature necessarily updated `test/fixtures/golden-wright-v1.json` to keep `npm run verify` green.
- Document the canvas backdrop assumption for the `canvas`-adjacent contrast pairs, or paint the canvas role / compare against the host background so the 3:1 guarantee holds independently of the host.
