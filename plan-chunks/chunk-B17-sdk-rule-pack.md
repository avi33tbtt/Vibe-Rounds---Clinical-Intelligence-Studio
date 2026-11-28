# Chunk B17: SDK builder for a rule pack

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B14 (DONE), B16 and B16a (DONE). Decisions: ED-56 (new, OPEN working choice below). Next: B18.

## What exists
Schema `ccis/schemas/rule-pack.schema.json` (10.7 KB): nine required members (`manifest` type `rule-pack`, `class`, `scope`, `inputs`, `parameters`, `rules`, `tests`, `provenance`, `sourceNote`) and optional `governance`; `x-` members allowed at every level except `message`, a test's `enables` items and expression nodes. Rules: 1 to 20, each `{ id, kind, severity, when, message: { template } }`, template `^\S.{0,199}$`. Inputs: `{ id (key), concept (conceptKey), element?, read: latest|all|count }`. Parameters: `{ id, label, kind: interval|count, units?, enabledByDefault: false (const), suppliedBy }`; declared, never valued (ED-40). Tests: `{ id, rule, fixture, synthetic: true, enables: [{ parameter, testOnly: true }], expect: fires|does-not-fire|not-evaluated }`. Expressions: operators `and or not is-present state-of eq ne lt le gt ge add sub time-diff convert if count select min max coalesce`; leaves `{ ref: input|parameter, id }` and `{ literal: boolean|string }`, never a number.
`ccis-sdk-analytics.js` has `exprAt` (line 66, private) with its OWN operator list (no `min max coalesce`) and ref kinds (`conversion` allowed); an analytics expression is therefore not a rule expression.

## What to build (ED-56, working choice)
1. `ccis-sdk-analytics.js`: turn `exprAt` into a factory `exprWith(operators, refKinds)` (export it; about 4 changed lines); `expr`, `OPERATORS`, `REF_KINDS` keep their values and behaviour, so the analytics tests stay byte-identical.
2. New `ccis-sdk-rules.js` (about 50 lines), export `rulePack(spec)`, `RULE_OPERATORS` (the schema list), `RULE_KINDS` (`missing stale conflicting inconsistent interval-documentation`), `RULE_SEVERITIES`, `PACK_CLASSES`, `READ_MODES`, `PARAMETER_KINDS_RULES` (`interval count`), `TEST_EXPECTS`. Built with `shape`, `list`, `withPath`, `manifest(x, { type: 'rule-pack' })`; `enabledByDefault` must be the boolean `false`, `synthetic` and `testOnly` must be `true`; rules list 1 to 20; `message` and `enables` items are closed objects.
3. Same rules as the file route and nothing more: duplicate ids, unknown input or parameter ids, unknown rule names in tests, operator arity, and the data-quality-only scope (ED-6) stay with the validator. No default, number or threshold is ever filled in.

## Files to read
`ccis-sdk-analytics.js` lines 1 to 20 and 61 to 83; `ccis-sdk-group.js` (shape of a small builder); `test/ccis-rule-pack-schema.test.js` lines 8 to 20 (a minimal valid pack). Not the validator.

## Files to change
`ccis-sdk-analytics.js` (factory, about 4 lines); new `ccis-sdk-rules.js`; new `test/ccis-sdk-rules.test.js`; CCIS tracker row `ECO-B17`. No dependency; `index.html` only if `--check` says the app imports the SDK analytics module.

## Tests (at most 8 cases)
1. A full pack built in code equals the hand-written object, frozen, input unchanged; enum constants equal the schema's.
2. Equivalence: JSON text against builder output through `compareRoutes` is identical, and `validateArtefact` gives the same per-layer verdict on both.
3. Each of the nine required members missing: S-002 with path; unknown member S-001; `x-` members pass through (not inside `message`).
4. `enabledByDefault: true`, `synthetic: false`, `testOnly: false`: S-003 with path; a parameter with a `value`, `default` or `min` member: S-001.
5. Zero rules and 21 rules: S-003 at `rules`; template over 200 characters or empty: S-003; a `message` with an extra member: S-001.
6. Expressions: `min`, `max`, `coalesce` accepted; `conversion` ref refused; a number literal refused; unknown operator refused, each with path.
7. Bad key, concept key with a space, unknown `read`, `kind`, `severity`, `class`, `expect`: S-003 with path.
8. Analytics `expr` still refuses `min` and still accepts a `conversion` ref (the factory changed nothing); validator-level checks (duplicate rule ids, unknown input id) are not made by the builder.
Run the new file, `ccis-sdk*.test.js`, `ccis-rule-pack*.test.js`, `ccis-rules.test.js`. Known: `4.2d.1` and `4.2b` already fail. At most 3 breakage checks.

## Size guard
1 new module (about 50 lines), 1 small edit, 1 test. Over 60 lines: stop and split `tests` and `governance` into B17a.

## Safety notes
A rule pack holds no number, default, threshold or action. Parameters arrive disabled. Wording is plain text. Synthetic fixtures only; class `clinical-prompt` is buildable but stays disabled until ED-2 names an owner.

## Context impact
Reads about 6 KB; adds about 54 module lines and 100 test lines.
