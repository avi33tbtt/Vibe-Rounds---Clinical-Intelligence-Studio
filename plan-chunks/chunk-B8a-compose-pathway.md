# Chunk B8a: `ccis-compose-pathway.js`, build a composed pathway from the group and the source pathways

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B8 (DONE). Decisions: ED-5, ED-12, ED-38 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` sections 2, 3, 4 and 6 (rules 1, 6, 7, 8). Before C14 and F17.

## What to build
New pure module `ccis-compose-pathway.js` (about 100 lines), importing `ccis-validator.js` first (the import cycle, as in B8) and `canonicalize` from `ccis-compile.js`. One export: `composePathway(input)`.
- `input`: `{ manifest: {id, name, version, publisher, standardVersion}, scope, intendedUse, exclusions, stages, group, pathways, elements }`. `group` is the `group` object from `composeGroup` (B8). `scope`, `intendedUse`, `exclusions` and `stages` are author text and are never generated; missing or empty gives blocker `missing-text`.
- `manifest`: `type: pathway`, `status: draft`, `references`: the group, then each source pathway, as `{id, type, version}` from their manifests, in the order given (rule 8).
- `elements`: group `members` order (first-seen), each `{id, type: contextual-element, range: <member version>}` (bare version, design section 2).
- `entryForm`: walk the sources in order. Sections with the same `id` merge, items appended; a different `label` for one id is blocker `section-conflict`. An `element` item is kept only at its first placement (rule 1); a `section` item (core name) likewise. A later placement that differs in `fields` or `rowGroups` is dropped with a line in `notes`. A shared concept shows once, at the owner (rule 6): for every group `sharedConcepts` field that is not the owner, the key is removed from its element's placement: `fields` becomes the explicit list (given, else all field keys of that element in order) without those keys, and `rowGroups` lose them (empty groups dropped, omitted if none left). A placement left with no field is blocker `no-field-left`. Field keys come from the supplied `elements` (highest version).
- `contexts`: union by `id` in first-seen order, `elements` united and deduplicated in first-seen order; a different `label` for one id is blocker `section-conflict` (same code, text says context).
- `analytics`, `rules`, `nodes` are not composed (E12 and later rows).
- Result `{ ok: true, pathway, bytes, notes }` or `{ ok: false, blockers }`; `bytes` is `ccis-canonical-json/1`. Same input gives identical bytes.

ED-38 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) new module; (b) first placement wins, later differing placements are dropped and noted, never merged; (c) sections merge by id, never by label; (d) `scope`, `intendedUse`, `exclusions` and `stages` are author inputs; (e) analytics and rules are not composed here.

## Files to read (line ranges)
- `ccis-compose-group.js` lines 1 to 20 and the return shape (B8).
- `test/ccis-validator-entryform.test.js` lines 1 to 25 (pathway and form fixture; copy).
- `test/ccis-compose-group.test.js` lines 1 to 30 (fixtures; copy).

## Files to change
New `ccis-compose-pathway.js`; new `test/ccis-compose-pathway.test.js`; CCIS tracker row `ECO-B8a`. Studio: `DECISIONS.md` `ED-38`, tracker row B8a. `index.html` not rebuilt.

## Tests (at most 8 cases, `node --test`)
1. Two sources with a shared section id and a shared element: one merged section, element placed once, references `[group, p1, p2]`.
2. Shared concept: the user key is removed from `fields` and from `rowGroups` (an emptied group dropped); the owner keeps it.
3. The pathway validates: `validateArtefact(pathway, {known: [group, p1, p2, ...elements]})` has no error row.
4. Byte determinism: same input twice gives equal `bytes`; no newline in `bytes`.
5. Contexts union and dedupe; a label conflict is `section-conflict`.
6. Section label conflict is `section-conflict`; missing `scope` is `missing-text`.
7. A placement left with no field is `no-field-left`.
8. A later differing placement is dropped and noted; inputs are not mutated.
Run only the new file plus `ccis-compose-group` and `ccis-validator-entryform`. At most 3 breakage checks.

## Size guard
1 new module (about 100 lines), 1 test. If the module passes about 120 lines, stop and split the form merge into a new row.

## Safety notes
Layout and references only. No clinical text, number or threshold is generated. No field is copied (design rule 8). Messages name ids and keys only.

## Context impact
Reads about 8 KB; adds about 100 lines and a test. Next: C-phase rows need B21; the CCIS contract continues with B9.
