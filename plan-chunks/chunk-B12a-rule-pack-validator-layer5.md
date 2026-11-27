# Chunk B12a: rule-pack validator part 2 (layer 5: limits, concept fit, messages)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B12 (DONE). Decisions: ED-6, ED-41, ED-42, ED-43 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 4, 6 and 8. Before B13.

## What exists
`ccis-validator-rulepack.js` (B12, layer 4: V4-012 to 018, exports `RULEPACK_RULES` and `rulePackSemantic`). `ccis-validator.js`: layer 5 is the `graph-and-compatibility` mark (one long line, about line 606, one conditional per kind) and the `RULES` list. Last layer 5 id: `CCIS-V5-047`. The pattern for a concept check against resolved elements is `ccis-validator-concept-package.js` (a field concept is `field.concept.code`; `ctx.resolve(ref)` gives `{ state: 'ok', doc }` or a skip state). Schema already limits a message to 200 characters (pattern, B11) and the pack to 20 rules.

## What to build
New module `ccis-validator-rulepack-checks.js` (about 90 lines, pure; a separate file keeps B12's module under 100 lines) exporting `RULEPACK_CHECK_RULES` and `rulePackChecks(doc, ctx)`; add the rules to `RULES` and one conditional (`kind === 'rule-pack'`) to the layer 5 line.
- `CCIS-V5-048` error: a rule condition is deeper than 8 operator levels (root is level 1) or has more than 40 nodes (operators, references and literals).
- `CCIS-V5-049` error: an input's concept is carried by no field of the element it names (or, with no `element`, by no resolved referenced element), when at least one such element resolved. An element that did not resolve is skipped (layer 3 reports it).
- `CCIS-V5-050` error: a message uses a placeholder outside the closed set `{concept}`, `{parameter}`, `{count}`, or has an unbalanced brace.
- `CCIS-V5-051` error: a message holds markup (`<` or `>`), a URL (`://`, `www.`, `mailto:`) or a control character.
- `CCIS-V5-052` error: a message contains an instruction word from a closed list: order, start, stop, give, prescribe, increase, reduce, discontinue, administer, initiate, withhold, titrate. Whole word, case-insensitive, placeholders removed first. A crude guard; the reviewer is the real control (design section 6).
- `CCIS-V5-053` warning: an input names an `element` that is not in `manifest.references`, so its concept cannot be checked.
Messages name ids and paths only. No clinical content.

ED-43 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) concept fit uses elements named in `manifest.references` (type `contextual-element`) matched by id, as ED-28 does for terminology packages; (b) depth counts operator nesting only; the node count includes leaves; (c) the verb list has 12 words and lives in the module, a change is a new row; whole-word matching means "ordered" is not caught; (d) the three placeholders carry no value in a message, they are filled at run time by B14; (e) a markup character is refused in a message even though layer 6 already scans strings, so the error names the rule and message.

## Files to read
`ccis-validator-concept-package.js` (whole, 40 lines); `ccis-validator-rulepack.js` lines 1 to 30; `ccis-validator.js` the layer 5 line and `RULES` (lines 45 to 70); `test/ccis-validator-rulepack.test.js` lines 1 to 25 (fixture; copy).

## Files to change
New `ccis-validator-rulepack-checks.js`, new `test/ccis-validator-rulepack-checks.test.js`; `ccis-validator.js` (about 3 lines); rebuilt `index.html`; CCIS tracker row `ECO-B12a`. Studio: ED-43.

## Tests (at most 8 cases, `node --test`)
1. A valid pack with a resolving element, a placeholder message and a depth-3 condition gives no layer 5 row.
2. V5-048: depth 9 refused, depth 8 accepted; 41 nodes refused.
3. V5-049: concept absent from the resolved element; present passes; an unresolved element is skipped.
4. V5-049 without `element`: absent from every resolved referenced element.
5. V5-050: `{creatinine}` and an unclosed `{` refused; the three allowed placeholders accepted.
6. V5-051: `<b>`, `https://x`, `www.x` and a tab each refused.
7. V5-052: "Order a repeat" and "STOP" refused; "No record found" and "reordered" accepted.
8. V5-053 warning, and all six ids are in `RULES`.
Run the new file plus `ccis-validator-rulepack`, `ccis-validator-concept-package`, `ccis-validator-contexts`, `ccis-rule-pack-*`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module, 1 test, 3 edited lines. If the module passes 100 lines or any existing test needs a change, stop and split. Do not touch the two stale tests (row B12b).

## Safety notes
Validator form checks only. The verb list is a guard, not clinical review. No number, no clinical value in any message.

## Context impact
Reads about 8 KB; adds about 90 module lines and 100 test lines. Next: B12b (stale test pins), then B13.
