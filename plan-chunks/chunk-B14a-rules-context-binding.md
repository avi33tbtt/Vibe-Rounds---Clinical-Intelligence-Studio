# Chunk B14a: bind a canonical context to rule-pack inputs by concept

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B14 (DONE). Decisions: ED-6, ED-9, ED-48, ED-49, ED-50 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 4 and 6. Before B15 and D20.

## What exists
`ccis-rules.js` (B14) takes `bound`: per pack input id `{state, value?, refs:[recordId]}`, with `value` in executor form (`{t:'number',v,unit}`, `{t:'instant',ms}`, `{t:'text',v}`, `{t:'bool',v}`); `state` without `value` while `present` reads as unknown. A pack input is `{id, concept, read: latest|all|count}`. The context (`ccis-context/1`, schema `context-envelope.schema.json`) has `observations[]`: `{id, concept:{id, codes:[{system, code, verification}]}, kind, value, effective:{value, zone}, status, state?}`. `ccis-bind.js` finds records for an analytic input by `concept.id === element.field-kebab` (line 102), refuses kinds other than `datetime` and `code` (B-005), and keeps `instantOf` (line 75) private. Concept keys are `local` code bindings (ED-9), so a context observation carries the key as a code with system `local`. No fixture in the repo uses that yet.

## What to build
New pure module `ccis-rules-bind.js` (about 70 lines) exporting `bindRuleInputs({ pack, context })` -> `{ ok: true, bound }` or `{ ok: false, refusal }` (fixed sentence, echoes nothing). Add `export` to `instantOf` in `ccis-bind.js` (one word, no logic change).
Per pack input, the records are the observations whose `concept.codes` hold `{system: 'local', code: <input.concept>}` and whose `status` is not `entered-in-error`.
- No record: `{state:'absent', refs:[]}`.
- `read: count`: `{state:'present', value:{t:'number', v:n, unit:'1'}, refs: sorted ids}`.
- `read: latest`: one record, or the record with the unique greatest `effective` instant, is the record read; its `state` (if any) is passed through. Several records with a tie or an unreadable time: `{state:'conflicting', refs: all ids}`. Value: kind `datetime` gives `instant` (via `instantOf`), kind `code` gives `text`; any other kind is `present` with no value (unknown to the rule, never zero or false).
- `read: all`: no entry (B14 fills unknown; `all` waits for B9a).
- Refs are record ids only, never values. Events are not read in this row.

ED-50 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) match by `local` code only, never by label, `concept.id` or similar key; (b) `latest` means unique greatest effective instant, else `conflicting`; (c) value follows the binder (datetime, code), other kinds unreadable; (d) one `export` added to `ccis-bind.js`; (e) found gap, not built here: a `stale` rule cannot read `asOf` (it is not an input), so time since the last record cannot be written yet. Row B14b decides that.

## Files to read
`ccis-bind.js` lines 60 to 97 (TIME, `instantOf`); `ccis-rules.js` lines 33 to 50; `test/ccis-rules.test.js` lines 1 to 30 (pack fixture; copy).

## Files to change
New `ccis-rules-bind.js`, `test/ccis-rules-bind.test.js`; `ccis-bind.js` (`export`); CCIS tracker row `ECO-B14a`. Studio: ED-50, new LOCKED row B14b. Not wired into the app. Check first with a scratch run that adding `export` leaves `ccis-bind*.test.js` and `ccis-app-build*.test.js` green; if the build test pins the export list, stop and split.

## Tests (at most 8 cases)
1. `count`: two matching records give `v: 2` and sorted refs; none gives `absent` with `refs: []`.
2. An `entered-in-error` record is ignored.
3. `latest` picks the unique greatest effective instant; refs hold its id only.
4. `latest` with a tie, or an unreadable time, gives `conflicting` with all ids.
5. A record's explicit `state` (for example `unknown`) passes through; a `quantity` record is present without a value.
6. Same code under another system, or `concept.id` equal to the key, does not match.
7. Chained into `evaluateRulePack`: a `missing` rule fires on an empty context; a time-difference rule on two datetime inputs fires with the expected refs and does not leak values.
8. A context that is not `ccis-context/1` gives a fixed sentence.
Run the new file plus `ccis-rules.test.js`, `ccis-bind*.test.js`, `ccis-app-build*.test.js`. At most 3 breakage checks.

## Size guard
1 module, 1 test, 1 word edit. Over about 100 lines: split `latest` into B14c.

## Safety notes
Refs are record ids, no values, no identity. A concept key match is a local code, not clinical meaning. Synthetic fixtures only.

## Context impact
Reads about 5 KB; adds about 70 module lines and 110 test lines. Next: B15 (B14b stays locked).
