# Chunk B6: pathway validator, `entryForm` rules (contexts split off to B6a)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B5 (DONE). Decisions: ED-5, ED-34 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` sections 3 and 9 (last bullet). Before B7.

## Why a split
The row text covers `entryForm` and `contexts`: about 12 rules, over the 8-case test guard. B6 takes `entryForm` (V5-036 to 043). New row B6a takes `contexts` (V5-044 to 047) and B8 then needs B6a too.

## What to build
New pure module `ccis-validator-entryform.js` (about 70 lines): `ENTRYFORM_RULES` and `entryFormChecks(doc, ctx)`. Layer 5, called for kind `pathway`. Skips everything when there is no `entryForm`. An element that did not resolve is skipped for field checks, never guessed (as in the group module).
1. `V5-036` error: an `element` item names an element id that is not in the pathway `elements`. Path `/entryForm/sections/<i>/items/<j>/element`. Needs no resolution.
2. `V5-037` error: an element in `elements` is placed nowhere in the form. Path `/elements/<k>`.
3. `V5-038` error: a field in `fields` or `rowGroups` that its element does not define. Path to the field.
4. `V5-039` error: a field placed more than once: repeated in `fields`, repeated across `rowGroups`, or in two placements of one element (a placement without `fields` places all fields). Path to the second place.
5. `V5-040` error: a `rowGroups` field that is not in the same item's `fields` (only when `fields` is given).
6. `V5-041` error: a `section` item names a name outside the closed five: `core.problems`, `core.allergies`, `core.medications`, `core.investigations`, `core.care-episode`.
7. `V5-042` error: two sections share an `id`. Path at the second.
8. `V5-043` warning: a `section` item `core.care-episode` (the EMR shows no area for it, design section 9).
Messages name ids and keys only.

ED-34 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) contexts split to B6a; (b) the five section names live in the module, not the schema (ED-33a); (c) an element may be placed twice only with disjoint field sets; (d) V5-037 applies only when an `entryForm` exists; (e) every rule applies at any standard version.

## Files to read (line ranges)
- `ccis-validator.js` lines 28 to 34 (imports), 56 to 62 (rule list), 602 (the layer 5 hook).
- `ccis-validator-group.js` lines 1 to 40 (module style; copy `isObj`, `arr`, `diag`, `fieldsOf`).
- `test/ccis-pathway-entryform-schema.test.js` lines 1 to 25 (fixture; copy). Element fixture: `ccis/examples/snake/snake-envenomation.element.json`.

## Files to change
New `ccis-validator-entryform.js`; `ccis-validator.js` (import, rule list, one hook entry); new `test/ccis-validator-entryform.test.js`; `index.html` rebuilt (`npm ci` first); CCIS tracker row `ECO-B6`. Studio: `DECISIONS.md` `ED-34`, tracker rows B6a and B8.

## Tests (at most 8 cases, `node --test`)
1. A valid form gives no V5-036 to 043 row; a pathway with no `entryForm` gives none.
2. V5-036 (element not in `elements`) and V5-037 (element left out).
3. V5-038 and V5-040 (unknown field; rowGroup field outside `fields`).
4. V5-039 (repeat in `fields`, across `rowGroups`, and across two placements).
5. V5-041 (unknown section name).
6. V5-042 (duplicate section id) at the second index.
7. V5-043 is a warning, not an error.
8. Unresolved elements: V5-036 and V5-041 still fire, no crash; a hostile id appears only inside message text.
Run only the new file plus `ccis-pathway-entryform-schema`, `ccis-demo-pathway`, `ccis-sdk-pathway`, `ccis-validator-group`, `ccis-market-diabetes-ckd`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 new module (about 70 lines), 1 test, 3 small wiring edits. If the module passes about 90 lines or an existing test changes, stop and split.

## Safety notes
Layout only. No clinical logic or number. Nothing is inferred from labels. Messages are plain, untrusted text on display.

## Context impact
Reads about 7 KB; adds about 70 source lines and a test of about 100 lines. Next: B6a (contexts rules).
