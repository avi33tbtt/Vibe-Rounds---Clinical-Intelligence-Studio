# Chunk B19: standard version 0.2.0 accepted

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B18, B18a, B1b (all DONE). Decisions: ED-60 (new, OPEN working choice below). Next: B20 (docs), then B21 (you tag).

## What a probe showed (nothing written, 2026-11-28)
- The contract fixtures, with only `manifest.standard.version` changed to `0.2.0` in memory, pass every layer with no error row: element, element-group, pathway and rule pack. `SUPPORTED_STANDARD_MAJORS = [0]` (`ccis-validator-semantic.js` line 43) already admits 0.2.0.
- Strict concept and group checks already switch on at minor 2 or more (`ccis-validator-concept.js` line 27, `ccis-validator-group.js` line 29).
- `ccis-compat.js` (line 288 to 291) checks only the major against the runtime's supported majors; `COMPAT_VERSION` is `0.1.0`.
- `STANDARD_VERSION = '0.1.0'` is stamped into records by `ccis-import.js` (line 23), `ccis-validation-record.js` (line 16) and `ccis-demo-loader.js` (line 24).

## What to build (ED-60, working choice)
Acceptance is proved, not re-implemented. The gates already work, so B19 is mostly one test plus the smallest edit the test shows is needed:
1. New `test/ccis-standard-0-2-0.test.js` (at most 8 cases) that takes the B18 and B18a fixtures, sets the standard to `0.2.0` in memory and checks: (a) validator verdicts and error rows equal the 0.1.0 verdicts; (b) strict mode is on at 0.2.0: a field with a malformed `concept.code` is an error at 0.2.0 and a warning at 0.1.0 (CCIS-V5-022); (c) compat reports no `CMP-3-05` for 0.2.0 and does report it for `1.0.0`; (d) the importer still reads a 0.1.0 envelope and the context it builds is unchanged; (e) every existing example under `ccis/examples/` still has no error row (reuse the loop in `ccis-validator-semantic.test.js`, lines to be found by `grep -n "examples"`).
2. Stamps stay `0.1.0` (records and their checksums are pinned in many fixtures). Only if the test shows a gate that refuses 0.2.0, make the smallest edit and say so.
3. No new rule id and no warning for "0.1.0 artefact uses 0.2.0 members" in this row. That is B19a (optional, unlock only if you say so): it would add CCIS-V5-054 and touch the rule catalogue tests.

## Files to read
`ccis-validator-semantic.js` lines 556 to 570; `ccis-compat.js` lines 280 to 295; `ccis-import.js` lines 20 to 30 and 55 to 70 (by `sed -n`); `test/ccis-validator-semantic.test.js` (grep for the examples loop); the B18 test lines 1 to 25 (helpers). Not the rest of the modules.

## Files to change
New test only (target). CCIS tracker row `ECO-B19`. If a gate refuses 0.2.0, one small edit in that module, named in the notes.

## Tests
New file, at most 8 cases. Also run: `ccis-validator-semantic`, `ccis-validator-concept`, `ccis-compat`, `ccis-import`, `ccis-contract-fixtures` and `ccis-contract-fixtures-2` tests. At most 3 breakage checks (set `SUPPORTED_STANDARD_MAJORS` to `[]`, loosen the strict test to `min >= 3`, change a fixture version to `1.0.0` and see the verdict flip). Known failures on a clean checkout (`ccis-rule-pack-schema` case 8, `4.2d.1`) stay as they are; say so.

## Size guard
One test file, at most 8 cases, at most one 10-line module edit. If a gate needs more than that, STOP and split off the part (`B19b`).

## Safety notes
No clinical content. Fixtures stay synthetic. Do not change any pinned checksum or the `CHANGELOG.md`; amendments are B20 and need your approval.

## Context impact
Reads about 6 KB; adds about 80 test lines.
