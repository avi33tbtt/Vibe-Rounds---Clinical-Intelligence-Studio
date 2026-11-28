# Chunk B18a: contract fixtures, part 2 (pathway and rule pack)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B18, B16a, B17 (all DONE). Decisions: ED-58 (new, OPEN working choice below). Next: B19.

## What exists
`ccis/fixtures/contract/` (B18): `core-vitals.element.json` (owns `systolic-pressure` as `bpSys`, and `hr`), `htn-core.element.json` (`sbp` uses it via `ownedBy`; `htnNote`), `contract.group.json`, `second-owner.element.json`, `expected.json`. SDK builders `pathway()` (with `entryForm` and `contexts`, B16a) and `rulePack()` (B17). `bindRuleInputs({ pack, context })` joins a `ccis-context/1` object to inputs by a `local` code (ignores entered-in-error; `latest` gives the record id in `refs`, no records give `absent`). `evaluateRulePack({ pack, bound, parameters, asOf })` returns prompts (rule id, pack version, checksum, kind, severity, message, record refs, no values). Known quirk (ED-38): a single `information` stage needs a leaving transition (CCIS-V5-006), so use one `report` stage.

## What to build (ED-58, working choice)
Three new files plus a hand-calculated one, same folder, same sorted-key LF form as B18, written by the SDK builders from a throwaway script (not committed):
- `contract.pathway.json`: id `contract.pathway`, elements `core.vitals` and `htn.core` by range `^0.1.0`, one `report` stage, `entryForm` with one section `vitals`: an element item `core.vitals` (`rowGroups` `[["bpSys","hr"]]`), an element item `htn.core` with `fields` `["htnNote"]` only (the shared `sbp` is not placed twice), and a section item `core.problems`; `contexts` one entry `hypertension` with both elements.
- `contract.rule-pack.json`: id `contract.rules`, class `data-quality`, scope the pathway, one input `sbp` (concept `systolic-pressure`, read `latest`), no parameters, one rule `sbpMissing` (kind `missing`, severity `info`, `when` not is-present `sbp`, plain template), tests: one `fires`, one `does-not-fire`.
- `contract.contexts.json`: three small `ccis-context/1` objects named `absent` (no records), `present` (one `local` `systolic-pressure` record), `enteredInError` (the same record with status `entered-in-error`).
- `expected-rules.json`: HAND-CALCULATED before any run: for each context the bound `sbp` (`absent` with no refs; `present` with the record id; `absent` for entered-in-error) and the prompts (one for `absent`, none for `present`, one for `enteredInError`, each with rule id `contract.rules.sbpMissing`, empty `recordRefs`, the `asOf` time passed in). The pack checksum is not typed in: the test checks it equals `checksumOf` of the canonical pack.

## Files to read
`test/ccis-rules-bind.test.js` lines 1 to 25 (observation and context shape); `test/ccis-rules.test.js` lines 14 to 35 (prompt shape); `ccis/fixtures/contract/expected.json` (style); the B18 test file lines 1 to 25 (helpers). Not the modules.

## Files to change
New fixtures (4 files); new `test/ccis-contract-fixtures-2.test.js`; CCIS tracker row `ECO-B18a`. No module change, no dependency.

## Tests (at most 8 cases)
1. The pathway passes `validateArtefact` with no error row, with the two elements as `known`.
2. Pathway file equals `pathway()` output (`compareRoutes`); same for the rule pack with `rulePack()`.
3. The `entryForm` places `sbp` nowhere (hand-calculated section list from `expected-rules.json` or inline).
4. The pack passes `validateArtefact` with the pathway as `known`; it holds no number and no parameter.
5. Bound inputs for the three contexts equal the expected ones.
6. Prompts for the three contexts equal the expected ones; the checksum equals `checksumOf` of the canonical pack.
7. Pack tests cover `fires` and `does-not-fire`; every `rule` named in a test exists.
8. Files are synthetic, LF-only, sorted-key; no identity member.
Run the new file, the B18 test, `ccis-rules.test.js`, `ccis-rules-bind.test.js`, `ccis-rule-pack-schema.test.js`. At most 3 breakage checks (drop a record, change the expected prompt, put `sbp` back in the `htn.core` item).

## Size guard
4 files (each under 3 KB), 1 test, at most 8 cases. If the validator refuses the pathway or pack in a way that needs a module change, STOP and report; do not edit modules in this row.

## Safety notes
No threshold, number, dose or clinical advice. The rule only reports that a record is missing. Demo text only.

## Context impact
Reads about 5 KB; adds about 10 KB of fixtures and 90 test lines.
