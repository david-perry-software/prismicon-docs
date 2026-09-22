# Review: wright-variant-regression-suite

Verdict: approve

Reviewed 2026-09-21 in worktree `/home/david/DP/prismicon-worktrees/plan-20260922-020159`,
branch `feature/wright-variant-regression-suite` at `92da995` (draft PR #23;
companion draft PR #8, companion HEAD `9ff1324`). `git fetch origin`;
`git merge-base --is-ancestor origin/main HEAD` → true (branch contains `origin/main`);
companion half on `feature/wright-variant-regression-suite` with
`git merge-base --is-ancestor origin/main HEAD` → true, `dirty: false`, `ahead: 0`.
Diff reviewed: `git diff origin/main...HEAD` (5 files, +2003/−1).
Skills consulted: modern-javascript-patterns (Node.js ESM, `const`/`Object.freeze`,
pure `animate`/`pose` identity contract, no mutation); vercel-react-best-practices
not applicable (no React code changed; `src/react.js` untouched).

## Acceptance checklist results

| # | Item | Result | Evidence (commands run by the Reviewer) |
|---|---|---|---|
| 1 | `test/golden-wright.test.js` exists, mirrors `test/golden-ncube.test.js`, passes | **pass** | File present in diff (`A test/golden-wright.test.js`, +53); body matches `test/golden-ncube.test.js` (keys-equal assertion + per-static-key/per-mounted-frame-hash re-capture). `node --test test/golden-wright.test.js` → exit 0, `# tests 2`, `# pass 2`, `# fail 0` |
| 2 | `test/fixtures/golden-wright-v1.json` contains representative Wright seeds — all four geometry families, all three palette families, at least one verified hybrid — in both static and mounted sections | **pass** | Inspected fixture: top-level key `['wright']`; static 38 keys (30 shared + 8 wright extras), mounted 13 keys (4 shared + 8 wright + 1 reduced-motion). Extras present in both sections: `wright-family-1` (prairie), `wright-family-5` (art-glass), `wright-family-3` (textile-block), `wright-family-0` (usonian), `wright-palette-1` (textile), `wright-palette-11` (stained-glass), `wright-palette-0` (concrete-wood), `wright-hybrid-2`. `deriveWright('wright-hybrid-2')` → `dominantFamily: art-glass`, `secondaryFamily: textile-block` (hybrid confirmed) |
| 3 | Existing variants byte-identical and unaffected | **pass** | `git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json` → exit 0. `node scripts/generate-golden.mjs` → exit 0; `git status --porcelain` empty afterwards (regeneration is deterministic; only `golden-wright-v1.json` differs from `origin/main`) |
| 4 | Event-frame stability hash-locked; mounted golden scenarios include the representative seeds | **pass** | New test `wright event-frame stability holds for representative seeds across every probed state` passes (`ok 134` in `npm test`). Mounted golden keys include all 8 representative seeds (item 2). `node --test test/wright.test.js` (via suite) and `npm run check:variants` → `✓ goldens` exit 0 |
| 5 | `scripts/measure-wright.mjs` exists, runs cleanly, same gate columns as ncube/orbit; output recorded at `evidence/wright-benchmark.txt` | **pass** | `node scripts/measure-wright.mjs` → exit 0, `static gate: pass`, `frame gate: pass` for all 13 seeds. `evidence/wright-benchmark.txt` exists in the companion half and contains `frame gate: pass` (grep count 1). Gate columns/constants match `scripts/measure-orbit.mjs` (`bytes`/`median paint ms`, `median frame ms`/`p95`/`settle frames`, `<= 32768`/`<= 5`/`<= 2`/`<= 4`/`<= 60`) |
| 6 | Full maintainer gate green with the extended fixtures | **pass** | `npm run verify` → exit 0: `npm test` `# tests 191`, `# pass 191`, `# fail 0`; `check:variants` `✓ contract`, `✓ exports`, `✓ types`, `✓ pack`, `✓ goldens` |

Lint gate (policy §5): AGENTS.md declares `Lint: none` and `package.json` has no lint
script; the plan records the baseline as `none configured` (exit status N/A, no
findings). The gate is therefore the full test + maintainer suite — fresh `npm run
verify` green with no failures and no new findings.

## Plan vs implementation

- Scope matches the plan exactly. `git diff --name-status origin/main...HEAD` is
  `A scripts/measure-wright.mjs`, `M test/fixtures/golden-wright-v1.json`,
  `A test/golden-wright.test.js`, `M test/helpers/golden.js`, `M test/wright.test.js`
  — nothing else. `src/`, `demo/`, `README.md`, `index.d.ts`, `src/react.js`, and
  every non-Wright fixture are untouched, satisfying the Out-of-scope contract.
- `test/helpers/golden.js` adds the seed extension per-variant
  (`VARIANT_SEEDS.wright` only; `extraSeeds(variant)` returns `[]` for every other
  family), so the shared `STATIC_SEEDS`/`MOUNTED_SCENARIOS` and all other fixtures
  are unchanged.
- `test/golden-wright.test.js` mirrors `test/golden-ncube.test.js` structurally: a
  keys-equal-to-registered-ids assertion, then a re-capture comparing every static
  key and every mounted frame hash. This brings Wright golden freshness into plain
  `npm test`, as planned.
- `test/wright.test.js` adds the event-frame stability test exactly as described:
  over the 8 representative seeds × every `PROBE_STATES` state it asserts deep-equal
  determinism across two runs, `NaN|Infinity`-free output, positions inside
  `WRIGHT_VIEWBOX_BOUNDS` (the coordinate regex matches real attributes — 40
  coordinates for a sample frame), and settle-to-rest by identity within a bounded
  frame count.
- `scripts/measure-wright.mjs` mirrors `scripts/measure-orbit.mjs` including gate
  columns and thresholds, and is not wired into `npm test`, `check:variants`, or CI
  (advisory only), matching the plan.
- No deviations or undocumented changes found.

## Roadmap audit

All 9 ticked boxes were spot-checked against the codebase and by rerunning their
`verify:` lines: 1.1/1.2 (`node scripts/measure-wright.mjs` → exit 0, both gates
`pass`; `evidence/wright-benchmark.txt` committed and containing `frame gate: pass`),
2.1 (`node --test test/golden-v1.test.js test/golden-ncube.test.js test/golden-orbit.test.js test/wright.test.js`
→ 37 pass / 0 fail), 2.2 (`node --test test/golden-wright.test.js` → 2 pass / 0 fail),
2.3 (`node scripts/generate-golden.mjs` → exit 0; `git diff --exit-code origin/main`
on the three non-Wright fixtures → exit 0; `git status --porcelain` empty),
3.1 (new stability test passes; `npm test` 191/191), 3.2 (mounted golden includes
the representative seeds; `npm run check:variants` `✓ goldens`), 4.1 (`origin/main`
is an ancestor; `npm run verify` exit 0), 4.2 (`npm test && npm run check:variants`
exit 0; non-Wright fixture isolation re-confirmed). No falsely ticked boxes. No
`(manual)` or `(manual, post-ship)` steps. No repairs were needed; `roadmap.md` is
unchanged by this review.

## Findings

1. **Info** — `scripts/measure-wright.mjs` prints the run date as
   `new Date().toISOString().slice(0, 10)` (UTC), so the recorded line can read one
   day ahead of the local run date near midnight (observed `2026-09-22` for a
   `2026-09-21` run). This is inherited verbatim from `scripts/measure-orbit.mjs`
   and `scripts/measure-ncube.mjs`, so it is pre-existing and cosmetic — no action
   required for this delivery.
2. Security: no concerns. The change adds test/benchmark code only; it introduces no
   new input handling, secrets, or network surface, and no `src/` behavior changes.

No findings above minor severity.

## Follow-ups

- (Optional) align the benchmark scripts' date line with local time or label it UTC
  (finding 1) — shared across all three `scripts/measure-*.mjs`; not blocking.
