# Chunk B13: rule-pack validator layer 6 (data-quality shape only)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B12a (DONE). Decisions: ED-6, ED-18, ED-44 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 2 and 8 (layer 6). Before B13a.

## Why this row is narrow
The old B13 held layers 6, 7 and 8. Layer 8 needs owner, author and reviewer fields the rule-pack schema does not have, and layer 7 needs the case rules of design section 10. They are now B13a (layer 7), B13b (schema fields) and B13c (layer 8).

## What exists
`ccis-validator-governance.js`: layer 6 `executableSafetyLayer(doc)` is generic (code-like and markup strings, V6-001, V6-002) and already covers every rule pack string. `ccis-validator.js` line about 615 runs it for every kind. Rule expressions are `{op,args}`, `{ref:'input'|'parameter',id}` or `{literal}` (boolean or string, never a number). Pack member `class` is `data-quality` or `clinical-prompt`.

## What to build
New module `ccis-validator-rulepack-shape.js` (about 70 lines, pure) exporting `RULEPACK_SHAPE_RULES` and `rulePackShape(doc)`; add the rules to `RULES` and one conditional to the layer 6 entry in `ccis-validator.js` (`kind === 'rule-pack'` adds its rows to `executableSafetyLayer`).
- `CCIS-V6-003` error: in a pack of class `data-quality`, a comparison (`eq ne lt le gt ge`) outside the allowed shapes (ED-44). Path: the comparison node.
Class `clinical-prompt` is not checked here (layer 8 and the EMR block it).

ED-44 working choices (one `DECISIONS.md` row, OPEN, change before the build). A comparison in a data-quality rule is allowed only as:
(a) field to field: both arguments read inputs (an input reference, or `select min max coalesce` of inputs);
(b) time difference against an interval parameter: one argument is `time-diff`, the other a parameter reference;
(c) state test: one argument is `state-of`, the other a string literal.
Anything else is refused, in particular an input value, `count` or `add`/`sub` result against a parameter or a literal (a single-value cut-off, ED-18). `and or not is-present if` are not comparisons and stay free. Not checked: whether the meaning is clinical (the reviewer does that, design section 9).

## Files to read
`ccis-validator-rulepack.js` lines 18 to 30 (BOOLEAN_OPS, walking style); `ccis-validator.js` the layer 6 line (about 615) and `RULES`; `test/ccis-validator-rulepack-checks.test.js` lines 1 to 40 (fixture; copy).

## Files to change
New module and new `test/ccis-validator-rulepack-shape.test.js`; `ccis-validator.js` (about 3 lines); rebuilt `index.html`; CCIS tracker row `ECO-B13`. Studio: ED-44.

## Tests (at most 8 cases)
1. A valid data-quality pack (is-present, field-to-field, time-diff against parameter) gives no V6-003.
2. `gt(input, parameter)` refused at the node path.
3. `lt(input, literal)` refused.
4. `ge(count(input), parameter)` refused.
5. `time-diff` against a parameter, and `state-of` against a string literal, accepted.
6. Nested comparison under `and`/`not` found at the right path.
7. A `clinical-prompt` pack with `gt(input, parameter)` gives no V6-003.
8. V6-003 is in `RULES`; the pack passes through `validateArtefact` with layer 6 failed for case 2.
Run the new file plus `ccis-validator-rulepack`, `ccis-validator-rulepack-checks`, `ccis-validator-governance*` (if present), `ccis-validator.test.js`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module (under 100 lines), 1 test, 3 edited lines. If any existing test needs a change or more than one module is needed, stop and split.

## Safety notes
Form check only; it does not judge clinical meaning. Messages name ids and paths only.

## Context impact
Reads about 6 KB; adds about 70 module lines and 90 test lines. Next: B13a.
