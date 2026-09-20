# Review: wright-variant-spec-scaffold

Verdict: approve

Reviewed at `96d1600` (`feature/wright-variant-spec-scaffold`, code PR #18 draft) against `origin/main` `69ecf9f` (confirmed ancestor of `HEAD` via `git merge-base --is-ancestor`, exit 0). Companion mode verified on mirrored companion branch `feature/wright-variant-spec-scaffold` with `artifact-pr: #3` and clean companion state.

Skills consulted: modern-javascript-patterns (domain match: modern JavaScript variant implementation and tests), vercel-react-best-practices (domain check only; no React source changes in this feature).

Reviewer-executed verification commands (2026-09-20):

- `npm test -- test/wright.test.js test/variants.test.js test/renderer-dispatch.test.js` (focused roadmap checks) -> exit 0, pass 166, fail 0.
- `npm test` (full suite) -> exit 0, pass 166, fail 0.
- `npm run check:variants` -> exit 0, all checks passing (`contract`, `exports`, `types`, `pack`, `goldens`).

Implementation diff reviewed with `git diff origin/main...HEAD`:

- `src/variants/index.js`
- `src/variants/wright.js`
- `test/helpers/golden.js`
- `test/ncube.test.js`
- `test/renderer-dispatch.test.js`
- `test/variants.test.js`
- `test/wright.test.js`
- `test/fixtures/golden-wright-v1.json`

## Acceptance checklist results

Tally: 4 pass / 0 fail / 0 deferred.

1. **Wright scaffold/spec framing is implemented as a contract-valid built-in variant integration point, with geometry/palette/motion sophistication explicitly deferred; verify by focused tests that exercise registration, resolution, and deterministic hook behavior.**
   - **PASS**.
   - Evidence in implementation: `src/variants/wright.js` defines a scaffold-only descriptor (`label: "Wright Scaffold"`, `spec: "wright-scaffold-v1"`) with deterministic derive/prepare/geometry/pose/animate/paint/flash hooks and no expansion into full downstream initiative scope.
   - Evidence in wiring: `src/variants/index.js` registers `wright` in `BUILT_IN_VARIANTS` while preserving default variant id `polyhedron`.
   - Evidence in tests: `test/wright.test.js` validates deterministic derivation, hook behavior, bounded paint output, and `validateVariant(wright)` pass; `test/renderer-dispatch.test.js` validates static and mounted rendering path for variant `wright`; `test/variants.test.js` validates list/resolve integration.
   - Focused verification command passed (see command list above).

2. **Existing public API/type export surface remains unchanged unless a documented, unavoidable exception is made; verify by comparing export/type contract checks in `npm run check:variants` and targeted tests.**
   - **PASS**.
   - `npm run check:variants` passed its `exports` and `types` checks.
   - `test/variants.test.js` includes `EXPECTED_PUBLIC_EXPORT_COUNT = 18` and asserts exact `src/index.js` public export keys remain unchanged.
   - No plan-documented unavoidable public API exception was required.

3. **Existing built-in variant behavior remains stable with no unintended identity or rendering regressions; verify by `npm run check:variants` (including goldens) and `npm test`.**
   - **PASS**.
   - `npm run check:variants` passed `goldens` and related contract checks.
   - `npm test` passed across full suite (166 pass / 0 fail), including existing variant families and dispatch coverage.
   - `test/helpers/golden.js` explicitly maps `wright` to its own `golden-wright-v1.json`, preserving fixture family partitioning rather than altering existing family mappings.

4. **Full required verification gates pass on the feature branch: `npm test` and `npm run check:variants`.**
   - **PASS**.
   - Reviewer independently executed both required gates on this branch; both exited 0.

## Plan vs implementation

Implementation is aligned with the approved plan and stated scope boundaries.

- Planned scaffold integration is present and minimally scoped:
  - New Wright scaffold module (`src/variants/wright.js`) with deterministic contract-valid hooks.
  - Built-in registry integration (`src/variants/index.js`) includes `wright` without changing default behavior.
- Planned verification expansion is present:
  - Focused Wright tests in `test/wright.test.js`.
  - Registry/resolve and list variant assertions updated in `test/variants.test.js`.
  - Static + mounted dispatch path coverage for Wright in `test/renderer-dispatch.test.js`.
- Planned contract stability is preserved:
  - Public export-count/assertion retained in tests; type/export checks pass.
  - Existing family golden fixtures remain stable while adding dedicated Wright fixture.

No undocumented source-code deviations were found.

## Roadmap audit

Result: no drift detected; no repairs required.

Spot-check of all ticked roadmap steps against code + reviewer-run verifies:

- 1.1 verified by independent full-gate run (`npm test && npm run check:variants`) and scaffold-only implementation boundaries in `src/variants/wright.js`.
- 1.2 verified by `npm run check:variants` passing and explicit public export-count assertions in `test/variants.test.js`.
- 2.1 verified by deterministic scaffold hooks and focused Wright tests in `test/wright.test.js`.
- 2.2 verified by built-in registration changes in `src/variants/index.js` and resolve/list assertions in `test/variants.test.js`.
- 2.3 verified by static + mounted dispatch coverage in `test/renderer-dispatch.test.js`.
- 3.1 verified by reviewer-run full gates and ancestry check that `origin/main` is an ancestor of `HEAD`.
- 3.2 verified by artifact state (`status: in-review`, empty `next-step`, all checks ticked) and no falsely ticked boxes found.

No `(manual)` or `(manual, post-ship)` steps exist for this feature.

## Findings

No findings above minor severity.

No request-changes findings were identified. Verification and acceptance evidence support approval.

## Follow-ups

- None required from this review.
