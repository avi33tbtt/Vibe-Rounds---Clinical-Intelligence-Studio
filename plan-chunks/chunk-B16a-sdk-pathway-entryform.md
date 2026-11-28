# Chunk B16a: SDK pathway builder, `entryForm` and `contexts`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B16 (DONE, pushed). Decisions: ED-54 (split), ED-55 (new, OPEN working choice below). Next: B17, then B18.

## What exists
`ccis-sdk-pathway.js` (38 lines) builds a pathway with `shape`; `x-` members pass through; it has no `entryForm` and no `contexts`, so a pathway that carries them is refused with S-001 today. Schema `ccis/schemas/pathway.schema.json` lines 103 to 190: `entryForm` is `{ sections }` (min 1); a section is `{ id, label, items }` (`id` is `idText`, `label` is `textOf`, `items` min 1); an item is either `{ kind: "element", element, fields?, rowGroups?, visibleByDefault? }` or `{ kind: "section", section, visibleByDefault? }`; `contexts` is a list (min 1) of `{ id, label, elements }` with `elements` min 1 artefact ids. No JSON fixture has `entryForm`; `test/ccis-compose-pathway.test.js` builds inline pathways with it (line 21 helper `mk`).

## What to build (ED-55, working choice)
Edit `ccis-sdk-pathway.js` only (about 14 added lines, file stays under 55); no new module.
- `entryForm`: `shape` with `sections` required, `list` min 1 of section.
- Section: `{ id: idText, label: textOf, items: list min 1 }`, all required.
- Item: dispatch on `kind` (`element` or `section`; anything else S-003 at `kind`; missing `kind` S-002). Element item: `element` `artefactId`; `fields` list min 1 of `keyText`; `rowGroups` list min 1 of lists min 1 of `keyText`; `visibleByDefault` boolean; `kind` and `element` required. Section item: `section` matching `^[a-z][a-z0-9.-]*$`; `visibleByDefault` boolean; `kind` and `section` required.
- `contexts`: list min 1 of `{ id: idText, label: textOf, elements: list min 1 of artefactId }`, all required.
- Both optional on the pathway, like `nodes`. Nothing invented: known section names, element membership, field existence and duplicate ids stay with the validator (B6, B6a).

## Files to read
`ccis-sdk-pathway.js` whole; `ccis-sdk.js` lines 106 to 109 (`idText`, `keyText`, `artefactId`, `textOf`); `test/ccis-sdk-pathway.test.js` lines 1 to 30; `test/ccis-compose-pathway.test.js` lines 15 to 35 (inline `entryForm` example). Do not read the validator modules.

## Files to change
`ccis-sdk-pathway.js`; new `test/ccis-sdk-pathway-form.test.js` (the old pathway test file stays untouched); CCIS tracker row `ECO-B16a`. No dependency. `index.html` only if `--check` says the app imports the pathway builder (it should not).

## Tests (at most 8 cases)
1. A pathway with `entryForm` and `contexts`, built in code, equals the hand-written object member for member, is frozen at each level and leaves the input unchanged.
2. Equivalence: the same pathway through `compareRoutes` (JSON text against builder output) is identical, and `validateArtefact` gives the same per-layer verdict on both.
3. Both new members are optional: the existing snake pathway still builds identically (its test file stays green).
4. Missing `sections`, `items`, `label`, `elements`: S-002 with path; empty `sections`, `items`, `elements`: S-003.
5. An item without `kind` is S-002; an unknown `kind` is S-003 at `kind`; an element item with a `section` member, or a section item with an `element` member, is S-001.
6. `fields` or `rowGroups` empty, or a row group that is empty or holds a bad key: S-003 with path.
7. `visibleByDefault` not boolean; section id with capital letter or space; section name with space: S-003.
8. `x-` members pass through in an item and a context; a bad section name is NOT checked against the five known names (validator's job).
Run the new file, `ccis-sdk*.test.js`, `ccis-compose-pathway.test.js`, `ccis-validator-entryform*.test.js`, `ccis-validator-contexts*.test.js`. Known: tests 4.2d.1 and 4.2b already fail before B16a. At most 3 breakage checks.

## Size guard
1 module edit (stay under about 55 lines in total), 1 test of at most 8 cases. If the module passes 60 lines, stop and split `contexts` into B16b.

## Safety notes
Layout and grouping only. No number, rule or threshold can be written. Synthetic ids and labels only.

## Context impact
Reads about 5 KB; adds about 14 module lines and 90 test lines.
