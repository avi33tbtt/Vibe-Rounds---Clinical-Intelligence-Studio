# Chunk B2a: concept ownership check

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B2, B1a, B1e (DONE). Design: `docs/PACKAGE-DESIGN.md` section 5 (last bullet) and the conflict report below section 6; ED-12; standing rule 5. Decision: ED-29 (new, OPEN working choices). Before B15.

## What to build
New pure module `ccis-concept-owners.js` (about 70 lines): no network, files, clock or randomness.

1. `conceptOwners(artefacts)` takes parsed artefacts, keeps only `contextual-element` documents, and keeps the highest version of each element id (compare with `compareSemver` from `ccis-validator.js`, as `ccis-market.js` does). Fields come from `fields` and module `fields`. An **owner field** has `concept.code` and no `ownedBy`; a **user field** has both. Returns a sorted list, one entry per concept code:
   `{ concept, owners: [{ element, key }], users: [{ element, key }], keys: [distinct field keys], status }`.
   `status`: `second-owner` when owner fields sit in two or more different elements (ED-12: one owner element; two fields of one element are not a second owner); `needs-owner` when no field carries the concept without `ownedBy`; otherwise `ok`.
   `keys` lists the distinct field keys the concept appears under across elements (the "same concept under different keys" report). It is data only: an owner key `bpSys` with a user key `sbp` is normal, so it raises no diagnostic.
2. `ownershipDiagnostics(doc, entries, strict)` returns layer 5 rows for the concepts the given element's own fields carry:
   - `CCIS-V5-028` second owner. Path `<field>/concept/code` on this element's owner field.
   - `CCIS-V5-029` needs owner. Path on this element's user field.
   Severity is `warning` unless `strict` is true, then `error`. Rows name element ids, keys and concept codes only.

Why this shape: `validate` stays a warning (design: "a warning in `validate`"); the error at publish comes from B15 calling the validator with `strictOwnership: true`. B2a adds the option, B15 passes it.

ED-29 working choices (add one `DECISIONS.md` row, OPEN): (a) owner means owner element, not owner field; (b) key differences across elements are reported in data, not as a diagnostic; (c) unit and type conflicts between an owner and its users stay in B1e (V5-015), not repeated here.

Wiring in `ccis-validator.js`: one import; spread two rule entries from an exported `OWNERSHIP_RULES` into `RULES`; at the layer 5 line (about 598) add `...(kind === 'contextual-element' ? ownershipDiagnostics(first.doc, conceptOwners([...(options.known ?? []), first.doc]), options.strictOwnership === true) : [])`. Where the same element id exists in `known` at another version, the highest version wins, so the checked document decides only when it is the highest; say so in a code comment.

## Files to read (line ranges)
- `ccis-validator-owner.js` lines 1 to 25 (style, `fieldsOf`; copy, do not import).
- `ccis-validator.js` lines 383 to 392 (`indexKnown`), 596 to 599 (layer 5 line), and the `compareSemver` export.
- `test/ccis-validator-concept-package.test.js` lines 1 to 30 (fixtures; copy).

## Files to change
New module, new `test/ccis-concept-owners.test.js`, 3 small edits in `ccis-validator.js`, `index.html` rebuilt (`npm ci` first if `esbuild` is missing), CCIS tracker row `ECO-B2a`. Studio: `DECISIONS.md` `ED-29`. No schema change. No change to `ccis-market.js` (that is B15).

## Tests (at most 8 cases, `node --test`)
1. One owner element and one user element give `ok`, `keys` lists both keys, no V5-028 or 029.
2. Owner fields in two elements give `second-owner` and a V5-028 warning on each checked element.
3. Two owner fields in one element give `ok` (not a second owner).
4. A concept carried only by user fields gives `needs-owner` and a V5-029 warning.
5. `strict` true makes both rows errors; the default makes both warnings.
6. An older version of an element is ignored when a higher one is present.
7. A module field is counted and gets its module path.
8. The `snake` and `diabetes-ckd` examples, a package and a group raise no new row; output is identical for the same input in a different order.
Run only the new file plus the `concept`, `concept-package`, `owner`, `terminology` and `element` validator tests and `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module (75 lines or less), 1 test, 3 small edits, 1 bundle. If the module passes about 90 lines, or a new warning breaks an existing test, stop and split.

## Safety notes
Identity only: no numbers, bands or thresholds. Ownership is never inferred from a label or a key. Messages are plain, untrusted text on display.

## Context impact
Reads about 5 KB of source; adds about 70 source lines and a test of about 100 lines. Next row after this: B3.
