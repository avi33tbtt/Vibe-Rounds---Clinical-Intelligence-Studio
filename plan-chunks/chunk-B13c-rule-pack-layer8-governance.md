# Chunk B13c: rule-pack validator layer 8 (governance)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B13a, B13b (DONE). Decisions: ED-2, ED-46, ED-47 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 7, 8 (layer 8) and 9. Before B14.

## What exists
`ccis-validator-governance.js` `governanceLayer(doc)` runs for every artefact kind: V8-001 to V8-003 compare `manifest.status` with `manifest.validation` and an evidence statement. A rule pack has no evidence member, so a validated pack only gets the V8-003 warning. B13b added the optional `governance` object `{author, owner, reviewers:[{name, version, date}]}`. The chain entry is `ccis-validator.js` line 621: `['governance', 'test-verification', () => governanceLayer(first.doc)]`. Ids V8-004 and up are free. `ccis-validator-governance.js` is not edited.

## What to build
New module `ccis-validator-rulepack-governance.js` (about 60 lines, pure, same style as `ccis-validator-rulepack-cases.js`) exporting `RULEPACK_GOVERNANCE_RULES` and `rulePackGovernance(doc)`. Add its rules to `RULES` and append its rows to the layer 8 entry when `kind === 'rule-pack'`. It compares claims in the file and never judges whether a claim is true.
- `CCIS-V8-004` error: a reviewer record names the author (names compared after trim, whitespace collapse and lower case). One row per matching record, path `/governance/reviewers/<i>/name`.
- `CCIS-V8-005` error: class `clinical-prompt` and the owner is missing or is the word `unassigned` (any case). Path `/governance/owner`. A `data-quality` pack may stay unassigned (design section 7).
- `CCIS-V8-006` error: status `reviewed`, `validated` or `conditionally-validated` and no reviewer record names the pack's exact `manifest.version`. Path `/manifest/status`.
- `CCIS-V8-007` error: status beyond `draft` (any status except `draft`) and no `governance.author`. Path `/manifest/status`.
The `validated` scope (`manifest.validation`, schema-required) and its version match (V8-002) already exist; this row adds nothing there.

ED-47 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) the four checks above; (b) `submitted` and `in-review` need an author but no reviewer record; (c) author against reviewer only, not owner against reviewer; (d) a `draft` pack is never refused for missing governance; (e) a name match is a claim check, not proof of independence (ED-19).

## Files to read
`ccis-validator-rulepack-cases.js` (whole, 45 lines: style); `ccis-validator.js` lines 66 to 78 (`RULES`) and 616 to 622 (layer chain); `test/ccis-validator-rulepack-cases.test.js` lines 1 to 30 (fixture; copy); `test/ccis-validator.test.js` line 395 (import-list pin).

## Files to change
New module and `test/ccis-validator-rulepack-governance.test.js`; `ccis-validator.js` (about 3 lines); rebuilt `index.html` (`node scripts/build-app.js`); one pin line in `test/ccis-validator.test.js` (import list, as in B12a and B13); CCIS tracker row `ECO-B13c`. Studio: ED-47.

## Tests (at most 8 cases)
1. A draft pack with no `governance` gives no layer 8 row from the new module.
2. A reviewer named like the author (different case and spacing) gives V8-004; a different name does not.
3. `clinical-prompt` with no owner, then `unassigned`, gives V8-005; a named owner does not; `data-quality` with `unassigned` does not.
4. Status `reviewed` with no reviewer record for this version gives V8-006; a record for another version still does.
5. Status `reviewed` with a record for this exact version gives no V8-006.
6. Status `submitted` without an author gives V8-007; with an author it does not.
7. A clean `reviewed` pack (author, owner, matching reviewer) gives no new row.
8. Through `validateArtefact`: the rows appear in the `governance` layer.
Run the new file plus `ccis-validator.test.js`, `ccis-validator-rulepack*.test.js`, `ccis-rule-pack-*-schema.test.js`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module, 1 test, 1 small edit, 1 pin line. Check first with a scratch run that the existing rule-pack fixtures (all `draft`) still pass all eight layers. If any existing test needs more than the pin line, stop and split.

## Safety notes
Compares names in a file. With one maintainer this shows intent and catches mistakes; it does not prove independence (design section 9, ED-19). No clinical content.

## Context impact
Reads about 6 KB; adds about 60 module lines and 90 test lines. Next: B14.
