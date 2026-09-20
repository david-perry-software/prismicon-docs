# Wright variant spec scaffold

## Problem

Implement the `### wright-variant-spec-scaffold` member block from [initiatives/2026/09/frank-lloyd-wright-variant/breakdown.md](../../../../initiatives/2026/09/frank-lloyd-wright-variant/breakdown.md):

- Summary: add the Wright variant descriptor scaffold, registration, and frozen derivation-spec framing so the new family is selectable through existing APIs without behavior changes elsewhere.
- Brief: establish where and how the new variant plugs into the repository abstractions and public variant selection flow.

This feature is intentionally a scaffold/spec boundary only. Geometry grammar, palette, motion semantics, and final legibility tuning are deferred to downstream initiative members (`wright-variant-geometry-grammar`, `wright-variant-palette-system`, `wright-variant-motion-events`, `wright-variant-legibility-tuning`).

User-visible effect for this feature alone: a new Wright variant id becomes selectable through existing variant resolution and rendering entry points, with deterministic scaffold behavior and no unintended changes to existing variant identities.

## Decisions

Clarifying decisions were provided and are retained verbatim:

1. Scope boundary: scaffold/spec framing only now; downstream initiative members handle geometry/palette/motion implementation.
2. Verification gates in acceptance and roadmap: require both `npm test` and `npm run check:variants`.
3. Public API boundary: no public API/type export changes unless absolutely required; default is none for this feature.
4. Execution style: safety-first incremental steps with minimal churn.

## Research

Skills consulted: modern-javascript-patterns, vercel-react-best-practices

- Variant registration and resolution seams are centralized in `src/variants/index.js` (`BUILT_IN_VARIANTS`, `resolveVariant`, `listVariants`) and `src/variants/registry.js` (`defineVariant`, `createVariantRegistry`), making scaffold insertion feasible without broad renderer rewrites.
- Renderer entry points already resolve per-icon variant keys in `src/core.js` (`renderStaticSVG`, `mountGlyph`) and enforce error contracts that tests pin in `test/variants.test.js` and `test/renderer-dispatch.test.js`.
- Authoring pathway (`createPrismicon`) in `src/authoring.js` depends on stable registry contracts and validates variants via `validateVariant`, so scaffold work should preserve deterministic hooks and avoid API broadening.
- Maintainer gate coverage in `scripts/check-variants.mjs` enforces exports/types/pack/golden invariants, which is aligned with keeping this feature scoped and low churn.

Lint baseline (policy section 5): repository `AGENTS.md` declares `Lint: none`; no separate lint command is configured. Verification baseline and gate evidence captured from this planning worktree:

- `npm test` exited 0 after dependency install (`npm ci`), with `pass 160`, `fail 0`.
- `npm run check:variants` exited 0, reporting `contract`, `exports`, `types`, `pack`, and `goldens` checks as passing.

Overlap decision:

- `gh pr list --state open --json number,headRefName` returned `[]`.
- No open delivery branch overlap exists at planning time, so no sequencing constraint is required beyond normal `origin/main` integration before pushes.

## Approach

- Add only the minimum Wright scaffold required for variant contract participation: descriptor definition, registry inclusion, and deterministic scaffold derivation/spec framing.
- Preserve current public API and type surface by default. If a public-surface change becomes unavoidable, document the necessity explicitly before implementation and limit it to the smallest compatible delta.
- Execute in incremental checkpoints with fast feedback after each checkpoint, favoring isolated changes to variant modules and targeted tests over broad refactors.
- Keep existing variant output stability as a hard guardrail: no drift in polyhedron/ncube/orbit identities or export contract behavior.

Primary files expected in scope:

- `src/variants/index.js`
- `src/variants/registry.js` (only if scaffold shape constraints require it)
- `src/variants/` (new Wright scaffold module)
- `test/variants.test.js`
- `test/renderer-dispatch.test.js` and/or focused Wright variant tests
- `test/helpers/golden.js`, `test/fixtures/` (only if scaffold requires fixture registration)

## Risks

- Scaffold work can accidentally expand into geometry/palette/motion implementation; mitigate by explicitly deferring those concerns and rejecting out-of-scope changes in this feature.
- Adding a new built-in id can unintentionally alter default behavior or export expectations; mitigate with explicit default-variant parity assertions and maintainers gate checks.
- Deterministic contract regressions could leak via derive/prepare/pose hooks; mitigate by enforcing deterministic tests and running `npm test` plus `npm run check:variants` at each milestone.
- Public API creep (types/exports) could create irreversible surface commitments; mitigate by treating API/type changes as disallowed unless strictly required and justified.

## Out of scope

- Wright geometry grammar implementation.
- Wright palette/material system behavior.
- Wright runtime motion/event semantics beyond a minimal deterministic scaffold.
- Legibility tuning and production visual fidelity across size bands.
- Companion/demo narrative polish for final Wright behavior (handled by later initiative members).

## Acceptance checklist

- [ ] Wright scaffold/spec framing is implemented as a contract-valid built-in variant integration point, with geometry/palette/motion sophistication explicitly deferred; verify by focused tests that exercise registration, resolution, and deterministic hook behavior.
- [ ] Existing public API/type export surface remains unchanged unless a documented, unavoidable exception is made; verify by comparing export/type contract checks in `npm run check:variants` and targeted tests.
- [ ] Existing built-in variant behavior remains stable with no unintended identity or rendering regressions; verify by `npm run check:variants` (including goldens) and `npm test`.
- [ ] Full required verification gates pass on the feature branch: `npm test` and `npm run check:variants`.
