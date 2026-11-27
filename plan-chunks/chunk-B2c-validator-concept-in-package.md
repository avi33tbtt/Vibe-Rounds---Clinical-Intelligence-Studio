# Chunk B2c: a field's concept must exist in the terminology package its element names

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B2, B1f (DONE). Design: `docs/TERMINOLOGY-DESIGN.md` section 7 (B2 line, second half). Decision: ED-28 (new, OPEN working choice). Before C4 and B2a.

## The open question, answered
How does an element name its package? Through the existing `manifest.references` list, with an entry of type `terminology-package` (id plus exact version or range). No schema change and no new field. Layer 3 already resolves these entries: a package that is not among the known artefacts gives `CCIS-V3-004` or `V3-005`, so this row never reports a missing package itself.

## What to build
New pure module `ccis-validator-concept-package.js` (50 lines or less): exports `PACKAGE_RULES` and `conceptPackageChecks(doc, ctx)`. Same style as `ccis-validator-owner.js`: fields from `fields` and module `fields`, JSON-pointer paths, messages name keys and ids only (never display text).

Algorithm: collect the element's `manifest.references` entries of type `terminology-package`; resolve each with `ctx.resolve`; keep those with state `ok`. If none resolved (no reference, or not known), return nothing: local-only and unresolved stay valid or are layer 3's. Otherwise build the set of concept keys across the resolved packages. For each field with `concept.code`:
1. `CCIS-V5-026` error: the code is in none of the resolved packages. Path `<field>/concept/code`.
2. `CCIS-V5-027` warning: the code is a deprecated concept; the hint names `replacedBy` when present.
A concept whose code is in at least one package passes, whichever package holds it.

ED-28 working choice (add one `DECISIONS.md` row, OPEN): an element with a `concept` but no `terminology-package` reference is valid (no row), so 0.1.0 artefacts and local-only use stay valid (design section 6). Whether a pathway or a publish step should require the reference is left to B2a or C4.

Wiring in `ccis-validator.js`: one import, spread `PACKAGE_RULES` into `RULES`, add `...(kind === 'contextual-element' ? conceptPackageChecks(first.doc, ctx) : [])` on the layer 5 line (about 598).

## Files to read (line ranges)
- `ccis-validator-owner.js` lines 1 to 40 (`ctx.resolve` use at about line 32, style).
- `ccis-validator.js` lines 394 to 404 (`resolve` result shape), 596 to 599 (layer 5 line).
- `test/ccis-validator-terminology.test.js` lines 1 to 20 (package fixture; copy).
- `test/ccis-validator-concept.test.js` lines 1 to 20 (element fixture; copy).

## Files to change
New module, new `test/ccis-validator-concept-package.test.js`, 3 small edits in `ccis-validator.js`, `index.html` rebuilt (`npm ci` first if `esbuild` is missing), CCIS tracker row `ECO-B2c`. Studio: `DECISIONS.md` `ED-28`.

## Tests (at most 8 cases, `node --test`)
1. A field whose concept is in the referenced package gives no V5-026 or 027 row.
2. A concept absent from the package gives V5-026 at the field path.
3. A deprecated concept gives V5-027 warning only; the hint names `replacedBy`.
4. No `terminology-package` reference: no row.
5. A reference to a package not in `known`: no V5-026 (layer 3 reports V3-004; assert it does).
6. Two referenced packages: a code in either one passes.
7. A module field is checked with its module path.
8. Packages and element groups get no row; the `snake` and `diabetes-ckd` examples raise no new row.
Run only the new file plus `ccis-validator-concept`, `-owner`, `-terminology`, `-element` tests and `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module, 1 test, 3 small edits, 1 bundle. If the module passes about 75 lines, stop and split.

## Safety notes
Identity checks only: no numbers, bands or thresholds. A package never says what to do. Messages are plain, untrusted text on display.

## Context impact
Reads about 6 KB of source; adds about 50 source lines and a test of about 90 lines. Next row after this: B2a.
