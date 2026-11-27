# Chunk B9: executor operators `min`, `max`, `coalesce` (`first` and `last` split to B9a)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B0 (DONE). Decisions: ED-39 (new, OPEN working choices). CCIS finding F-9, CCIS `DECISIONS.md` D-115, D-121 (read those two rows only). Before E11 and B14.

## Why a split
Inputs are single values: several records for one field are `conflicting` (D-048). `first` and `last` need a series input, so with today's input model they could only duplicate `coalesce`. They move to new row B9a, optional, behind a series input row (D-121 still stands for them).

## What to build
Three variadic operators in `ccis-exec.js`, three-valued like `add`:
1. `min`, `max` (2 or more arguments): all numbers with one unit, else `X-010` (the `add` rule). Any unknown argument gives the merged unknown (`mergeBecause`). Result keeps the unit. Value and unit are never converted.
2. `coalesce` (2 or more arguments): the first argument that is not unknown wins; all unknown gives the merged unknown. All arguments are evaluated (no short-circuit, D-116). Known arguments must be one type (and one unit for numbers), else `X-010`. A literal argument is refused at prepare time with new refusal `X-018` (a literal would turn an unknown into a value, for example `coalesce(x, false)`). References (inputs, parameters) and sub-expressions are allowed.
3. Operator-set bump: `OPERATOR_SET.version` becomes `0.2.0` (additive; the major check in prepare is unchanged). A package that uses one of the three and declares an operator-set minor below 2 is refused at prepare with new refusal `X-017`. Packages that do not use them are untouched.
4. Schema: add `min`, `max`, `coalesce` to the `op` enum in `ccis/schemas/analytics.schema.json` line 257; regenerate validators with `npm run build`.
5. `ccis-demo-exec.js` is unchanged: it keeps refusing what it does not support.

ED-39 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) `first` and `last` are not added; B9a, optional, unlocks only if you say so; (b) `min` and `max` need the same unit and refuse otherwise, never convert; (c) `coalesce` refuses literal arguments (X-018); (d) operator set `0.2.0`, X-017 for a package that uses the new operators under an older set; (e) docs, SPEC and README line "18 operators" are amended in B20, not here.

## Files to read (line ranges)
- `ccis-exec.js` lines 27 to 45 (ops, ARITY), 46 to 62 (refusal table), 108 to 125, 160 to 180 (prepare checks), 335 to 372 (evaluator).
- `ccis/schemas/analytics.schema.json` lines 250 to 262.
- `test/ccis-exec.test.js` lines 1 to 60 and the `add` cases (fixture style; copy).

## Files to change
`ccis-exec.js` (about 30 lines); schema (one enum line); regenerated validators; `index.html` rebuilt (`npm ci` first); `test/ccis-exec.test.js` only for the pinned facts (18 operators, 21 now; set `0.1.0`, `0.2.0` now; test-only change, as D-104); new `test/ccis-exec-min-max-coalesce.test.js`; CCIS tracker row `ECO-B9`. Studio: `DECISIONS.md` `ED-39`, tracker rows B9, B9a, E11.

## Tests (at most 8 cases, `node --test`)
1. `min` and `max` of two and of three same-unit numbers (hand-calculated).
2. `min` with an unknown argument gives unknown with the reason; with mixed units gives `X-010`.
3. `coalesce` returns the first known value; the second known is ignored.
4. `coalesce` with all unknown gives unknown; type mismatch gives `X-010`.
5. `coalesce` with a literal argument is refused at prepare (`X-018`).
6. A package using `min` and declaring operator set `0.1.0` is refused (`X-017`); declaring `0.2.0` prepares.
7. Wrong argument counts give `X-003`.
8. Same input twice gives identical results; existing snake analytics still prepare and run unchanged.
Run only the new file, `ccis-exec`, `ccis-run`, `ccis-demo-record`, `ccis-sample-gap-analytic`, and the validator tests the build touches. At most 3 breakage checks.

## Size guard
About 30 lines in `ccis-exec.js`, 1 enum line, 1 new test, 2 pinned lines in an existing test. If any golden record, checksum or fixture changes, stop and split.

## Safety notes
Unknown never becomes a value: `coalesce` cannot turn missing data into a constant. No thresholds. Operators compute on values already in the context; nothing here authorises an action.

## Context impact
Reads about 10 KB; adds about 30 source lines and a test. Next: B10.
