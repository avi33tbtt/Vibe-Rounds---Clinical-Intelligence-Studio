# Chunk B13p: refresh the rule-pack test fixture before B13 (ED-44 consequence)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B12b (DONE). Decisions: ED-44. Before B13.

## Why
B13 (layer 6 shape check V6-003) was built on a fresh clone and verified: the new module and its 8-case test pass. It was not delivered, because the B13 size guard says that if any existing test needs a change, stop and split. Two existing tests need a change:
- `test/ccis-validator-rulepack.test.js` (B12): the fixture rule is `gt(parameter, input)` on a `data-quality` pack. Under ED-44 that is a single-value cut-off and V6-003 refuses it, so its tests 1 and 7 fail.
- `test/ccis-validator.test.js` test 23: the pinned import list must gain `./ccis-validator-rulepack-shape.js`. This cannot be done before the module exists (the pin would fail), so it stays in B13.

## What to change (this row)
One line in `test/ccis-validator-rulepack.test.js`, in `base()` (line 17): in the `gt`, keep `{ ref: 'parameter', id: 'window' }` first and replace the second argument `{ ref: 'input', id: 'crea' }` by `{ op: 'time-diff', args: [{ ref: 'input', id: 'crea' }, { ref: 'input', id: 'crea' }] }`. The parameter keeps its path, so every other case stays valid. This is allowed shape (b) of ED-44. Tested already: with this edit and the B13 module, all 102 tests in the related files pass.

## Files to read
`test/ccis-validator-rulepack.test.js` lines 10 to 25.

## Tests
Run `ccis-validator-rulepack.test.js` and `ccis-validator-rulepack-checks.test.js`: all pass before and after (the module does not exist yet, so the old and new fixture both pass). No breakage check is possible for a fixture-only change: say so in the notes.

## Size guard
1 changed test line, nothing else. If another test fails, stop and split.

## Approval needed from you
Approving this chunk also approves one named exception for B13: B13 may add `'./ccis-validator-rulepack-shape.js'` to the import pin in `test/ccis-validator.test.js` (test 23), because adding an import always changes that pin (B12b and B12a showed it). Without your OK B13 stops again.

## Context impact
Reads about 1 KB; changes 1 line. Next: B13.
