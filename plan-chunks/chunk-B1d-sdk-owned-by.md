# Chunk B1d: SDK field builder accepts `ownedBy`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1a (DONE 2026-11-28), B1c (DONE 2026-11-27). Comes before C4. Split from B1a. Decisions: ED-12 (one owner per concept).

## What to build
`field()` in `ccis-sdk-element.js` accepts the optional `ownedBy` member B1a added to the element schema, with the same rules and nothing more (the SDK adds no rule the file route lacks):

1. `ownedBy`: object, members `element` and `key` both required.
   - `element`: `artefactId` from `ccis-sdk.js` (line 108; lower-case dotted segments).
   - `key`: `keyText` (line 107; same pattern as the schema `$defs/key`).
   - Unknown member: S-001 with the member path. `x-` members pass as JSON data (the schema allows them), so use `shape` without `closed`.
   - Missing `element` or `key`: S-002. Wrong type or pattern: S-003. Never quote the refused value.
2. `ownedBy` without `concept` on the same field: S-002, path `concept` (the schema's `dependentRequired`). Add one check next to the existing cross-member checks in `field()`.

Whether the owner exists, has the same concept, or agrees on type and unit stays with B1e (validator layer 5). Self-ownership also stays with B1e. The result stays a plain frozen field, exactly as the file holds it; nothing is defaulted.

## Files to read (line ranges)
- `ccis-sdk-element.js` lines 1 to 50 (`concept`, `contexts` helpers, `field`, the `allowed` map).
- `ccis-sdk.js` lines 28 to 49 (`shape`, `need`) and 106 to 108 (`keyText`, `artefactId`; check both are exported).
- `test/ccis-sdk-field-concept.test.js` lines 1 to 30 (helpers `refused`, `sdkRefuses`, `fileRefuses`, `same`; copy, do not edit).
- `ccis/schemas/element.schema.json` lines 116 to 127 and 147 (the `ownedBy` block, already known from B1a).

## Files to change
- `ccis-sdk-element.js`: one `ownedBy` helper (about 6 lines), one entry in the `allowed` map, one dependency check (1 line).
- New `test/ccis-sdk-field-owned-by.test.js` (the B1c test file stays untouched).
No new module. No schema change, so `npm run build` is not needed (confirm with `node scripts/build-validators.js --check`).

## Tests (at most 8 cases, `node --test`)
1. A field with `concept` and `ownedBy` is identical on the JSON route and the code route through `compareRoutes` (same checksum).
2. Same for a module-level field object (the builder is shared).
3. `ownedBy` without `concept` is refused (S-002, path `concept`).
4. `ownedBy` missing `element` or `key` is refused (two assertions, S-002).
5. `ownedBy` with an unknown member is refused (S-001); an `x-` member is accepted and survives on both routes.
6. `element` with uppercase or a space, and `key` with a leading digit, are refused (S-003).
7. Every field of the snake element and the existing B1c cases still build unchanged (guards existing behaviour).
8. For each refusal in 3 to 6 the JSON route (validator on the same field inside an element) also fails, so both routes refuse the same inputs.
Run only the new file, `test/ccis-sdk-field-concept.test.js`, `test/ccis-sdk-field.test.js`, `test/ccis-sdk-elementbody.test.js` and `test/ccis-element-owned-by-schema.test.js`. At most 3 breakage checks (for example drop the dependency check and see case 3 fail; use `keyText` as `artefactId` and see case 6 fail; restore).

## Size guard
1 module edit (about 8 lines), 1 test file of about 80 lines. If the edit needs more than about 20 lines, or a shared helper in `ccis-sdk.js` must change, stop and split.

## Safety notes
- `ownedBy` carries identity only: no numbers, bands or thresholds.
- Ids and keys are plain text, untrusted on display; this row adds no UI.
- No external code system appears anywhere in this change.

## Context impact
Reads about 6 KB of source. Adds about 8 source lines and one test. On completion: Studio tracker, CCIS tracker (`ECO-B1d`). No `DECISIONS.md` rows, no CHANGELOG amendment. Next row after this: B1e.
