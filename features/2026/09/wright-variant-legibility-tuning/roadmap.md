```yaml
status: in-progress
branch: feature/wright-variant-legibility-tuning
last-updated: 2026-09-21
next-step: "2.1"
artifact-pr: "#7"
initiative: "frank-lloyd-wright-variant"
```

## Phase 1: Thresholds and reduction

- [x] 1.1 Add frozen `WRIGHT_SMALL_SIZE`, `WRIGHT_SMALL_GRID`, and `WRIGHT_SMALL_DECORATIONS` constants and a `small: size < WRIGHT_SMALL_SIZE` flag to `prepareWright`, leaving `deriveWright`, `WRIGHT_DRAW_ORDER`, and `WRIGHT_SPEC_VERSION` untouched — verify: `node --test --test-name-pattern="wright.*size|wright.*derive" test/wright.test.js`
- [x] 1.2 Make `buildWright` cap the effective grid columns/rows to `WRIGHT_SMALL_GRID` and `decorationCount` to `WRIGHT_SMALL_DECORATIONS` when `params.small` is true, and rework the size test in `test/wright.test.js` to assert reduced module/decoration counts at size 24 and full counts at 64/72 — so size 24 emits fewer modules/decorations while `size >= 28` stays byte-identical — verify: `node --test test/wright.test.js`

## Phase 2: Legibility invariants

- [ ] 2.1 Extend the size test with the remaining legibility invariants — every layer inside `WRIGHT_VIEWBOX_BOUNDS`, strokes `>= 1.3`, a minimum grid-module pane footprint, and no `NaN`/`Infinity` across sizes 24/64/72/140 — verify: `node --test --test-name-pattern="wright.*legibility|wright.*size" test/wright.test.js`
- [ ] 2.2 Confirm WCAG structural contrast `>= 3:1` and the red painted-area ratio `<= 0.10` still hold at small sizes after reduction (narrow the small-size accent width, floor `1.3`, only if the ceiling is actually breached) — verify: `node --test --test-name-pattern="wright.*contrast|wright.*red" test/wright.test.js`

## Phase 3: Freeze, golden, and verification

- [ ] 3.1 Regenerate the Wright golden and confirm only the six `{"size":24}` keys (five static + the `maya` mounted scenario) change, with every other Wright entry and all non-Wright fixtures byte-identical — verify: `node scripts/generate-golden.mjs && npm run check:variants && git diff --exit-code origin/main -- test/fixtures/golden-v1.json test/fixtures/golden-ncube-v1.json test/fixtures/golden-orbit-v1.json && node -e 'const fs=require("fs"),cp=require("child_process");const cur=JSON.parse(fs.readFileSync("test/fixtures/golden-wright-v1.json","utf8"));const old=JSON.parse(cp.execSync("git show origin/main:test/fixtures/golden-wright-v1.json","utf8"));const bad=[];const walk=(a,b,p)=>{for(const k of new Set([...Object.keys(a),...Object.keys(b)])){const x=a[k],y=b[k],q=p+"/"+k;if(x&&y&&typeof x==="object"&&typeof y==="object"&&!Array.isArray(x))walk(x,y,q);else if(JSON.stringify(x)!==JSON.stringify(y)&&!q.includes("{\"size\":24}"))bad.push(q);}};walk(old,cur);if(bad.length){console.error("unexpected golden changes:",bad);process.exit(1)}console.log("golden diff limited to size-24 keys")'`
- [ ] 3.2 Drive the unchanged demo in a browser and confirm the Wright variant stays recognizable at small sizes and retains architectural character at larger sizes, capturing screenshots to `evidence/` — verify: `local:3184` — `ss -ltn | grep -c ':3184 '` prints `0`, then serve `python3 -m http.server 3184 --directory . --bind 127.0.0.1`, open `http://localhost:3184/demo/index.html`, select Wright in the variant `<select>`, and confirm small-size legibility and large-size character
- [ ] 3.3 Integrate `origin/main` (product and companion), recheck concurrent-delivery overlap, and run the full no-lint gate — verify: `node --test test/wright.test.js && npm run verify`
