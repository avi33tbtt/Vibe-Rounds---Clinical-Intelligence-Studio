# Terminology design: local codes now, external codes later

Prototype. Synthetic data only. Not for clinical use.

Row A10. Answers ED-7 (local vocabulary only), ED-9 (concept keys are local bindings) and ED-22 (external codes arrive later as separate maps). Read by B1, B1b, B2, C4, F11, I0 to I9 and H3.

Sources read: CCIS `ccis/schemas/terminology-binding.schema.json` (lines 1 to 63) and EMR `local-terms.js` (lines 1 to 131), at the commits cloned on 2026-11-27.

## 1. What exists today

**CCIS binding slot** (`terminology-binding.schema.json`). One object per binding:
- `codeSystem` and `valueSet` are system references: an `id` plus either a `version` or `noVersionPublished: true`.
- `code` is any string without whitespace. `display` is for people and is never used for matching.
- `mapping.status` is `mapped`, `lossy`, `low-confidence` or `unmapped`. Lossy and low-confidence need `confidence` and `note`. Unmapped needs a `note` and carries no code and no system.
- Any binding that is not unmapped must have both `codeSystem` and `code`.

**EMR local terms** (`local-terms.js`). A free in-house vocabulary, system key `local`:
- A term has `code`, `version`, `display`, `category` and an optional `definition`.
- `code` matches `^[a-z0-9][a-z0-9._-]{0,63}$`. `display` is 1 to 200 plain characters. `definition` is up to 500.
- Categories: finding, condition, test, procedure, observation, drug, other.
- A new version of a code cannot be opened while an earlier one is draft, in review or reviewed.
- Status path: draft, in review, reviewed, published; also rejected and retired. Publishing a version retires the earlier published one in the same transaction.
- The author and the submitter can never review. Each reviewer reviews once.
- No `replacedBy`, no value sets, no concept-owner field.

## 2. Local code rules

1. **A key is an identity.** Once published, a key never changes meaning and is never reused for another meaning. A different meaning is a new key.
2. **Key format.** A concept key must satisfy the EMR rule `^[a-z0-9][a-z0-9._-]{0,63}$`, so the same key is valid in both apps. The CCIS binding allows looser codes; the validator (B2) applies the stricter rule to `local` keys.
3. **Display text is not identity.** Display text may be corrected in a new version. Matching always uses the key (Standing rule 5). Display is plain text and untrusted on display (Standing rule 9).
4. **Deprecate, never delete.** A key is retired with a `deprecated` flag and an optional `replacedBy` key. Retired keys stay resolvable so old observations still read. B1b adds `replacedBy` to the package; the EMR table has no such column, so a later EMR row adds it (see section 7).
5. **Versioned value sets.** A value set is a named, versioned list of keys with display text. A published version is immutable; a change is a new version. Pinned by `version` in the system reference.
6. **Who may add a key.** The model is the EMR flow: draft, submit, review by someone who is neither author nor submitter, publish. Until ED-2 names a clinician owner, every key is demo content.
7. **One owner per concept** (ED-12). The owner element is recorded outside the vocabulary (B1a `ownedBy`, B2a check). The vocabulary says what a concept is; it does not say who owns it.
8. **No numbers, bands or thresholds** in keys, display text or definitions (Standing rule 3).

## 3. How the B1b `local` package and EMR `local-terms.js` fit

- The CCIS `terminology-package` (B1b) is the authoring and distribution form: code system `local`, concept keys, value sets, display text, in a checksummed artefact.
- The EMR `local_terms` table is the runtime copy. Import (D5 to D8) turns package concepts into EMR terms through the existing draft, review and publish rules. No direct SQL.
- The package carries the `local` system id and its package version. The EMR keeps its own per-code `version`. Provenance (artefact id, version, checksum) is recorded on the imported rows, as in D4.
- Category is an EMR field. B1b should carry it as an optional package field so import does not guess. If a package gives none, the EMR default `other` applies.

## 4. The one binding slot holds a concept and a value set

The slot is one binding object with `codeSystem`, `code` and an optional `valueSet`.
- A field's **concept** is `codeSystem = { id: "local", version }`, `code = <concept key>`, `mapping.status = "mapped"`.
- A coded field's **value set** is `valueSet = { id, version }` in the same binding.
- Because the schema requires `codeSystem` and `code` unless the binding is unmapped, a binding that has only a `valueSet` is not valid today. B1b must check how the validator resolves `valueSet` and decide between making `code` optional when `valueSet` is present or always requiring the concept key. This is an open point for B1b, not decided here.
- External codes never go in this slot as the identity. If a future binding carries an external system, it is a map entry (section 5), not the field's concept.

## 5. Extension path: external terminologies as separate maps

Local keys stay the identity (ED-22). SNOMED CT, LOINC or any other system arrives later as a `terminology-map` artefact (phase I), never by rewriting elements, observations or pathways.

A map entry holds: local concept key, external system id, system version, external code, display, mapping status (the existing four values), reviewer, and a licence note.

Rules:
- **Official release file only.** A code and its display are copied from a release file you supply, never typed from memory, and checked against it (I3).
- **No redistribution.** Maps may hold the code and a short display needed to use it. They must not carry licensed code lists or descriptions in bulk. Licence terms and attribution notes for each system are recorded at I0 and are yours to confirm; this document does not state them.
- **Pinned version.** Each map names the system version it was checked against. One active map per concept and system (I2).
- **Status is honest.** Use `lossy`, `low-confidence` or `unmapped` with the required note rather than a guess.
- **Maps are additive.** Phase I adds a schema; 0.2.0 artefacts keep validating.
- **Used at the edge.** Export (H3) uses a verified map only for systems licensed in I0. Otherwise codes stay as text. A `local` code is never written out as a Coding.

## 6. Local-only always stays valid

An element with only a `local` concept is complete. An unmapped concept is valid and is reported, not an error. No analytics, pathway, rule or composition step may require an external code.

## 7. Findings for later rows

- **B1:** `concept` on an element field is a binding in the `local` system; use the key format in rule 2.
- **B1b:** add `deprecated`, `replacedBy`, optional `category`, value-set versioning; settle the `valueSet`-only binding question in section 4.
- **B2:** reject a `local` key that fails the key format; flag a key used by a field but absent from the pinned package.
- **C4:** keys are written by the SDK from one script; the ownership check (B2a) must pass.
- **D-phase (EMR):** `local_terms` has no `replacedBy`. A later EMR row adds it by migration (next free number) and updates the permission matrix and audit tests. Not planned in this row.
- **I0:** choose systems, edition, licence status and holder before I1.
