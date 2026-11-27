# Chunk B1c: SDK field builder accepts `concept` and `contexts`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1 (DONE 2026-11-27). Comes before C4, which authors concepts through the SDK.

## What to build
`field()` in `ccis-sdk-element.js` accepts the two optional members B1 added to the element schema, with the same rules and nothing more (the SDK adds no rule the file route lacks):

1. `concept`: built with the existing `binding` helper from `ccis-sdk.js`, then narrowed exactly as the schema narrows it:
   - `codeSystem` and `code` present (S-002 if missing; the `binding` helper already does this for non-unmapped status);
   - `codeSystem.id` is `"local"`;
   - `mapping.status` is `"mapped"`;
   - `valueSet` is refused (S-001, a member that is not part of this artefact).
   - Wrong narrowing gives S-003 or S-004 with the member path, as the other field rules do. Never quote the refused value.
2. `contexts`: `list(v, p, 1, tag)` where a tag is a string matching `^[a-z][a-z0-9-]*$`; duplicates refused (S-004, as `uniqueItems` in the schema).

The key format of `code` and the set of known tags stay with B2 (validator), not the SDK. The result stays a plain frozen field, exactly as the file holds it; nothing is invented or defaulted.

## Files to read (line ranges)
- `ccis-sdk-element.js` lines 1 to 45 (`field`, the `shape` allowed map).
- `ccis-sdk.js` lines 49 to 105 (`need`, `binding`, mapping rules); grep `list` and `withPath` exports.
- `test/ccis-sdk-field.test.js` lines 1 to 30 (helpers `refused`, `compareRoutes` use, `BASE`).
- `ccis/schemas/element.schema.json` the `concept` and `contexts` block (about 18 lines, already known from B1).

## Files to change
- `ccis-sdk-element.js`: one `concept` helper (about 10 lines) and two entries in the `allowed` map.
- New `test/ccis-sdk-field-concept.test.js` (not an edit of the existing field test, so that file stays untouched).
No new module. No schema change, so `npm run build` is not needed (confirm with `--check`).

## Tests (at most 8 cases, `node --test`)
1. A field with `concept` and `contexts` is identical on the JSON route and the code route through `compareRoutes` (same checksum).
2. Same for a module-level field object (the builder is shared).
3. `concept` in a system other than `local` is refused (S-003 or S-004, path `concept/codeSystem/id` style as the SDK writes paths).
4. `concept` with status `unmapped`, `lossy` or `low-confidence` is refused.
5. `concept` without `code` is refused; `concept` with `valueSet` is refused.
6. `contexts` empty, duplicated, uppercase or with a space is refused (one case, four assertions); a valid list passes.
7. Every field of the snake element still builds with no change (guards existing behaviour).
8. For each refusal in 3 to 6 the JSON route (validator on the same field inside an element) also fails, so the two routes refuse the same inputs.
Run only the new file, `test/ccis-sdk-field.test.js`, `test/ccis-sdk-elementbody.test.js` and `test/ccis-element-concept-schema.test.js`. At most 3 breakage checks (for example drop the `local` check and see case 3 fail, restore).

## Size guard
1 module edit (about 12 lines), 1 test file. If the edit needs more than about 25 lines, or the `binding` helper must change (it also serves `binding` on `code` fields), stop and split.

## Safety notes
- `concept` is identity only: no numbers, bands or thresholds.
- `display` stays plain text, untrusted on display; this row adds no UI.
- No external code system is accepted in `concept` (phase I uses separate maps).

## Context impact
Reads about 6 KB of source. Adds about 12 source lines and one test of about 80 lines. Studio tracker and CCIS tracker (`ECO-B1c`) update on completion. Next row after this: B1a.
