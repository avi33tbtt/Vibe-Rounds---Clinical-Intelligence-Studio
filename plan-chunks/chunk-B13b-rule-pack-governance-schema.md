# Chunk B13b: rule-pack schema, optional governance members

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B13 (DONE). Decisions: ED-2, ED-46 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 7 and 9. Before B13c.

## What exists
`ccis/schemas/rule-pack.schema.json` (B10, B11): closed pack object, required `manifest class scope inputs parameters rules tests provenance sourceNote`, `$defs` `text key conceptKey expr`, `provenance` is only `{origin, basis}`. The shared manifest schema is closed and has no author, owner or reviewer member. The design (section 7) wants author, owner and a reviewer record per version. Validators are generated: `npm run build` rewrites `ccis/validators/ccis-validators.js` and `index.html` is rebuilt with `node scripts/build-app.js`.

## What to build
Schema only. One new optional top-level member `governance` on the rule pack:
```
governance: { author, owner, reviewers: [ { name, version, date } ] }
```
all closed objects (`additionalProperties: false`, `^x-` allowed). New `$defs.person`: string, pattern `^\S.{0,119}$`. `author` and `owner` are `person` (`owner` may be the word `unassigned`; whether that is acceptable is B13c's job, not the schema's). `reviewers`: 0 to 20 items, each `name` (person), `version` (reference `manifest.schema.json#/$defs/semver`), `date` (pattern `^\d{4}-\d{2}-\d{2}$`), all required. The schema checks form only. Rules about who may differ from whom, and that a clinical prompt needs a named owner, are B13c's layer 8 rules.

ED-46 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) the members live in a new top-level `governance` object, not in the shared manifest (the manifest schema is shared by every artefact type; changing it is a larger decision); (b) `governance` is optional in the schema (additive: every existing pack still validates), and B13c makes the parts that matter required by layer 8; (c) a reviewer record names the exact version it reviewed and a date, so a review of an older version cannot be read as a review of this one; (d) names are plain text up to 120 characters, no role or identity checking.

## Files to read
`ccis/schemas/rule-pack.schema.json` the `required` list and `$defs` (about 60 lines); `scripts/build-validators.js` lines 1 to 20 (how to regenerate); `test/ccis-rule-pack-rules-schema.test.js` lines 1 to 25 (fixture; copy).

## Files to change
`ccis/schemas/rule-pack.schema.json`; regenerated `ccis/validators/ccis-validators.js`; new `test/ccis-rule-pack-governance-schema.test.js`; rebuilt `index.html`; CCIS tracker row `ECO-B13b`. Studio: ED-46. No validator module changes, no import pin change.

## Tests (at most 8 cases)
1. A pack with a full `governance` (author, owner, one reviewer) passes.
2. A pack with no `governance` still passes.
3. `owner: unassigned` passes at schema level.
4. A reviewer without `version`, and one without `date`, are refused.
5. A malformed date (`2026-1-2`) and a non-version string are refused.
6. An unknown member inside `governance` is refused; an `x-` member is accepted.
7. An empty author and a 121-character name are refused; 120 characters pass.
8. 21 reviewers are refused, 20 pass.
Run the new file plus `ccis-rule-pack-schema`, `ccis-rule-pack-rules-schema`, `ccis-validator-rulepack*`, `ccis-validator.test.js`, `ccis-app-build-base` and `node scripts/build-validators.js --check`. At most 3 breakage checks.

## Size guard
1 schema edit, 1 test, 2 regenerated files. If the regenerated validators change anything outside the rule-pack validator, or any existing test needs a change, stop and split.

## Safety notes
Form only. A name in a reviewer record is a claim, not proof of review or independence (design section 9, ED-19). No clinical content.

## Context impact
Reads about 5 KB; adds about 25 schema lines and 90 test lines. Next: B13c.
