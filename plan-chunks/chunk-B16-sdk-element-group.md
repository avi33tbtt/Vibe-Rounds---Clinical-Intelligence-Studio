# Chunk B16: SDK builder for an element group

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B8 (DONE). Decisions: ED-54 (new, OPEN working choice: split). Narrowed from the old B16: `entryForm` and `contexts` on the pathway builder are B16a. Before B16a, B18.

## What exists
`ccis-sdk.js` holds the shared helpers (`shape`, `list`, `need`, `textOf`, `keyText`, `artefactId`, `referenceAt`, `manifest`, `withPath`, `SdkError`, S-001 to S-004). `ccis-sdk-pathway.js` (38 lines) is the model: one `shape` with `allowed` and `required`, `x-` members pass through, returns a frozen plain object exactly as the file holds it, adds no rule the file route lacks. No group builder exists. `ARTEFACT_TYPES` already lists `element-group`. Schema: `ccis/schemas/element-group.schema.json`, all six members required (`manifest`, `intent`, `members`, `sharedConcepts`, `versionPolicy`, `provenance`), no extra members; `members` at least 1 item; a shared concept's `fields` at least 2 items. Real fixture: `ccis/examples/diabetes-ckd/diabetes-ckd.group.json` (82 lines).

## What to build
New `ccis-sdk-group.js` (about 40 lines), export `elementGroup(spec)`:
- `manifest`: `manifest(x, { type: 'element-group' })` through `withPath`.
- `intent`: closed `shape` of `purpose`, `setting`, `population`, `rationale` (`textOf`) and `exclusions` (list of `textOf`, may be empty); all required.
- `members`: list, min 1, each `{ element, required }`: `element` is a `referenceAt` with `type` `contextual-element` (a `version` or a `range`, as the schema's reference allows), `required` a boolean.
- `sharedConcepts`: list, min 0, each `{ concept, meaning, fields, resolution }`: `concept` text matching `^[a-z0-9][A-Za-z0-9._-]*$`; `meaning` `textOf`; `fields` list min 2 of `{ element, field }` (`element` `^\S+$`, `field` `keyText`); `resolution` one of `same-concept`, `related-not-merged`.
- `versionPolicy`: `{ conflict }`, one of `refuse`, `prefer-listed-order`. `provenance`: `{ origin, basis }` (`textOf`).
- Same rules as the file route and nothing more: duplicate members, unknown elements, shared fields that do not exist, and cohort-style meaning stay with the validator (B3a, B4). No defaults invented.

## Files to read
`ccis-sdk-pathway.js` whole (38 lines); `ccis-sdk.js` lines 31 to 60, 84 to 90, 138 to 170; `test/ccis-sdk-pathway.test.js` lines 1 to 30 (equivalence pattern with `compareRoutes`); `ccis/examples/diabetes-ckd/diabetes-ckd.group.json`.

## Files to change
New `ccis-sdk-group.js`; new `test/ccis-sdk-group.test.js`; CCIS tracker row `ECO-B16`. No other module, no dependency, no `index.html` change unless `--check` says the app imports the SDK (it should not; if it does, rebuild). Studio: ED-54, tracker.

## Tests (at most 8 cases)
1. The diabetes-ckd group through `elementGroup(JSON.parse(file))` is identical on both routes (`compareRoutes`) and validates the same per layer (as the pathway test does).
2. A minimal group built in code equals the hand-written object member for member.
3. A missing required member gives S-002 with the path (each of the six).
4. An unknown member gives S-001; an `x-` member passes through.
5. A member with a wrong type, a non-boolean `required`, or a reference to another type gives S-003 or S-004 with the path.
6. `fields` with one item, `members` empty, a resolution or conflict outside its list: refused with the path.
7. A shared concept key with a space or capital first letter is refused.
8. The result is frozen and the input spec is not changed.
Run the new file, `ccis-sdk*.test.js`, `ccis-element-group-schema.test.js`, `ccis-validator-group*.test.js`. At most 3 breakage checks.

## Size guard
1 new module (about 40 lines), 1 test. More than that: stop and split (for example, `sharedConcepts` first).

## Safety notes
Wording is plain text only. The builder never fills a number, threshold or clinical wording. Synthetic fixtures only.

## Context impact
Reads about 6 KB; adds about 40 module lines and 100 test lines. Next: B16a, then B17.
