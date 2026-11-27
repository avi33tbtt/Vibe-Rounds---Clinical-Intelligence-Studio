# Chunk B1f: validator checks for a terminology package

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1b (DONE 2026-11-28). Split from B1b. Comes before B2. Design: `docs/TERMINOLOGY-DESIGN.md` sections 2 and 7.

## What to build
New pure module `ccis-validator-terminology.js`, written like `ccis-validator-owner.js`: exports `TERMINOLOGY_RULES` and `terminologyChecks(doc)`. It checks one package against itself, so it needs no `ctx.resolve`. Diagnostics use JSON-pointer paths; messages name keys and ids only, never display text.

Rules (layer 5, ids continue after V5-016):
1. `CCIS-V5-017` error: two concepts share a key (path of the second one, `/concepts/<i>/key`). A key never changes meaning and is never reused (design rule 1).
2. `CCIS-V5-018` error: `replacedBy` names the concept itself, or a key that is not in `concepts` (`/concepts/<i>/replacedBy`).
3. `CCIS-V5-019` error: a value-set member key is not in `concepts` (`/valueSets/<i>/members/<j>/key`).
4. `CCIS-V5-020` error: a value-set id appears twice, or a member key appears twice in one value set.
5. `CCIS-V5-021` warning: a value-set member names a deprecated concept (still resolvable, so a warning).
Malformed shapes (non-array, non-object, non-string key) are skipped: layer 2 reports them.

Wiring in `ccis-validator.js`: import the module, spread `TERMINOLOGY_RULES` into `RULES` (after `OWNER_RULES`), and at the layer 5 line (about line 594) add `...(kind === 'terminology-package' ? terminologyChecks(first.doc) : [])`.

## Files to read (line ranges)
- `ccis-validator-owner.js` lines 1 to 20 (style, `diag`, `isObj`, `arr`); do not import from it.
- `ccis-validator.js` lines 28 to 33 (imports), 50 to 56 (`RULES`), 592 to 596 (layer 5 wiring).
- `test/ccis-terminology-package-schema.test.js` lines 1 to 25 (the `pkg()` fixture; copy, do not edit).

## Files to change
- New `ccis-validator-terminology.js` (about 45 lines).
- `ccis-validator.js`: import, `RULES` spread, one wiring edit.
- New `test/ccis-validator-terminology.test.js`.
- `index.html`: rebuild with `npm run build:app` (it embeds the validator).
No schema change, so `npm run build` is not needed (confirm with `node scripts/build-validators.js --check`).

## Tests (at most 8 cases, `node --test`)
1. The valid package from the B1b fixture gives no error and no V5-017 to 021 row.
2. A repeated concept key gives V5-017 at the second concept.
3. `replacedBy` naming itself gives V5-018; naming a missing key gives V5-018; naming an existing key passes.
4. A member key absent from `concepts` gives V5-019 with the member path.
5. A repeated value-set id gives V5-020; a repeated member key in one set gives V5-020 (two assertions).
6. A member naming a deprecated concept gives V5-021 as a warning only; outcome stays `no-errors-so-far`.
7. A package with a malformed `concepts` (not an array) fails at layer 2 and never reaches this module (no crash).
8. A contextual element and an element group still validate with no new row (guards wiring).
Run only the new file, `test/ccis-terminology-package-schema.test.js`, `test/ccis-validator-owner.test.js`, `test/ccis-validator-element.test.js` and `test/ccis-app-build-base.test.js`. `test/ccis-validator.test.js` has 2 old failures: report, do not fix. At most 3 breakage checks (drop the duplicate-key check and see case 2 fail; treat a deprecated member as an error and see case 6 fail; restore).

## Size guard
1 new module (about 45 lines), 3 small edits in `ccis-validator.js`, 1 test file, 1 generated bundle. If the module passes about 70 lines or the bundle changes beyond the embedded validator, stop and split.

## Safety notes
- Checks identity only: no numbers, bands, thresholds or external code systems.
- Keys in messages are plain text, untrusted on display.

## Context impact
Reads about 5 KB of source. Adds about 45 source lines and one test of about 90 lines. On completion: Studio tracker, CCIS tracker (`ECO-B1f`). No `DECISIONS.md` rows. Next row after this: B2.
