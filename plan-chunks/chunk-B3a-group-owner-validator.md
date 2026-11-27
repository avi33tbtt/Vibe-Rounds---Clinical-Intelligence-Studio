# Chunk B3a: element-group validator, owner and concept key

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B3 (DONE). Decisions: ED-12, ED-30, ED-31 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` section 6. Before B4.

## What to build
Edit the existing pure module `ccis-validator-group.js` (4.3 KB, 56 lines of logic): add three layer 5 rules to `GROUP_RULES` and about 25 lines to `groupChecks`. Ids continue after the last used (`V5-029` is B2a's).
1. `CCIS-V5-030` error: `owner` is not one of the concept's listed `fields` (compare element id and field key). Path `/sharedConcepts/<i>/owner`. Needs no member resolution.
2. `CCIS-V5-031`: a `same-concept` entry has no `owner`. Error when `manifest.standard.version` is 0.2.0 or later, warning at 0.1.0 (same switch as V5-024 in `ccis-validator-concept.js`; copy the 3-line version helper, do not import). A `related-not-merged` entry needs no owner (separate facts, nothing to own); if it has one, V5-030 still applies.
3. `CCIS-V5-032` error: `concept` matches neither the legacy camelCase key `^[a-z][A-Za-z0-9]*$` nor the local key format `^[a-z0-9][a-z0-9._-]{0,63}$` (the schema accepts a wider set on purpose, B3).
Messages name ids, keys and concept codes only.

ED-31 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) `related-not-merged` needs no owner; (b) `ccis-diabetes-ckd-group.test.js` line 60 expects no diagnostics at all after flipping a concept to `same-concept`; at 0.1.0 that now gives the V5-031 warning, so that one assertion changes to "no error rows" (`filter((d) => d.severity === 'error')`), which keeps its meaning (passes the validator, checksum changes). The other option, no V5-031 row at 0.1.0, would leave the test untouched but differs from the row text; say which you prefer. (c) The owner field's own `ownedBy` and concept agreement with the element field stay out of this row (B1e and B2a cover element fields).

## Files to read (line ranges)
- `ccis-validator-group.js` lines 1 to 20 and 20 to 56 (rule list; the `sharedConcepts.forEach` loop).
- `ccis-validator-concept.js` lines 14 to 18 and 28 to 33 (the `major` helper and `strict` switch; copy).
- `test/ccis-validator-group.test.js` lines 1 to 28 (fixtures; copy).
- `test/ccis-diabetes-ckd-group.test.js` lines 58 to 63 (the one assertion to change).

## Files to change
`ccis-validator-group.js` (edit); new `test/ccis-validator-group-owner.test.js`; one assertion in `test/ccis-diabetes-ckd-group.test.js`; `index.html` rebuilt (`npm ci` first if `esbuild` is missing); CCIS tracker row `ECO-B3a`. Studio: `DECISIONS.md` `ED-31`. No schema change, no wiring change (`groupChecks` is already called at layer 5).

## Tests (at most 8 cases, `node --test`)
1. A same-concept entry whose owner is a listed field gives no V5-030 to 032 row.
2. An owner that is not a listed field (wrong element, then wrong field) gives V5-030 at the owner path (two assertions).
3. A same-concept entry with no owner: warning at 0.1.0, error at 0.2.0 (V5-031).
4. A related-not-merged entry with no owner gives no row; with a bad owner it gives V5-030.
5. `concept` values `bpSys`, `systolic-pressure` pass; `Bad-Key`, `-x` and a 65-character code give V5-032.
6. Unresolved members still give V5-030 to 032 where they need no resolution (no crash, no guess).
7. The `diabetes-ckd` example and the existing group tests give no new error row.
8. A group with two same-concept entries, one good and one with no owner, gives exactly one V5-031 at the right index.
Run only the new file plus `ccis-validator-group`, `ccis-diabetes-ckd-group`, `ccis-element-group-schema`, `ccis-group-owner-schema`, `ccis-market-diabetes-ckd` and `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module edit (about 25 lines), 1 test, 1 changed assertion, 1 bundle. If the edit passes about 40 lines or any other existing test needs a change, stop and split.

## Safety notes
Identity only: no numbers, bands or thresholds. Ownership is never inferred from a label or a key. Messages are plain, untrusted text on display.

## Context impact
Reads about 6 KB of source; adds about 25 source lines and a test of about 90 lines. Next row after this: B4, which is re-checked against V5-009 to 013 and V5-030 to 032 before it is built (ED-30).
