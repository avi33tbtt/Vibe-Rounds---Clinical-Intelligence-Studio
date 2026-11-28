# Chunk B14: `ccis-rules.js`, pure rule-pack evaluator (core, over pre-bound inputs)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B13c (DONE). Decisions: ED-6, ED-48 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 4, 5, 6 and 10. Before B14a and B15.

## What exists
`ccis-exec.js`: `prepareAnalytic({analytic, checksum, runtime})` and `evaluateOutput({prepared, outputId, inputs, parameters})`, a pure three-valued interpreter of all 21 whitelist operators (lines 102 to 135 and 237 to 262 read). It checks structure at prepare, not input types. Its inputs are single-valued: `{state, value?}` per input key (D-048: several records are `conflicting`). A rule-pack input is `{id, concept, read: latest|all|count}`; a rule is `{id, kind, severity, when, message:{template}}`; parameters are `{id, kind, enabledByDefault:false}`. Canonical JSON and checksum: `canonicalize`, `checksumOf` in `ccis-compile.js`. The context binder `ccis-bind.js` finds records by element and field, not by concept, so it is not reused here.

## What to build
New pure module `ccis-rules.js` (about 90 lines) exporting `RULES_ENGINE` and `evaluateRulePack({ pack, bound, parameters, asOf })`.
- `bound`: per pack input id, `{ state, value?, refs: [recordId] }`. The caller (B14a) builds it from a context; this row takes it as given. `parameters`: per parameter id `{ enabled, value? }`, missing means disabled. `asOf`: an ISO instant string, the only time source.
- Adapter: build a synthetic analytic from the pack (inputs keyed by input id, one output per rule, one expression tree per rule) and run it through `prepareAnalytic` and `evaluateOutput`. No second interpreter, no change to `ccis-exec.js`.
- Result: `{ ok: true, packChecksum, asOf, prompts: [...], notEvaluated: [ruleId] }` or `{ ok: false, refusal }` (fixed sentence, never echoes input). True fires one prompt; false gives nothing; unknown, or a rule reading a disabled parameter, goes to `notEvaluated`. Never zero or false substituted.
- Prompt (design section 6): `ruleId` (`packId.ruleId`), `packVersion`, `packChecksum`, `kind`, `severity`, `message` (template with `{concept}`, `{parameter}`, `{count}` filled), `recordRefs` (sorted, de-duplicated refs of the inputs that rule reads; never values), `asOf`. Output ordered by rule id.
- `read: count` binds as an integer (absent when 0) by the caller; `read: all` is not supported by single-valued inputs, so a rule that reads it is `notEvaluated` until B9a.

ED-48 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) reuse the executor through a synthetic analytic; (b) inputs arrive pre-bound, binding by concept is row B14a; (c) `read: all` rules are `notEvaluated`; (d) refusals are fixed sentences; (e) the pack checksum is `checksumOf(canonicalize(pack))`.

## Files to read
`ccis-exec.js` lines 287 to 330 (result shape); `ccis-compile.js` exports (`grep -n "^export"`); `test/ccis-validator-rulepack-governance.test.js` lines 1 to 25 (pack fixture; copy).

## Files to change
New `ccis-rules.js` and `test/ccis-rules.test.js`; CCIS tracker row `ECO-B14`. Studio: ED-48, new row B14a. Not wired into the app, so no `index.html` change.
First check with a scratch run that the adapted pack passes `prepareAnalytic` and a `time-diff` rule and a `state-of` rule evaluate. If the adapter needs an executor edit, stop and split.

## Tests (at most 8 cases)
1. A `missing` rule on a count input of 0 fires one prompt with the expected fields and `recordRefs: []`.
2. The same rule with count 1 gives no prompt and is not in `notEvaluated`.
3. An unknown input: `notEvaluated`, no prompt.
4. A rule on a disabled parameter: `notEvaluated`; enabled with a value it evaluates.
5. `read: all` rule: `notEvaluated`.
6. Message placeholders filled; no input value appears in the output.
7. Same input twice gives byte-identical canonical output.
8. A tampered or malformed pack gives `ok: false` with a fixed sentence.
Run the new file plus `ccis-exec*.test.js`. At most 3 breakage checks.

## Size guard
1 module, 1 test. If it grows past about 120 lines, split the message rendering into B14c.

## Safety notes
A prompt describes an information gap and never orders anything. No values in output. Synthetic only.

## Context impact
Reads about 5 KB; adds about 90 module lines and 100 test lines. Next: B14a.
