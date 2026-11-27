# Chunk B4: element-group validator, duplicates and single-member concepts

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B3, B3a (both DONE). Decisions: ED-12, ED-30, ED-32 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` section 6. Before B7.

## Re-check against what exists (ED-30c, done 2026-11-28)
Probed the current validator with the diabetes-ckd group. Already covered, nothing to build:
- Match by concept, never by label: the schema has no label member and no rule reads labels; V5-012 flags undeclared key overlap only. Covered by existing tests; B4 adds one assertion that a near-identical label with a different key gives no row.
- Unresolved member or shared concept: members are resolved at layer 3 (`ccis-validator.js` line 415; V3-004 and V3-005); a concept naming a non-member is V5-009; an undefined field is V5-010; same concept in two entries is V5-013; same-concept with unequal type or units is V5-011; owner rules are V5-030 to 032.
- One field listed in a concept: the schema refuses it (`minItems: 2`, V2-004).
Gaps found (no diagnostic today):
1. The same member element listed twice in `members`.
2. The same `concept` code used by two `sharedConcepts` entries ("no duplicate keys").
3. A concept whose fields all belong to one member element (nothing is shared; V5-012 fires as a side effect, but the cause is not named).

## What to build
Edit `ccis-validator-group.js` (84 lines): add three layer 5 rules and about 20 lines to `groupChecks`. Ids continue after V5-032.
1. `CCIS-V5-033` error: a member element id appears more than once. Path `/members/<i>/element`, reported at the second and later positions.
2. `CCIS-V5-034` error: a `concept` code is used by more than one `sharedConcepts` entry. Path `/sharedConcepts/<i>/concept`, at the second and later positions.
3. `CCIS-V5-035` error: the fields of one shared concept name fewer than two different member elements. Path `/sharedConcepts/<i>/fields`. Skipped when a field names a non-member (V5-009 already applies).
Messages name ids and concept codes only.

ED-32 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) all three are errors at every standard version, since none exists in 0.1.0 artefacts that pass today; the shipped diabetes-ckd group and its pinned checksum are unchanged; (b) V5-035 allows two fields of the same element only when a second member also appears; (c) no new schema rule.

## Files to read (line ranges)
- `ccis-validator-group.js` lines 1 to 84.
- `test/ccis-validator-owner.test.js`-style fixtures: copy the `group()` fixture from `test/ccis-validator-group-owner.test.js` lines 1 to 30.

## Files to change
`ccis-validator-group.js` (edit); new `test/ccis-validator-group-duplicates.test.js`; `index.html` rebuilt (`npm ci` first); CCIS tracker row `ECO-B4`. Studio: `DECISIONS.md` `ED-32`. No schema, wiring or existing-test change.

## Tests (at most 8 cases, `node --test`)
1. Duplicate member gives V5-033 at the second position only.
2. Duplicate concept code gives V5-034 at the second entry only.
3. A concept naming one member twice gives V5-035; naming two members gives none.
4. V5-035 is skipped when a field names a non-member (V5-009 only).
5. A near-identical label with a different key and no shared concept gives no V5-033 to 035 row.
6. The diabetes-ckd example gives no new row.
7. Unresolved members: V5-033 and V5-034 still fire, no crash (call `groupChecks` with a `resolve` that returns `missing`).
8. Hostile-looking ids (angle brackets, quotes) appear only inside message text, no crash.
Run only the new file plus `ccis-validator-group`, `ccis-validator-group-owner`, `ccis-diabetes-ckd-group`, `ccis-element-group-schema`, `ccis-group-owner-schema`, `ccis-market-diabetes-ckd`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module edit (about 20 lines), 1 test, 1 bundle. If the edit passes about 40 lines or an existing test needs a change, stop and split.

## Safety notes
Identity only: no numbers, bands or thresholds. Nothing is inferred from labels or keys. Messages are plain, untrusted text on display.

## Context impact
Reads about 5 KB of source; adds about 20 source lines and a test of about 80 lines. Next row after this: B5 (pathway `entryForm` schema) or B7 (compose part 1), whichever comes first in table order.
