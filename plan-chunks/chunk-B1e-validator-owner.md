# Chunk B1e: validator layer 5, concept owner checks

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1a (DONE 2026-11-28). Comes before B2a, which counts owners and users. Split from B1a. Decisions: ED-12 (one owner per concept), ED-16 (a user field repeats the owner's type and unit).

## What to build
New pure module `ccis-validator-owner.js`, written like `ccis-validator-group.js`: exports `OWNER_RULES` and `ownerChecks(doc, ctx)`. A field with `ownedBy` is checked against its owner, resolved through `ctx.resolve({ id, type: 'contextual-element', range: '>=0.0.0' })` (highest known version, as layer 3 does at `ccis-validator.js` line 480). An owner that does not resolve (`state` not `ok`) is skipped, never assumed. Never infer an owner from a key or a label.

Rules (ids continue after V5-013; all layer 5):
1. `CCIS-V5-014` error: the owner field does not exist in the owner element, or has no `concept`, or its `concept.code` differs from the user field's. Self-ownership (owner element is this element) is also this error.
2. `CCIS-V5-015` error: `type` or sorted `units` differ from the owner's (ED-16).
3. `CCIS-V5-016` warning: the user field's cardinality is wider than the owner's (larger `max`, or smaller `min`; `unbounded` is widest).
Fields come from `fields` and from module `fields`. Diagnostics use JSON-pointer paths like the group module (`/fields/2/ownedBy`, `/modules/0/fields/1/ownedBy`). Messages name ids and keys only, never values.

Wiring: in `ccis-validator.js` line 592 add `...(kind === 'contextual-element' ? ownerChecks(first.doc, ctx) : [])` next to the group call; import the module; spread `OWNER_RULES` into `RULES` (line 53 area). No other layer changes.

## Files to read (line ranges)
- `ccis-validator-group.js` lines 1 to 30 (`diag`, `fieldsOf`, resolve pattern); copy the style, do not import from it.
- `ccis-validator.js` lines 28 to 32 (imports), 50 to 55 (`RULES`), 585 to 593 (layer 5 wiring).
- `test/ccis-validator-group.test.js` lines 1 to 40 (helpers, `run`, `ids`, `mutate`).
- `ccis/examples/snake/snake-envenomation.element.json` (one field, for a base fixture).

## Files to change
- New `ccis-validator-owner.js` (about 55 lines).
- `ccis-validator.js`: import, `RULES` spread, one wiring line.
- New `test/ccis-validator-owner.test.js`.
- `index.html`: the app bundle embeds the validator, so run `npm run build:app` (esbuild is already pinned) and commit the generated file. Confirm with the build tests named below.

## Tests (at most 8 cases, `node --test`)
Build two synthetic elements in the test: an owner (`demo.owner`, field `ownerKey` with `concept`, type, units, cardinality) and a user (`demo.user`, field with `concept` and `ownedBy`).
1. A matching owner and user pass with no layer 5 row.
2. Owner element not among the known artefacts: skipped, no V5 row (layer 3 behaviour unchanged).
3. Owner field missing: V5-014.
4. Owner field has no `concept`, or a different `concept.code`: V5-014 (two assertions).
5. Self-ownership: V5-014.
6. `type` differs, and `units` differ: V5-015 (two assertions); same units in another order pass.
7. Wider cardinality: V5-016 warning only, outcome stays `no-errors-so-far`.
8. A module field with `ownedBy` is checked with the module path; every existing example element still validates with no change in diagnostics.
Run only the new file, `test/ccis-validator-group.test.js`, `test/ccis-validator-element.test.js`, `test/ccis-validator.test.js` and `test/ccis-app-build-base.test.js`. At most 3 breakage checks (drop the type comparison and see case 6 fail; treat a missing owner as an error and see case 2 fail; restore).

## Size guard
1 new module (about 55 lines), 3 small edits in `ccis-validator.js`, 1 test file, 1 generated file (`index.html`). If the module passes about 80 lines, or `npm run build:app` changes anything beyond the embedded validator, stop and split.

## Safety notes
- Checks identity, type, unit and cardinality only: no numbers, bands or thresholds.
- Ids and keys in messages are plain text, untrusted on display.
- Fail closed on structure: a malformed `ownedBy` is left to layer 2; this module skips non-objects.
- No external code system appears anywhere.

## Context impact
Reads about 8 KB of source. Adds about 60 source lines, one test of about 100 lines and one regenerated bundle. On completion: Studio tracker, CCIS tracker (`ECO-B1e`); CCIS `CHANGELOG.md` only if you approve an amendment. No `DECISIONS.md` rows. Next row after this: B1b.
