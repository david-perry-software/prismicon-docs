# Review: wright-variant-geometry-grammar

Verdict: approve

## Acceptance checklist results

Tally: 6 pass / 0 fail / 0 deferred.

1. **PASS — Deterministic derivation covers all four dominant families and controlled hybrids without conditional draw-order drift.** `src/variants/wright.js` consumes all 12 named PRNG draws before branching and restricts secondary families through `WRIGHT_HYBRID_COMPATIBILITY`. `node --test test/wright.test.js` exited 0 with 14/14 tests. A reviewer-only 10,000-seed probe exited 0, reached all four dominant families and all eight allowed dominant/secondary pairs, and found no quota, freeze, finiteness, or bounds failure.
2. **PASS — Every composition has the required bounded semantic hierarchy and recognizable family structure.** `buildWright` emits frozen `primaryMasses`, `horizontalPlanes`, `gridModules`, `decorations`, and `accents` within `WRIGHT_LAYER_LIMITS`; the focused hierarchy and paint-order assertions passed. Independent browser validation selected `wright` in `demo/index.html` and rendered a 30-SVG matrix covering four pure families plus one controlled hybrid in both themes at 24, 64, and 72 pixels. Evidence: [reviewer-wright-geometry-matrix.png](evidence/reviewer-wright-geometry-matrix.png).
3. **PASS — Geometry and SVG output are immutable, finite, deterministic, and stroke-aware within the viewBox.** The focused size-matrix test passed at 24, 64, and 72, repeated paint output was byte-equal, and the reviewer 10,000-seed/all-state probe found no `NaN`, `Infinity`, bounds, or deep-freeze failure. The browser matrix showed no clipping at any requested size.
4. **PASS — Existing palette inputs and motion/flash semantics remain intact while painting uses the richer model.** The diff preserves hash-derived `hue`/`hue2`, `poseWright`, `animateWright`, and `flashWright`; the focused motion, settling, deterministic paint, and flash assertions passed. The full Wright golden includes the existing lifecycle states and passed the golden check.
5. **PASS — Existing variants, public exports, package contents, and non-Wright golden identities remain unchanged.** `npm run check:variants` exited 0 with `contract`, `exports`, `types`, `pack`, and `goldens` passing. SHA-256 comparisons against `origin/main` proved `golden-v1.json`, `golden-ncube-v1.json`, and `golden-orbit-v1.json` byte-identical. Only the intentionally changed Wright-owned fixture moved.
6. **PASS — The no-lint configuration remains accurate and the complete replacement gate passes.** The package-script probe confirmed no `lint` script. Reviewer runs of `npm test` (175 pass, 0 fail), `npm run check:variants`, and `npm run verify` all exited 0. VS Code diagnostics reported no errors in the four changed JavaScript files.

## Plan vs implementation

Skills consulted: modern-javascript-patterns.

The implementation follows the planned fixed-budget derivation, dominant-family profiles, bounded hybrid influence, semantic layer model, stable paint order, deep freezing, and existing registry boundary. No registry, demo, React, declaration, package API, palette-system, or motion-event source changed, and no security-sensitive input, persistence, network, or privilege surface was introduced.

The product diff expanded from the two primary files anticipated by the plan to five files. This is documented by roadmap step 3.4: the intentional Wright spec/output change required two scaffold-era integration expectation updates and regeneration of `test/fixtures/golden-wright-v1.json`. The reviewer confirmed the other three golden families are byte-identical to `origin/main`, so the expansion does not alter unrelated variant identities.

The independent visual drive used the allocated `local:3172` target from this exact product worktree. The real demo successfully selected `wright` and mounted its working SVG; the reviewer matrix then exercised the plan's representative pure-family/hybrid, light/dark, and 24/64/72 combinations.

## Roadmap audit

All 10 ticked steps were spot-checked against source, commits, reviewer-run commands, and browser output. No falsely ticked box or missing-work step was found, so `roadmap.md` required no repair.

- Steps 1.1 and 3.3: independently verified by the passing no-lint probe and complete `npm run verify` gate.
- Steps 1.2 and 2.1–2.4: independently verified by the focused Wright suite, direct source inspection, the 10,000-seed probe, and browser rendering.
- Step 3.1: the committed evidence exists, and the reviewer independently re-drove `local:3172` and captured separate evidence.
- Step 3.2: correctly marked obsolete with its reason retained; the replacement integration work is represented by added step 3.4.
- Step 3.4: independently verified by the 43-test integration command, PR file audit, full maintainer gate, and non-Wright fixture hashes.

Both product and companion branches contain their refreshed `origin/main`, are not behind their mirrored remote branches, and PRs #19 and #4 were open, draft, merge-clean, and scoped to the expected files at review time.

## Findings

- **Minor — stale scaffold naming:** `src/variants/wright.js` still describes the module as a placeholder scaffold and exposes the registry label `Wright Scaffold`, although this delivery replaces the scaffold with the geometry grammar. This does not affect correctness or the acceptance criteria, but the source description and demo-facing label no longer describe the implementation accurately.
- No correctness, security, API compatibility, or blocking code-quality findings.

## Follow-ups

- Rename the Wright descriptor label and module header when the product naming is finalized so the implemented variant is no longer presented as a scaffold.