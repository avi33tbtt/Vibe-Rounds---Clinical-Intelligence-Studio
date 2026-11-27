# Chunk B13a: rule-pack validator layer 7 (test cases per rule, fixture names)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B13 (DONE). Decisions: ED-6, ED-45 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 8 (layer 7) and 10. Before B13b.

## What exists
`ccis-validator-governance.js` `testVerificationLayer(doc, kind, ctx)` returns `[]` unless `kind === 'analytics-package'`, so a rule pack gets no layer 7 check. It emits V7-004 (no fixture list supplied: warning) and V7-005 (fixture name not in the list: error) for analytics packages. Rule-pack tests (schema, B11): `{ id, rule, fixture, synthetic: true, enables: [{parameter, testOnly: true}], expect: 'fires' | 'does-not-fire' | 'not-evaluated' }`. V4-014 already checks that a test names a real rule and a declared parameter. Layer 7 runs at `ccis-validator.js` line about 618.

## What to build
New module `ccis-validator-rulepack-cases.js` (about 70 lines, pure) exporting `RULEPACK_CASE_RULES` and `rulePackCases(doc, ctx)`; add its rules to `RULES` and make the layer 7 entry add `rulePackCases(first.doc, { fixtures: options.fixtures })` when `kind === 'rule-pack'`. `ccis-validator-governance.js` is not edited.
- V7-004 and V7-005 for a rule-pack test `fixture`, same wording and severities as for analytics packages (the module emits them itself; the rule ids already exist in `RULES`, so they are not added again).
- `CCIS-V7-006` error: a rule lacks a required case. One row per rule and missing case, path `/rules/<i>`, message names the rule id and the missing case.
- `CCIS-V7-007` error: a rule that reads a parameter has no test that leaves its parameter disabled (see ED-45).

ED-45 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) required cases per rule, from design section 10: a test with `expect: fires`, one with `does-not-fire`, one with `not-evaluated`; (b) a rule that reads a parameter also needs a `not-evaluated` test that does not enable that parameter (the "parameter disabled" case); the other `not-evaluated` test must enable every parameter the rule reads (the "input unknown" case), so the two are told apart; (c) a rule's parameters are found by walking its `when` for `{ref:'parameter'}`; (d) the missing cases are errors, also for status `draft`; (e) this layer does not run the rule: that the evaluator reproduces the expected result is checked in B14's test, not here.

## Files to read
`ccis-validator-governance.js` lines 87 to 116 (the V7-004 and V7-005 logic, copy the wording); `ccis-validator-rulepack-shape.js` (whole, 34 lines: walking style); `ccis-validator.js` the layer 7 chain entry (about line 618) and `RULES` tail; `test/ccis-validator-rulepack-shape.test.js` lines 1 to 30 (fixture; copy).

## Files to change
New module and `test/ccis-validator-rulepack-cases.test.js`; `ccis-validator.js` (about 3 lines); rebuilt `index.html`; one pin line in `test/ccis-validator.test.js` (the import list, test 23, same as B12a and B13); CCIS tracker row `ECO-B13a`. Studio: ED-45.
Existing tests that use a rule-pack fixture (`ccis-validator-rulepack*.test.js`, the shape test) have one test with `expect: not-evaluated`, so a pass through all eight layers (rulepack test 1) would now fail at V7-006. If that happens, stop and split: add a row to complete those fixtures. Check this first with a scratch run before writing the module.

## Tests (at most 8 cases)
1. A pack whose rule has the three cases (no parameter) gives no layer 7 row.
2. Missing `fires`, then missing `does-not-fire`, then missing `not-evaluated`: one V7-006 each, naming the case.
3. A parameter rule with the three cases but no disabled case gives V7-007.
4. A parameter rule with all four cases passes.
5. Two tests with `not-evaluated` that both enable the parameter do not count as a disabled case (V7-007).
6. V7-004 warning when no fixture list is supplied; V7-005 error when a fixture name is not in the list; none when it is.
7. A rule with no test at all gives all its missing-case rows.
8. V7-006 and V7-007 are in `RULES`; `validateArtefact` reports them at layer 7 and layer 8 is skipped.
Run the new file plus `ccis-validator-rulepack`, `ccis-validator-rulepack-checks`, `ccis-validator-rulepack-shape`, `ccis-validator-governance`, `ccis-validator.test.js`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module (under 100 lines), 1 test, 3 edited lines, one pin line. If an existing rule-pack fixture needs more than adding tests to it, or any other existing test needs a change, stop and split.

## Safety notes
Form check only: it counts declared cases, it does not run rules and it does not judge whether a case is clinically sensible. Messages name rule ids and case names only.

## Context impact
Reads about 7 KB; adds about 70 module lines and 100 test lines. Next: B13b.
