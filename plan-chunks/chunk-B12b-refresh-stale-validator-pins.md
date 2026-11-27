# Chunk B12b: refresh two stale pins in `ccis-validator.test.js`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B12 (DONE). Decisions: none new. Before B13.

## What exists
Two tests in `test/ccis-validator.test.js` fail on HEAD (checked 2026-11-28 on a fresh clone, 21 pass, 2 fail). They fail because earlier rows legitimately changed the validator, not because of a bug.
- Test 12 "layer 2 ... a type with no schema yet" (starts line 165; the group case at lines 187 to 190): a manifest-only `element-group` was expected to give one warning `CCIS-V2-006`. Since B3 the element group has a schema, so it now gives an error `CCIS-V2-003` (required member missing).
- Test 23 "the library has no dynamic code ..." (about line 396): the import list is pinned to three modules. `ccis-validator.js` now imports 13 (group, entryform, rulepack, rulepack-checks, contexts, owner, terminology, concept, concept-package, concept-owners, plus the original three).

## What to change
Test file only. No source change.
1. Lines 187 to 190: the "no schema yet" case must use a type that still has no schema. Find one with `grep -n "V2-006" ccis-validator.js` and the schema list; if none is left, replace the case by the manifest-only group giving `['error', 'CCIS-V2-003']` and rename the case text ("a manifest-only group is missing required members"). Keep the rest of test 12 unchanged.
2. Line 396: replace the pinned list by the exact current import list, in file order (read it with `grep -nE "^import .* from|^\} from" ccis-validator.js`).

## Files to read
`test/ccis-validator.test.js` lines 160 to 195 and 390 to 400; `ccis-validator.js` import lines only.

## Tests
No new file. Run `node --test test/ccis-validator.test.js` (expect 23 pass, 0 fail) and the full-file check `ccis-validator-*.test.js`. One breakage check: add a made-up import to the pin, see test 23 fail, restore.

## Size guard
2 edits in 1 existing test file. If a third test fails or any source file needs a change, stop and split.

## Safety notes
Pins only. Do not weaken the forbidden-API list in test 23: only the allowed import list changes.

## Context impact
Reads about 3 KB; changes about 6 lines. Next: B13.
