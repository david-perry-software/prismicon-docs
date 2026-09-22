# Step 1.1 — Audit: `wright` registration and React selection

## Finding

`wright` is already registered and resolves through the React binding with no
per-variant code. No code change was required for this step.

- `src/variants/index.js` registers `wright` in the immutable built-in registry:
  `BUILT_IN_VARIANTS = createVariantRegistry([polyhedron, ...ncubeVariants, orbit, wright], …)`
  and re-exports `wright` plus `WRIGHT_SPEC_VERSION`.
- `src/react.js` (`Prismicon`) resolves the `variant` prop through
  `ctx.registry.resolve(variant)` with no per-variant branches, and
  `PrismiconProvider` accepts any `VariantRegistry`, so `wright` works through
  the React binding unchanged.
- `listVariants()` returns the ids `polyhedron,ncube,ncube-3,ncube-4,ncube-5,ncube-6,orbit,wright`.

## Verify output

```
$ node -e "import('./src/index.js').then(m=>console.log(m.listVariants().map(v=>v.id).join(',')))"
polyhedron,ncube,ncube-3,ncube-4,ncube-5,ncube-6,orbit,wright
```

```
$ node --test test/renderer-dispatch.test.js test/react-variant.test.js
ℹ tests 30
ℹ pass 30
ℹ fail 0
```

The dispatch suite includes `built-in Wright variant renders from static and
mounted entries` — confirming `wright` resolves through the renderer's dispatch
path as well as through the React binding.
