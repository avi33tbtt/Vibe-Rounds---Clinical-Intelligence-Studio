# Chunk B18: contract fixtures, part 1 (elements, group, second owner, compose)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B16 (DONE; B16a and B17 also DONE). Decisions: ED-57 (new, OPEN working choice: split). Next: B18a (pathway and rule pack), then B19.

## Why split
The full B18 is 2 elements, a group, a pathway with `entryForm`, a rule pack, a refusal case and hand-calculated compose and rule outputs: about 9 files plus a test with two engines. Too big for one row. B18 takes the element side; B18a (new row) takes the pathway and the rule pack. B19 then needs B18a.

## What exists
`ccis/fixtures/` holds `compile`, `diabetes-ckd`, `schemas`, `snake`. The element builder (`ccis-sdk-element.js`) supports a field `concept` binding and `ownedBy` (B2c). `ccis-concept-owners.js` gives layer 5 rules CCIS-V5-028 (a concept owned by two elements; error when strict ownership is set) and CCIS-V5-029 (no owner). `conflictReport(pathways, elements)` is in `ccis-compose.js`; the market refuses a second owner as M-014. The B8a test (`test/ccis-compose-pathway.test.js` lines 8 to 14) shows how a synthetic element is made with `concept()` and `ownedBy`.

## What to build
New folder `ccis/fixtures/contract/`, all synthetic, local codes only:
- `core-vitals.element.json`: fields `bpSys` (concept `systolic-pressure`, the owner) and `hr` (concept `heart-rate`).
- `htn-core.element.json`: field `sbp` with concept `systolic-pressure` and `ownedBy` `core.vitals` / `bpSys`; field `htnNote`.
- `contract.group.json`: built with `elementGroup` (B16) from the two elements; shared concept `systolic-pressure` with `owner`, resolution `same-concept`.
- `second-owner.element.json`: element `dup.vitals` that also owns `systolic-pressure` (no `ownedBy`): the refusal case.
- `expected.json`: HAND-CALCULATED, written before running anything: the concept to owner map, the users of each concept, the conflict report for the pair (one shared concept, no conflict), and for the refusal case the rule ids expected (V5-028 as warning, error when strict; M-014 from the market).

## Files to read
`test/ccis-compose-pathway.test.js` lines 8 to 14; `ccis-concept-owners.js` lines 1 to 12 and the export list (`grep -n "^export"`); `ccis-compose.js` lines 1 to 20; `grep -n strictOwnership ccis-validator.js` and its callers for how strict is passed; `ccis-market.js` lines 175 to 190. Not whole files.

## Files to change
New fixture folder (5 files); new `test/ccis-contract-fixtures.test.js`; CCIS tracker row `ECO-B18`. No module change, no dependency. A generator script is not needed: the files are committed as written.

## Tests (at most 8 cases)
1. Each of the three good artefacts passes `validateArtefact` with no error row (the group with the elements as `known`).
2. The group file equals the `elementGroup` builder output (`compareRoutes`).
3. The owner map from the concept-owners module equals `expected.json` (one owner of `systolic-pressure`: `core.vitals`).
4. The conflict report equals `expected.json` byte for byte (canonical JSON).
5. Adding `second-owner.element.json` gives V5-028 as a warning, and as an error under strict ownership.
6. The market refuses the same pair with M-014 naming ids and concept codes only.
7. Every fixture is synthetic: no member holds a name, date of birth or free narrative; each file has a LF-only canonical form.
8. Existing example checksums are unchanged (the snake and diabetes-ckd tests stay green).
Run the new file, `ccis-concept-owners.test.js`, `ccis-compose-group.test.js`, `ccis-diabetes-ckd-group.test.js`. At most 3 breakage checks (remove `ownedBy`, add a second owner, change `expected.json`).

## Size guard
5 JSON files (each under 3 KB) and 1 test of at most 8 cases. More: stop and split.

## Safety notes
No clinical thresholds, doses or targets in any fixture. Concept codes are local plain words. Labels are demo text only.

## Context impact
Reads about 5 KB; adds about 14 KB of fixtures and 90 test lines.
