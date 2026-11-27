# Chunk B8: `ccis-compose-group.js`, build a composed element group from owner choices (pathway build split to B8a)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B7, B6, B6a (DONE). Decisions: ED-5, ED-12, ED-36, ED-37 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` section 6 (rules 1 to 8). Before B8a.

## Why a split
Group plus pathway (entry form merge, contexts, references, stages) is about 150 lines and 14 tests, over the guard. B8 builds the group; new row B8a the pathway. C14 and F17 also need B8a.

## What to build
New pure module `ccis-compose-group.js` (about 90 lines), importing `conflictReport` from `ccis-compose.js` (import `ccis-validator.js` first, as `ccis-compose.js` does) and `canonicalize` from `ccis-compile.js`. One export: `composeGroup(input)`.
- `input`: `{ manifest: {id, name, version, publisher, standardVersion}, intent, provenance, pathways, elements, choices }`. `intent` and `provenance` are supplied by the author in the shape of the group schema; the module never writes clinical or descriptive text of its own. Missing or empty gives a blocker `missing-text`. `choices`: `{ <concept>: {element, key} }`.
- Run `conflictReport(pathways, elements)` (B7). Any blocker other than `second-owner` (with a valid choice) stops the build: return `{ ok: false, blockers, report }`, no group.
- A choice is valid when it names one of the concept's owner fields. An invalid choice is a blocker `bad-choice`. A choice for a concept that is not shared is ignored with a blocker-free note in `notes`.
- Group (schema `element-group`): `manifest` from input with `type: element-group`, `status: draft`, `standard.version` from input; `members` in first-seen element order, each `{element: {id, type: contextual-element, version: <exact highest version>}, required}`; `sharedConcepts` sorted by concept, one per concept with status `ok` or a chosen `second-owner`: `{concept, meaning, fields: [{element, field}] sorted, resolution: same-concept, owner: {element, field}}`; `versionPolicy: {conflict: refuse}`. `possible-duplicate` concepts are never listed.
- `meaning` is the owner field's `concept.display` (taken, not written). Empty gives blocker `missing-text`.
- A chosen `second-owner` also returns `revisions: [{element, key, action: make-user-of, owner}]` for the owner fields that lost. Then `ready` is false: the elements need new versions before publish (design rule 3; B15 refuses a second owner). `ready` is true when `revisions` is empty.
- Result: `{ ok: true, ready, group, bytes, revisions, notes }`; `bytes` is the `ccis-canonical-json/1` text. Same input gives byte-identical `bytes`.

ED-37 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) new module, not a growth of `ccis-compose.js`; (b) `required` is true when the element is used by every source pathway, else false; (c) a choice never edits an element; it lists the revisions instead (design rule 3); (d) `intent` and `provenance` are inputs, never generated; (e) possible duplicates are left out of the group.

## Files to read (line ranges)
- `ccis-compose.js` lines 1 to 71 (B7; its report shape).
- `ccis/examples/diabetes-ckd/diabetes-ckd.group.json` (whole; shape to match).
- `ccis-compile.js` lines 15 to 25 and 203 to 215 (`canonicalize`).
- `test/ccis-compose-conflicts.test.js` lines 1 to 20 (fixtures; copy).

## Files to change
New `ccis-compose-group.js`; new `test/ccis-compose-group.test.js`; CCIS tracker row `ECO-B8`. Studio: `DECISIONS.md` `ED-37`; tracker row B8 narrowed, new row B8a, `Needs` of C14 and F17 gain B8a. `index.html` is not rebuilt.

## Tests (at most 8 cases, `node --test`)
1. Clean case (design example, blood pressure shared): group has two members, one shared concept, owner set, `ready` true.
2. Output validates: `validateArtefact(group, {known: elements})` has no error row (the 0.1.0 warnings are allowed).
3. Byte determinism: reversed `elements` and `pathways` order of one pathway each gives identical `bytes`; `bytes` has no whitespace outside strings.
4. `required`: true for an element in both pathways, false for one in a single pathway.
5. Second owner with a valid choice: group built, `revisions` lists the loser, `ready` false.
6. Invalid choice (not an owner field) gives `bad-choice`; no group.
7. Unit conflict or missing intent gives blockers `unit-conflict` or `missing-text`; no group.
8. Possible duplicates are not in `sharedConcepts`; inputs are not mutated.
Run only the new file plus `ccis-compose-conflicts` and `ccis-validator-group`. At most 3 breakage checks.

## Size guard
1 new module (about 90 lines), 1 test. If the module passes about 110 lines, stop and split `revisions` into a new row.

## Safety notes
No clinical text is generated: `meaning` is copied from the element; `intent` and `provenance` come from the author. No number or threshold. Messages name ids, keys and codes only.

## Context impact
Reads about 8 KB; adds about 90 lines and a test. Next: B8a.
