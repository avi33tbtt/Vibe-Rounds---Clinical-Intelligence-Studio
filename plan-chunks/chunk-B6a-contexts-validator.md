# Chunk B6a: pathway validator, `contexts` rules

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B6 (DONE). Decisions: ED-5, ED-35 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` section 4. Before B7 and B8.

## What to build
New pure module `ccis-validator-contexts.js` (about 60 lines): `CONTEXTS_RULES` and `contextsChecks(doc, ctx)`. Layer 5, called for kind `pathway`. Skips everything when there is no `contexts`. An element that did not resolve is skipped for the tag check, never guessed. Imports `KNOWN_CONTEXTS` from `ccis-validator-concept.js` (one source for the tag list). Copy `isObj`, `arr`, `diag`, `keysOf`-style helpers; do not import them.
1. `CCIS-V5-044` error: a context lists an element id that is not in the pathway `elements`. Path `/contexts/<i>/elements/<j>`. Needs no resolution.
2. `CCIS-V5-045` error: two contexts share an `id`. Path `/contexts/<i>/id` at the second.
3. `CCIS-V5-046` error: a context `id` is not in `KNOWN_CONTEXTS`. Path `/contexts/<i>/id`. Hint lists the known tags.
4. `CCIS-V5-047` error: disagreement with the element's own tags (design section 4: "the package list wins for display; a mismatch is a validation error"). Let T be the union of `contexts` tags over the element's fields and P the ids of the pathway contexts that list the element. When T is not empty and P is not empty and T differs from P, report at `/contexts/<i>/elements/<j>` of the first context that lists it. Message names the element id and the two tag sets, sorted.
Messages name ids and tags only.

ED-35 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) new module instead of growing `ccis-validator-entryform.js`, which would pass the 90-line guard (the row said "1 module edit"); (b) an element with no field tags, or listed in no context, is not compared (nothing to disagree with); (c) a repeated element id inside one context is not a rule (the schema may already bound it; not re-checked); (d) every rule applies at any standard version.

## Files to read (line ranges)
- `ccis-validator.js` lines 28 to 34 (imports), 56 to 62 (rule list), 604 (layer 5 hook).
- `ccis-validator-concept.js` lines 1 to 40 (`KNOWN_CONTEXTS`, how field tags are read).
- `ccis-validator-entryform.js` lines 1 to 30 (module style; copy helpers).
- `test/ccis-validator-entryform.test.js` lines 1 to 30 (fixture and element resolution; copy).

## Files to change
New `ccis-validator-contexts.js`; `ccis-validator.js` (import, rule list, one hook entry); new `test/ccis-validator-contexts.test.js`; `index.html` rebuilt (`npm ci` first); CCIS tracker row `ECO-B6a`. Studio: `DECISIONS.md` `ED-35`, tracker row B6a.

## Tests (at most 8 cases, `node --test`)
1. A valid pathway gives no V5-044 to 047 row; a pathway with no `contexts` gives none.
2. V5-044 (element not in `elements`), path to the second list item.
3. V5-045 (duplicate id) at the second index.
4. V5-046 (unknown tag) and the hint lists the known tags.
5. V5-047 fires when field tags `ckd` and the context is `diabetes`.
6. V5-047 stays silent: element with no field tags; element in no context; tags equal.
7. Unresolved element: V5-044 to 046 still fire, V5-047 skipped, no crash.
8. A hostile id appears only inside message text.
Run only the new file plus `ccis-validator-entryform`, `ccis-validator-concept`, `ccis-pathway-entryform-schema`, `ccis-demo-pathway`, `ccis-market-diabetes-ckd`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 new module (about 60 lines), 1 test, 3 small wiring edits. If the module passes about 90 lines or an existing test changes, stop and split.

## Safety notes
Grouping labels only. A context never says what to do for a condition. No clinical logic or number. Nothing is inferred from labels. Messages are plain, untrusted text on display.

## Context impact
Reads about 6 KB; adds about 60 source lines and a test of about 100 lines. Next: B7.
