# Chunk B15b: the market publishes element groups (narrow exception to the cohort-name screen)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B15 (DONE). Decisions: ED-5, ED-12, ED-53 (new, OPEN working choice). Split from B15. Before B16, B18 and C15.

## What exists
`ccis-market.js`: `PUBLISHABLE_TYPES` has four types (line 27). The patient-data screen `lintArtefact` (line 88) refuses any member named like a cohort (`COHORT_NAME`, line 76: `cohort*`, `members`, `membership`, `memberIds`) as L-002 at any depth. The element-group schema (`ccis/schemas/element-group.schema.json` line 38) requires a top-level `members` array whose items are exactly `{ element: <reference>, required: boolean }` plus optional `x-` members. So every group is refused M-004 at `/members`. B15 scratch run: a group compiles and passes validation; only L-002 blocks it.

## Decision (ED-53, change before the build)
Add `element-group` to `PUBLISHABLE_TYPES`, and give L-002 one exception, as narrow as the schema allows: the member named `members` is skipped by the name rule only when ALL hold: it sits at the artefact root (path `/members`), `manifest.type` is `element-group`, and it is an array of objects whose non-`x-` keys are exactly `element` and `required` (`element` an object, `required` a boolean). Everything under it is still walked, so a patient-record name, an email, a long digit run or a cohort key nested inside is still found. Not allowed: `members` in any other type, nested, or holding strings or other shapes; `cohort*`, `membership`, `memberIds` anywhere. Alternative: rename the schema member (a standard change, B20 territory); not recommended.

## What to build (about 12 lines in `ccis-market.js`)
- `PUBLISHABLE_TYPES` gains `element-group`; M-002 sentence becomes "contextual elements, element groups, analytics packages, pathways and rule packs" (this restores the wording B15 left out).
- A small pure check `isGroupMembers(artefact, path, value)` used in the name loop (`if (COHORT_NAME.test(n) && !groupMembersOk(...)) add('L-002', here)`). The `LINT_RULES` meaning for L-002 gets one added sentence naming the exception. `lintArtefact` signature unchanged.
- The M-002 pin in `test/ccis-market.test.js` line 211 drops `element-group` from the refused types; line 212 pins the type list; the `LINT_RULES` meaning pin, if any.

## Files to read
`ccis-market.js` lines 25 to 45, 62 to 125, 146 to 175; `test/ccis-market.test.js` lines 205 to 215, 248 to 262 and the `LINT_RULES` pin (`grep -n "LINT_RULES\|meaning" test/ccis-market.test.js`); the group fixture in `test/ccis-element-group-schema.test.js` lines 7 to 20 and the market fixture helpers of `test/ccis-market-b15.test.js` lines 1 to 40.

## Files to change
`ccis-market.js`; new `test/ccis-market-b15b.test.js`; at most 3 pin edits in `test/ccis-market.test.js`; rebuilt `index.html`; regenerated market folders only if `--check` says they differ (the diabetes-ckd set may already hold a group: check). CCIS tracker row `ECO-B15b`. Studio: ED-53, tracker. If the compiler, validator or schema needs a change, stop and split.

## Tests (at most 8 cases)
1. A group publishes (members published first); entry type `element-group`, its member references listed, `provides` and `requires` empty.
2. Same bytes again: `unchanged`; changed bytes, same version: M-006.
3. A group whose `members` hold strings (a list of ids) gives L-002 at `/members`.
4. An `element` or `required` key replaced by another key (for example `name`) in an item gives L-002 or L-001 as before.
5. A patient-record name or a long digit run inside a member item is still found (L-001, L-006).
6. `cohort`, `membership`, `memberIds` at the group root still give L-002; `members` on a contextual element or nested in `x-` still gives L-002.
7. A group whose member element is not published gives M-005 and nothing is added.
8. `loadMarket` round-trip of a market with a group.
Run the new file, `ccis-market*.test.js`, `ccis-app-market.test.js`, `ccis-app-build*.test.js`. At most 3 breakage checks (drop the root check, drop the item-shape check, drop the type check).

## Size guard
1 module edit (about 12 lines), 1 test, at most 3 pin edits, rebuilt outputs. More than that: split.

## Safety notes
This loosens a safety screen, so it needs your approval. The screen is not a proof; the exception covers one key of one type with one shape, and the contents are still screened. No clinical content; no values in refusal details.

## Context impact
Reads about 8 KB; adds about 12 module lines and 100 test lines. Next: B16.
