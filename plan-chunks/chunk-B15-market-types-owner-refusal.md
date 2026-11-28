# Chunk B15: market publishes element groups and rule packs; refuses a second concept owner

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B6, B13c, B2a (all DONE). Decisions: ED-5, ED-12, ED-51 (new, OPEN working choices). Narrowed from the old B15: the index `requires` and `provides` lookups are B15a (they change the index format); the `market-build` update is C15's task. Before B15a and C15.

## What exists
`ccis-market.js` (288 lines, pure): `PUBLISHABLE_TYPES = ['contextual-element','analytics-package','pathway']` (line 26), refusal `M-002` names those three (line 41), `publishArtefact` checks the type at lines 151 and 161, compiles and validates with the market's files as `known`, and writes a 12-member index entry. `ENTRY_MEMBERS` (line 258) is closed at exactly 12, so no index member is added here. `ccis-concept-owners.js` `conceptOwners(artefacts)` reads the highest version of each element and returns per concept `status: ok | needs-owner | second-owner` (owner = a field without `ownedBy`). The validator already knows `element-group` and `rule-pack` (`TYPE_TO_SCHEMA`, layers 5 to 8).

## What to build
In `ccis-market.js` only (about 15 lines):
- `PUBLISHABLE_TYPES` gains `element-group` and `rule-pack`; the `M-002` sentence says "contextual elements, element groups, analytics packages, pathways and rule packs". `terminology-package` stays out (C4 decides).
- New refusal `M-014` ("A concept in this element already has an owner element in the market. Name the existing owner instead of owning the concept again."). In `publishArtefact`, after the compile step and before the immutability check, when the subject is a `contextual-element`: run `conceptOwners([...known, artefact])`; if any concept carried by the new element has `status: 'second-owner'`, refuse with `detail: { concepts: [...], elements: [...] }` (concept codes and element ids only, sorted, no values). The market is unchanged on refusal.
- A new version of the same element id is not a second owner (the highest version of an id wins, as in `conceptOwners`).

ED-51 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) add the two types, not `terminology-package`; (b) the refusal reuses `conceptOwners` and runs for elements only; (c) no index format change in B15; (d) `requires` and `provides` move to B15a with their own format decision (optional members or `ccis-market/2`); (e) the `market-build` update merges into C15.

## Files to read
`ccis-market.js` lines 25 to 45 and 144 to 175; `test/ccis-market.test.js` lines 1 to 30 and every line with `M-002` or `PUBLISHABLE`; an element-group fixture (`grep -ln element-group test/*.js | head -3`, then the fixture lines); the rule-pack fixture in `test/ccis-validator-rulepack-governance.test.js` lines 1 to 25.

## Files to change
`ccis-market.js`; new `test/ccis-market-b15.test.js`; at most 2 pin edits in `test/ccis-market.test.js` (type list, M-002 sentence); rebuilt `index.html` (the app imports the market); CCIS tracker row `ECO-B15`. Studio: ED-51, row B15a, C15 Needs. First check with a scratch run that a group and a rule pack pass `compileArtefact` and `buildValidationRecord`; if either needs a change outside `ccis-market.js`, stop and split.

## Tests (at most 8 cases)
1. An element group publishes (its member elements published first) and the entry has type `element-group` with its references.
2. A rule pack publishes (scope pathway published first); entry type `rule-pack`.
3. Same bytes again: `unchanged: true`; changed bytes with the same version: `M-006`.
4. A second element owning a concept that another published element owns gives `M-014` with the concept and element ids, and the market is unchanged.
5. A new version of the owner element publishes; a field that names `ownedBy` is not a second owner.
6. A `terminology-package` still gives `M-002` with the new sentence.
7. `loadMarket` of `indexText` and files with both new types round-trips.
8. Refusal detail holds ids and codes only (hostile string in a field label never appears).
Run the new file plus `ccis-market*.test.js`, `ccis-app-market.test.js`, `ccis-app-build*.test.js`. At most 3 breakage checks.

## Size guard
1 module edit, 1 test, at most 2 pin edits, 1 rebuilt file. More than that: split.

## Safety notes
A checksum proves bytes, not safety. Ownership is by element id, never by label. No clinical content.

## Context impact
Reads about 8 KB; adds about 15 module lines and 110 test lines. Next: B15a.
