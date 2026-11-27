# Chunk B11: rule-pack schema part 2 (rules, messages, provenance, tests)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B10 (DONE), B9 (DONE). Decisions: ED-6, ED-40, ED-41 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 2, 3, 4, 6, 7, 10. Before B12.

## What exists
`ccis/schemas/rule-pack.schema.json` (B10, 3 KB): required `manifest`, `class`, `scope`, `inputs`, `parameters`, `sourceNote`; `rules` and `tests` are refused. `$defs`: `text`, `key`, `conceptKey`. `ccis/schemas/analytics.schema.json` lines 249 to 280 hold the `expr` def (whitelist of 21 operators, `ref` leaves, boolean or string literals, never a number) and lines 171 to 215 the `tests` shape.

## What to build (shape only, no cross-checks)
1. `rule-pack.schema.json`: add required `rules`, `tests`, `provenance` (closed `origin`, `basis` text, copy the element-group one).
2. `rules`: array, `minItems` 1, `maxItems` 20. Item closed (plus `x-`), required `id` (key), `kind` enum `missing`, `stale`, `conflicting`, `inconsistent`, `interval-documentation`, `severity` enum `info`, `attention` (no `critical`), `when` (`$defs/expr`), `message`. `message` is a closed object with required `template` (text, `maxLength` 200).
3. `$defs/expr`: copied from the analytics schema with two changes: `ref` enum is `input`, `parameter` only (no `conversion`), and `args` has no extra rule. A literal stays boolean or string, never a number.
4. `tests`: array, `minItems` 1. Item closed, required `id` (key), `rule` (key), `fixture` (pattern `^[a-z0-9][a-z0-9-]*$`), `synthetic` (const true), `enables` (as in analytics, `testOnly` const true), `expect` (enum `fires`, `does-not-fire`, `not-evaluated`).
5. Regenerate validators (`npm run build`) and `index.html` (`npm run build:app`).

ED-41 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) `expr` is copied, not shared, because `$defs` are per schema; a test compares the operator enum with the analytics one so they cannot drift; (b) the schema allows a message template as plain text only; the closed placeholders, the instruction-verb list, depth 8 and node 40 limits are B12; (c) `rules` and `tests` become required, so the B10 test fixture and its "rules refused" case change (planned, not a surprise); (d) a test names a fixture by id only, the fixture files and the reproduction of results are B13 and B18; (e) `kind` is a label for the reviewer; the check that the condition really fits the kind is B13.

## Files to read
- `ccis/schemas/rule-pack.schema.json` whole file.
- `ccis/schemas/analytics.schema.json` lines 249 to 280 and 171 to 215.
- `test/ccis-rule-pack-schema.test.js` whole file (fixture and case 4 change).

## Files to change
`rule-pack.schema.json` (about 90 added lines), `test/ccis-rule-pack-schema.test.js` (fixture gains `rules`, `tests`, `provenance`; case 4 no longer refuses `rules`), new `test/ccis-rule-pack-rules-schema.test.js`, regenerated validators and `index.html`, CCIS tracker row `ECO-B11`. Studio: `DECISIONS.md` ED-41.

## Tests (new file, at most 8 cases, `node --test`)
1. A pack with one rule of each kind and a test per rule passes.
2. Missing `rules`, `tests` or `provenance` is refused; empty `rules` or `tests` refused; 21 rules refused.
3. Bad `kind`, `severity: critical`, or an unknown rule member refused; `x-` accepted.
4. A numeric literal, a `conversion` ref, or an operator off the whitelist in `when` is refused.
5. A message with a template over 200 characters, empty, or a non-string is refused.
6. A test with `synthetic` false or missing, a bad `expect`, or `testOnly` false is refused.
7. The operator enum equals the analytics operator enum.
8. The generated validators equal a fresh build.
Run the two rule-pack test files plus `ccis-element-group-schema`, `ccis-group-owner-schema`, `ccis-exec-min-max-coalesce`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 schema edit, 1 new test, 1 edited test, 2 regenerated files. If any test other than the B10 file needs a change, or the regenerated validators differ in more than `rulePack`, stop and split.

## Safety notes
Shape only. No number anywhere: literals exclude numbers, parameters carry no value, messages are untrusted plain text. A rule has no write member and no action member.

## Context impact
Reads about 10 KB; adds about 90 JSON lines and about 100 test lines. Next after this: B12 (references, operator types, parameter and message rules).
