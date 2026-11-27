# Chunk B7: `ccis-compose.js` part 1, shared-concept matching and conflict report

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B4 (DONE). Decisions: ED-5, ED-12, ED-29, ED-36 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` section 6 (rules 1 to 7 and the conflict report). Before B8.

## What to build
New pure module `ccis-compose.js` (about 80 lines), no clock, no network, no randomness. One export: `conflictReport(pathways, elements)`.
- `pathways`: pathway docs in the order given. `elements`: element docs (any versions; the highest version of each id wins, as in `conceptOwners`). An element id used by a pathway but absent from `elements` is a blocker `missing-element`; nothing is guessed.
- `sharedElements`: element ids used by two or more pathways (rule 1), in first-seen order.
- `concepts`: one entry per concept carried by fields of two or more different used elements (rule 2: by concept code, never by label). Each entry: `concept`, `owner` `{element, key}` or `null`, `users` `[{element, key}]`, `unit` (the owner's sorted `units` joined by `|`, else `null`), `status`. Owners and users come from `conceptOwners(usedElements)` (B2a); sorted by concept, then element, then key.
- `status`: `ok`, `needs-owner` (no owner field), `second-owner` (owner fields in two elements), `unit-conflict` (a user's sorted `units` differ from the owner's), `type-conflict` (a user's `type` differs from the owner's), `possible-duplicate` (see ED-36b). First match wins in that order, after `second-owner`.
- `blockers`: `[{status, concept, message}]` for every status other than `ok` and `possible-duplicate`, plus `missing-element`. Messages name ids, keys and concept codes only.
- The result is plain JSON with stable order (rule 7); calling twice gives equal values.

ED-36 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) report only concepts shared by two or more used elements; a concept inside one element is not listed; (b) `possible-duplicate`: two fields in different used elements with the same label after trimming and lowercasing but different concept codes are reported on the concept of the first, as a warning-like status that never blocks and never merges (design rule 5); (c) `unit` is `null` when there is no owner; (d) owner choices are not an input here, B8 adds them; (e) no schema or validator change.

## Files to read (line ranges)
- `ccis-concept-owners.js` lines 1 to 55 (`conceptOwners`, `latest`; import it, do not copy it).
- `ccis/schemas/pathway.schema.json` lines 100 to 120 (`elements` refs have `id`).
- `test/ccis-concept-owners.test.js` lines 1 to 30 (element fixture style; copy).

## Files to change
New `ccis-compose.js`; new `test/ccis-compose-conflicts.test.js`; CCIS tracker row `ECO-B7`. Studio: `DECISIONS.md` `ED-36`, tracker rows B7 and B8 (B8 then adds owner choices and the build). `index.html` is not rebuilt: nothing in the app imports the module yet.

## Tests (at most 8 cases, `node --test`)
1. No shared concept: `concepts` is `[]`, no blockers.
2. Shared concept, one owner, one user, same type and unit: status `ok`, owner and user named (design example: blood pressure).
3. Same concept, different field keys (`bpSys`, `sbp`): matched by code, not by key.
4. `needs-owner` (both fields name `ownedBy`) and `second-owner` (two owners) are blockers.
5. `unit-conflict` and `type-conflict` are blockers; hand-calculated expected entries.
6. `possible-duplicate` (same label, different codes) is listed and is not a blocker; nothing is merged.
7. `sharedElements` for two pathways using the same element; missing element gives `missing-element`.
8. Determinism: reversed input order of `elements` gives deep-equal output; the input objects are not mutated.
Run only the new file plus `ccis-concept-owners`, `ccis-validator-group`. At most 3 breakage checks.

## Size guard
1 new module (about 80 lines), 1 test. If the module passes about 100 lines, stop and split the duplicate check into B7a.

## Safety notes
Matching is by concept code, never by label (the label hint in ED-36b only reports). No clinical logic, no number, no threshold. Messages are plain, untrusted text on display.

## Context impact
Reads about 5 KB; adds about 80 source lines and a test of about 110 lines. Next: B8 (needs B6a, DONE, and this row).
