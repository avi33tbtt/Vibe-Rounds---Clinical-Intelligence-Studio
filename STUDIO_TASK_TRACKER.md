# Studio: Agent Protocol and Master Task Tracker (EMR + CCIS + Marketplace + Studio)

Repo: https://github.com/avi33tbtt/Vibe-Rounds---Clinical-Intelligence-Studio
Other repos this tracker drives: CCIS https://github.com/avi33tbtt/Vibe-Rounds---CCIS , EMR https://github.com/avi33tbtt/Vibe-Rounds---EMR-Enhancer-App , Marketplace https://github.com/avi33tbtt/Vibe-Rounds---Marketplace (created; only a LICENSE so far; Pages is still yours to enable in row C0). Later phases (J, K): CIP https://github.com/avi33tbtt/Vibe-Rounds-Contextual-Interaction-Platform (architecture scaffold, contracts and docs, no code yet) and PHR https://github.com/avi33tbtt/Vibe-Rounds---PHR (empty, LICENSE only).
Plan: `docs/ECOSYSTEM-PLAN.html`. Conventions: `plan-chunks/01-core.md`. Decisions: `DECISIONS.md`.
Prototype. Synthetic data only. Not for clinical use.

---

## PROTOCOL (instructions for Claude)

Trigger phrase: **"do next"**. Keep the session small: this protocol is written for a free-tier limit.

1. **Refresh first**: `git clone --depth 1` the Studio repo and the ONE other repo named in the next row's Repo column (CCIS, EMR, Marketplace, CIP or PHR; URLs are in the line above). If a clone fails, stop. Do not pick a task or write anything before both succeed.
2. **Read only these**: this file, `plan-chunks/01-core.md`, the chunk file of the row (if any), and from the target repo only its own conventions file (CCIS: `plan-chunks/01-core.md`; EMR: `emr-plan-chunks/01-core.md`; CIP: `docs/05-development-roadmap.md` and `contracts/README.md`; PHR: the conventions file row K3 writes). Do not read the other repo's tracker or `DECISIONS.md` unless the chunk names a range. For source files over 5 KB use `grep -n` and `sed -n` on the ranges the chunk names.
3. **Pick the next row**: the first row in table order with status `TODO` or `IN PROGRESS` whose **Needs** are all `DONE`. ONE row per "do next".
4. **Locks and owner rows**: phases B to H are `LOCKED` until you say **"start phase B"** (and so on). Never start a `LOCKED` row. Rows marked **(YOU)** are yours: Claude never marks them `DONE`; you say "done <id>".
5. **Chunk file first**: a docs-only row is done inline. Any other row without a chunk file: the task is to write `plan-chunks/chunk-<id>-<slug>.md` (docs only, under 5 KB: what to build, files to read with line ranges, files to change, tests, size guard, safety notes, context impact) from the row, the plan page, `DECISIONS.md` and what earlier rows learned. Then stop and ask you to approve it. The next "do next" builds it. Never edit a chunk file after its row has started: add a new row instead.
6. **Size check**: if the row cannot finish within its size guard or the target repo's budget rule, STOP, say which part to split off, add it as a new row (`B7a`). Do not half-build. Do not implement later rows.
7. **The target repo's conventions win**: its `01-core.md`, migration, route and test rules (EMR: next free migration number, permission matrix and audit tests, no DELETE or TRUNCATE grants, targeted tests; CCIS: pure modules, canonical JSON, CHANGELOG amendments with your approval, at most 3 `DECISIONS.md` rows). This file only decides which row runs and how the result is delivered. If the plan is ambiguous, take the most reasonable option, add at most 3 rows to `DECISIONS.md` (prefix `ED-`) and say so in the summary.
8. **Verify lightly**: run only the new or edited test files plus the safety files the target repo names. EMR needs PostgreSQL (`TEST_DATABASE_URL`, a scratch database). CCIS needs Node 22 or newer. At most 3 deliberate-breakage checks (undo a fix, see the new test fail, restore). If tests could not run, write that in the notes. Never claim a pass that was not seen.
9. **Deliver as one zip** named `eco-<id>-<date>.zip` (for example `eco-B1-2026-11-28.zip`). Top-level folders are `repo-studio/`, `repo-ccis/`, `repo-emr/`, `repo-marketplace/`, `repo-cip/`, `repo-phr/`; inside each, the repo-relative paths of new or changed files only. Always include `repo-studio/STUDIO_TASK_TRACKER.md` updated (status `DONE`, date, files, ONE line of notes, 200 characters max) and `repo-studio/DECISIONS.md` if changed. List the zip (`unzip -l`) and present it. Summary: 2 to 4 sentences: what was built, what needs your attention, the next row, and one line: "Extract each folder at its repo root, commit, push."
10. **Mirror rule**: when a row changes CCIS or EMR, also add ONE `DONE` row to that repo's own tracker: ID `ECO-<id>`, plan file `(Studio tracker)`, one-line task and notes. That keeps its tracker truthful. Never edit existing chunk files in any repo. CIP and PHR have no tracker yet: add one line under `Unreleased` in their `CHANGELOG.md` instead (K3 creates the PHR one).
11. **One change in flight per repo**: commit and push a zip before starting another row on the same repo. The CCIS and EMR chats may keep running their own rows between ecosystem rows; pull first.
12. **Never mark `DONE`** unless implemented and verified. **Never invent clinical content**: all clinical wording is synthetic demo content until a named clinician owner reviews it (ED-2).
13. **Free-tier budget**: read at most 5 files per task besides this tracker; a chunk file stays under 5 KB; a row creates at most 3 new modules, or 1 module plus 1 test plus small assets; a test file has at most 8 cases; no row adds more than one dependency, and a new dependency needs your OK first. Completed rows move to `docs/TRACKER-ARCHIVE.md` when this file passes 40 KB: do not read the archive.
14. **Integration rule (ED-20)**: the four apps exchange data natively and directly: the CCIS context envelope (EMR to CCIS), the run record (CCIS to EMR) and checksummed CCIS artefacts (Studio, Marketplace, EMR, CCIS). Never design or build a row in phases A to G that needs FHIR: no Questionnaire, Bundle or other FHIR resource as an internal format, a package format or the Studio file format. FHIR is an external export only and is built last, in phase H, which stays `LOCKED` until you say "start phase H". If a row seems to need FHIR, stop and ask.

Other commands: `status` (summary only), `redo <id>`, `skip <id>`, `split <id>`, `start phase <X>`, `done <id>`.

If a chat ends midway, nothing is `DONE` until verified: open a new chat, attach this file and say "do next".

---

## STATUS LEGEND
`TODO` | `IN PROGRESS` | `DONE` | `BLOCKED` | `SKIPPED` | `SPLIT` | `LOCKED`

## BUILD ORDER (critical path)

`A` decide and define -> `B` CCIS contract (standard 0.2.0) -> `C` content and Marketplace -> then `D` EMR, `E` CCIS analytics and `F` Studio can run in any order -> `G` integration and hardening; `H` external FHIR export comes last and is not needed for 0.1.0. After `G1`, three later phases may run in any order and none is needed for 0.1.0: `I` external terminology maps (SNOMED CT, LOINC) on top of the local codes, `J` the Contextual Interaction Platform (CIP), `K` the patient health record (PHR). Phase A row `A10` (terminology design) comes before `B1`.
Why this order: the Studio, the EMR import and the analytics all depend on the contract. Do not scaffold the Studio before B is DONE: it would hard-code a contract that changes.
Known gaps this plan closes: two meanings of "pathway", two element models, no concept key on EMR observations, hard-coded condition contexts (`context-map.js`), no element-group or rule-pack artefact in CCIS, and executor operators `first last min max coalesce` missing (CCIS finding F-9).

## DECISIONS NEEDED (row A1 writes these into `DECISIONS.md`)

| ID | Decision | Recommendation | Blocks |
|----|----------|----------------|--------|
| ED-1 | What "trajectory" means: design time (pathway) vs use time (what accumulates for a patient across pathways) | Yes: pathway is designed, trajectory is keyed by concept | A2, A5, D14 to D16 |
| ED-2 | Clinician owner for DM and CKD content, analytics parameters and any clinical rule | Name one; until then everything is demo and thresholds stay disabled | C7 to C14, E5 to E11, F12, D23 |
| ED-3 | Marketplace host, repo name, who can merge | GitHub repo plus Pages; you merge. Repo: https://github.com/avi33tbtt/Vibe-Rounds---Marketplace | C0 to C3 |
| ED-4 | Studio stack | Static, serverless, same as CCIS (hashed script, strict CSP) | F1 to F3 |
| ED-5 | Source of truth for pathways | Published CCIS artefacts; the EMR materialises sets and pathway versions from them | A5, D4 to D8 |
| ED-6 | Rule scope in version 1 | Data-quality rules only; clinical prompts wait for ED-2 | A6, B10 to B14, D19 to D23, F12 |
| ED-7 | Terminology | Local vocabulary only, as decided on 2026-11-25; binding slot kept | A3d, C4 |
| ED-8 | Name "CCIS" and product names | Decide before any public release | G9 |
| ED-9 | Concept keys | Implemented as terminology bindings in a `local` code system, not a new artefact type | A3d, B1, C4 |
| ED-10 | Where first content is authored | In the CCIS repo under `ccis/examples/ecosystem/` with the SDK and `market-build`, then synced to the Marketplace; the Studio takes over later | C4 to C16 |
| ED-11 | How apps share code | Copy with a pinned commit and a staleness test (as the EMR does for `contracts/`), no submodules | D1, D2, F2 |
| ED-12 | Concept ownership | Each concept has exactly one owner element; any other element that uses it references the owner, and a second owner is refused at publish | A3d, B2a, B15, D5, F11a |
| ED-20 | How do the apps exchange data? "EMR and CCIS talk with FHIR" must not mean FHIR-based exchange. | ANSWERED 2026-11-27 (your instruction): native and direct inside the ecosystem (envelope, run record, CCIS artefacts). FHIR only as an external export, built late (phase H). | H0 to H6, G1, G3 |
| ED-21 | The EMR already has FHIR inbound paths (R6e Questionnaire import, R6f Bundle import, `sdc.js`) and CCIS has LOCKED FHIR Mode rows (6.1 to 6.4). Keep them? | OPEN: keep them as they are, as optional external-edge tools. No ecosystem row depends on them and nothing new is built on them. Decide in H0. | H0 |
| ED-22 | How do local codes grow into SNOMED CT, LOINC and other terminologies later? | ANSWERED 2026-11-27 (your instruction): local keys are the identity and never change; external codes arrive later as separate `terminology-map` artefacts; local-only stays valid. Systems and licences are chosen in I0. | A10, B1, B1b, C4, I0 to I9, H3 |
| ED-23 | What is CIP's role and place in the ecosystem? | OPEN: the collection layer (asks, collects, hands over as proposals); the EMR decides what enters the record; native contracts only (ED-20). Fifth app. | J0 to J7, K7 |
| ED-24 | What is the PHR: who uses it, where the data lives, who holds consent? | OPEN: a patient-owned app on the same elements, pathways and concept keys; shares with a clinic only through CIP and EMR review; patient entries are never verified. | K0 to K9 |

---

## PHASE A: Decide and define (Studio repo, docs only)

Rows A0 to A8 are DONE (2026-11-27) and moved to `docs/TRACKER-ARCHIVE.md`. Do not read the archive.

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| A9 | Studio | A8 | Audit the terminology plan and the two new repos; record ED-22 to ED-24; add A10 and phases I, J, K; edit B1, B1b, H3, G9. | DONE | 2026-11-27 | docs | Local codes were half planned; no SNOMED CT or LOINC path. Added A10 and phase I. CIP and PHR added as phases J, K. ED-22 answered; ED-23, ED-24 OPEN. |
| A10 | Studio | A9 | `docs/TERMINOLOGY-DESIGN.md`: local code rules (stable keys, a key never changes meaning or is reused, deprecate with `replacedBy`, display text, versioned value sets, who may add a key); how the B1b `local` package and EMR `local-terms.js` fit; the extension path (external codes only in separate `terminology-map` artefacts keyed by concept key, mapping status, licence and attribution notes for SNOMED CT and LOINC; codes copied only from an official release file, never typed from memory); local-only always stays valid; how the one binding slot holds a concept and a value set. Read CCIS `terminology-binding.schema.json` and EMR `local-terms.js` by line range. | DONE | 2026-11-27 | docs | `docs/TERMINOLOGY-DESIGN.md` written. Open point for B1b: binding schema needs `code`, so a valueSet-only binding is invalid today. EMR `local_terms` has no `replacedBy`. |

## PHASE B: The contract (CCIS repo, standard 0.2.0)

Phase B unlocked 2026-11-27 (you said "start phase B"). Row B0 is yours; B1 onward waits for "done B0".

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| B0 | CCIS (YOU) | A7 | Confirm `git tag v0.1.0` is pushed, run the full `npm test` once, and tell Claude the base commit. | DONE | 2026-11-27 | none | Marked done on your instruction. Base commit 57d89cd. No v0.1.0 tag on remote when checked; npm test not run by Claude. |
| B1 | CCIS | B0, A10 | Element field: optional `concept` (a binding in the `local` system) and `contexts` tags; schema, regenerated validators, test. | DONE | 2026-11-27 | 1 schema edit, 1 test | Schema + regenerated validators + test (8 cases) built; 171 tests in 4 files pass; 2 breakage checks seen. Key format and known tags wait for B2. Full npm test not run. |
| B1c | CCIS | B1 | SDK field builder accepts `concept` and `contexts` (same rules); equivalence with JSON through `compareRoutes`; one test. Before C4. | DONE | 2026-11-27 | 1 module edit, 1 test | SDK field builder takes concept and contexts; new test (8 cases), 32 tests in 4 files pass; 2 breakage checks seen. Full npm test not run. Next: B1a. |
| B1a | CCIS | B1 | Element field `ownedBy` (owner element id and field key), schema only; requires `concept`. Chunk file: `plan-chunks/chunk-B1a-field-owned-by.md`. The owner type and unit check moved to B1e, the SDK to B1d. | DONE | 2026-11-28 | 1 schema edit, 1 test | Schema `ownedBy` + regenerated validators + test (8 cases); 179 tests in 5 files pass; 2 breakage checks seen. Full npm test not run. Next: B1d. |
| B1d | CCIS | B1a | SDK field builder accepts `ownedBy` (same rules); JSON and code routes identical through `compareRoutes`; one test. Before C4. | DONE | 2026-11-28 | 1 module edit, 1 test | `ccis-sdk-element.js` +ownedBy; new test (7 cases); 39 tests in 5 files pass; 2 breakage checks seen. Full npm test not run. |
| B1e | CCIS | B1a | Validator layer 5, new pure `ccis-validator-owner.js`: owner field exists and has the same concept; type and units equal the owner's (ED-16); wider cardinality is a warning; unresolved owner is skipped. Before B2a. | DONE | 2026-11-28 | 1 module, 1 test | New ccis-validator-owner.js (V5-014 to 016) + test (8 cases); 91 tests in 4 files pass; 2 breakage checks seen; index.html rebuilt. Old failures: validator.test.js 103, 114. |
| B1b | CCIS | B1, A10 | Local terminology package (artefact type `terminology-package`): code system `local`, concept keys, value sets with display text; check how the validator resolves `valueSet`; no external code (SNOMED CT, LOINC) lives here, see I1; schema, test. | DONE | 2026-11-28 | 1 schema, 1 test | terminology-package schema + wiring; 138 tests in 5 files, 2 fail (old, validator.test.js). 2 breakage checks seen. maxLength avoided (breaks generated code). Next: B1f. |
| B1f | CCIS | B1b | Validator checks for a terminology package, new pure module: duplicate concept keys, `replacedBy` naming no concept or itself, value-set member keys absent from `concepts`, duplicate members. Before B2. | TODO | | 1 module, 1 test | Split from B1b. |
| B2 | CCIS | B1, B1f | Validator rules for `concept` and `contexts` (concept required on repeating numeric fields, unit consistent per concept, known context tags); test. | TODO | | 1 module edit, 1 test | |
| B2a | CCIS | B2, B1a, B1e | Concept ownership check: pure `conceptOwners(artefacts)` lists each concept with its owner element and its users; a concept with no owner or a second owner is an error at publish and a warning in `validate`; the same concept under different keys across elements is reported. | TODO | | 1 module, 1 test | |
| B3 | CCIS | B1 | Element-group schema (CCIS-REQ-023): members, mandatory or optional, owner per shared concept, version constraints; validators; test. | TODO | | 1 schema, 1 test | |
| B4 | CCIS | B3 | Validator rules for element groups (REQ-024): match by concept never by label, unresolved shared concept is an error, no duplicate keys; test. | TODO | | 1 module edit, 1 test | |
| B5 | CCIS | B3 | Pathway schema: optional `entryForm` (sections, ordered items of kind `element` or `section`, repeat grouping, default visibility) and `contexts`; validators; test. | TODO | | 1 schema edit, 1 test | |
| B6 | CCIS | B5 | Validator rules for `entryForm` and `contexts`: every key exists, each field placed once, known section names; test. | TODO | | 1 module edit, 1 test | |
| B7 | CCIS | B4 | `ccis-compose.js` part 1: pure shared-concept matching and conflict report (shared concepts, candidate owners, unit conflicts). | TODO | | 1 module, 1 test | |
| B8 | CCIS | B7, B6 | `ccis-compose.js` part 2: build a composed group and pathway from owner choices; deterministic bytes; test. | TODO | | 1 module edit, 1 test | |
| B9 | CCIS | B0 | Executor operators `first last min max coalesce` (finding F-9) with an operator-set bump; three-valued; test. | TODO | | 1 module edit, 1 test | |
| B10 | CCIS | B1 | Rule-pack schema part 1: manifest, scope, class (`data-quality` or `clinical-prompt`), inputs, parameters; validators; test. | TODO | | 1 schema, 1 test | |
| B11 | CCIS | B10, B9 | Rule-pack schema part 2: rules as expression trees from the whitelist, plain-text message templates, provenance, tests member; validators; test. | TODO | | 1 schema edit, 1 test | |
| B12 | CCIS | B11 | Validator rules for rule packs, layers 3 to 5: references, operator types, parameters default disabled, messages plain text. | TODO | | 1 module edit, 1 test | |
| B13 | CCIS | B12 | Validator rules for rule packs, layers 6 to 8: executable safety, test verification, governance (clinical-prompt class needs a named owner and cannot claim validated without a record). | TODO | | 1 module edit, 1 test | |
| B14 | CCIS | B13 | `ccis-rules.js`: pure evaluator of a rule pack over a canonical context; returns fired prompts with rule id, version and record refs, never values; test. | TODO | | 1 module, 1 test | |
| B15 | CCIS | B6, B13, B2a | Market: index types `element-group` and `rule-pack`, `requires` and `provides` lookups, `market-build` update; `publishArtefact` refuses a second owner of a concept (B2a); test. | TODO | | 1 module edit, 1 script edit, 1 test | |
| B16 | CCIS | B8 | SDK builders for element group and `entryForm`; equivalence with JSON files through `compareRoutes`. | TODO | | 1 module edit, 1 test | |
| B17 | CCIS | B14 | SDK builder for rule pack; equivalence test. | TODO | | 1 module edit, 1 test | |
| B18 | CCIS | B16, B17 | Contract fixtures: two elements sharing a concept, a group, a pathway with `entryForm`, a data-quality rule pack, a second-owner refusal case; hand-calculated compose and rule outputs; test. | TODO | | fixtures, 1 test | |
| B19 | CCIS | B18, B1b | Standard version 0.2.0 accepted by validator, compatibility and importer; all 0.1.0 artefacts and fixtures still validate; test. | TODO | | small edits, 1 test | |
| B20 | CCIS | B19 | Docs: SPEC amendments with new CCIS-REQ ids, CHANGELOG amendments, TRACEABILITY, README section. Needs your approval of each amendment. | TODO | | docs | |
| B21 | CCIS (YOU) | B20 | Run the full `npm test` and `npm run accept`, then `git tag v0.2.0` and push. Say "done B21". | TODO | | none | |

## PHASE C: Content and Marketplace

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| C0 | Marketplace (YOU) | B21 | Create the repo (public), enable Pages, confirm ED-3. Say "done C0". | LOCKED | | none | Repo already exists (LICENSE only). Still yours: make it public, enable Pages, say "done C0". |
| C1 | Marketplace | C0 | Scaffold: README with banners, `.gitattributes` (`* -text`), `index.json` copied from CCIS `market/`, LICENSE, CONTRIBUTING stub. | LOCKED | | docs + small files | LICENSE already in the repo; do not recreate it. |
| C2 | Marketplace | C1 | CI: check out CCIS at a pinned commit; validate every artefact (eight layers), recompute checksums, refuse any change to a released file. | LOCKED | | 1 workflow, 1 script | |
| C3 | Marketplace | C2 | `docs/PUBLISHING.md`: how a Studio bundle becomes a pull request, naming and immutability rules. | LOCKED | | docs | |
| C4 | CCIS | B21, A3d, B1c, B1d | Content: `local` concept bindings with one owner element per concept (SDK, ownership check passes) under `ccis/examples/ecosystem/`. | LOCKED | | 1 script, 1 test | |
| C5 | CCIS | C4 | Content: `core.presentation`, `core.examination`, `core.assessment-plan`. | LOCKED | | 3 JSON, 1 test | |
| C6 | CCIS | C4 | Content: `core.vitals`, `core.fluid-balance`, `core.event-times`. | LOCKED | | 3 JSON, 1 test | |
| C7 | CCIS | C4 | Content: `dm.core`, `dm.glycaemic-monitoring`. Needs ED-2 wording review before any use. | LOCKED | | 2 JSON, 1 test | |
| C8 | CCIS | C4 | Content: `ckd.core`, `ckd.kidney-function`. | LOCKED | | 2 JSON, 1 test | |
| C9 | CCIS | C4 | Content: `htn.core`. | LOCKED | | 1 JSON, 1 test | |
| C10 | CCIS | C7 | Optional content: `dm.foot`, `dm.dka`. Unlock only if you say so. | LOCKED | | 2 JSON, 1 test | |
| C11 | CCIS | C5, C6 | Pathway package `basic-emr`: core elements, `section` items, `entryForm`. | LOCKED | | 1 JSON, 1 test | |
| C12 | CCIS | C7, C11 | Pathway package `dm`. | LOCKED | | 1 JSON, 1 test | |
| C13 | CCIS | C8, C11 | Pathway package `ckd`. | LOCKED | | 1 JSON, 1 test | |
| C14 | CCIS | C12, C13, B8 | Composed pathway `dm-ckd` built with `ccis-compose.js`; conflict report fixture with hand-calculated expected result. | LOCKED | | 1 JSON, 1 fixture, 1 test | |
| C15 | CCIS | C14, B15 | `market-build` includes the ecosystem content; `--check` passes; `market/` regenerated. | LOCKED | | 1 script edit, generated folder | |
| C16 | Marketplace | C15, C2 | Import the built `market/` folder; CI green. | LOCKED | | generated folder | |

## PHASE D: EMR Enhancer

Migrations use the next free number. Every new route updates the permission matrix and audit tests. Every new table gets the DELETE-forbid trigger and only the grants `emr_app` needs.

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| D1 | EMR | B21 | Refresh `contracts/` from CCIS 0.2.0 (element, pathway, group, rule-pack, manifest) with the commit pin; test that the copy matches. | LOCKED | | docs + files, 1 test | |
| D2 | EMR | D1 | Vendor the CCIS validator, canonical JSON, SHA-256 and compose modules under `vendor/ccis/` with a pinned commit file and a staleness test. | LOCKED | | files, 1 test | |
| D3 | EMR | D2 | Migration: `concept_key` on element fields and on observations (nullable, set at write from the element). Run the migration safety test. | LOCKED | | 1 migration, 1 test | |
| D4 | EMR | D3 | Migration: source artefact id, version and checksum on element set versions and pathway versions; `entry_form` and `contexts` on pathway versions, immutable once published. | LOCKED | | 1 migration, 1 test | |
| D5 | EMR | D2, C16 | Pure `package-import.js` (like `bundle-import.js`): package to draft set definitions, pathway proposal, concept map and blocker report (a concept already owned by another imported element is a blocker); hostile-text checks. | LOCKED | | 1 module, 1 test | |
| D6 | EMR | D5, D4 | Staging server: migration for package imports (append-only), stage, list and read routes, audit, matrix tests. | LOCKED | | 1 migration, 1 routes file, tests | |
| D7 | EMR | D6 | Accept a staged package: create set drafts through the EXISTING set rules, submit for review, keep concept and source columns. | LOCKED | | 1 routes edit, 1 test | |
| D8 | EMR | D7 | Accept the pathway: after sets are published, create the pathway draft with `entry_form` and `contexts` through the existing pathway rules. | LOCKED | | 1 routes edit, 1 test | |
| D9 | EMR | D8 | Page: Import tab "Marketplace package" (file chooser first, no network), proposals, blockers, accept; hostile-string test. | LOCKED | | 1 page module, 1 test | |
| D10 | EMR | D8 | Pure `pathway-form.js`: published pathway version to a form model (sections, fields, `section` items mapped to existing tabs). | LOCKED | | 1 module, 1 test | |
| D11 | EMR | D10 | Page module that renders the pathway form; built-in sets keep the old stacked form. | LOCKED | | 1 page module, 1 test | |
| D12 | EMR | D10 | Composition at apply: a read route returns the composed form for a case with two or more pathways, using the vendored compose module. | LOCKED | | 1 routes file, 1 test | |
| D13 | EMR | D12, D11 | Page: composed form shows shared fields once; test. | LOCKED | | 1 page edit, 1 test | |
| D14 | EMR | D3, D12 | Observations by concept: reads of current value and series by concept across sets; tests with two sets that share a concept. | LOCKED | | 1 routes edit, 1 test | |
| D15 | EMR | D8, D14 | Contexts read from applied pathway `contexts`; `context-map.js` stays as fallback; the R9 tests stay green. | LOCKED | | 1 module edit, 1 test | |
| D16 | EMR | D15 | Longitudinal explorer keyed by concept; missing, unknown and entered-in-error never plotted as zero; test. | LOCKED | | 1 page edit, 1 test | |
| D17 | EMR | D8 | Envelope builder uses source artefact id, version and checksum for imported sets; CCIS envelope fixtures still pass. | LOCKED | | 1 module edit, 1 test | |
| D18 | EMR | D9, C16 | Seed script that imports the built packages through the import path (not direct SQL); reconcile `longitudinal:seed`. | LOCKED | | 1 script, 1 test | |
| D19 | EMR | D8, B14 | Rule pack import and pin to a pathway version (migration, staging, accept through review, append-only). | LOCKED | | 1 migration, 1 routes file, tests | |
| D20 | EMR | D19, D17 | Rule runner: build the context with the existing envelope builder, evaluate pinned rule packs with the vendored `ccis-rules`; data-quality class only; audited; matrix tests. | LOCKED | | 1 routes file, 1 test | |
| D21 | EMR | D20 | `rule_fires` append-only log (rule id, version, case, record refs, time; no values) and list route. | LOCKED | | 1 migration, 1 routes file, 1 test | |
| D22 | EMR | D21 | Page: prompts panel with rule id, version and the records read; dismiss with a reason. | LOCKED | | 1 page module, 1 test | |
| D23 | EMR | D22 | Local enabling of rule parameters by a clinician, recorded. Unlock only if ED-6 allows clinical prompts and ED-2 names an owner. | LOCKED | | 1 migration, 1 routes file, tests | |
| D24 | EMR | D16, D22 | Real-browser acceptance: import, review, publish, apply DM and CKD, composed form, values, trajectory, rule prompt; hook into R8a. | LOCKED | | 1 test file | |
| D25 | EMR | D24 | Docs: README, go-live checklist, and the R8d safety case gain the import and rule hazards. | LOCKED | | docs | |

## PHASE E: CCIS analytics by pathway

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| E1 | CCIS | B21 | Static pathway compatibility `resolveStaticCompat`: the existing checks without data readiness; pure; test. | LOCKED | | 1 module edit, 1 test | |
| E2 | CCIS | E1, B15 | Catalogue model: filter analytics by pathway using the market `requires` lookup; test. | LOCKED | | 1 module edit, 1 test | |
| E3 | CCIS | E2 | Catalogue view shows pathway filter and compatibility reasons; jsdom test, not wired. | LOCKED | | 1 module edit, 1 test | |
| E4 | CCIS | E3 | Wire into the page, rebuild `index.html`. | LOCKED | | small edits, 1 test | |
| E5 | CCIS | C6 | Analytics `core.vitals-documentation-completeness` (SDK, hand-calculated fixtures, validator passes, no threshold). | LOCKED | | 1 JSON, fixtures, 1 test | |
| E6 | CCIS | C7 | Analytics `dm.glycaemic-monitoring-gap`: the interval is a disabled parameter. | LOCKED | | 1 JSON, fixtures, 1 test | |
| E7 | CCIS | C7 | Analytics `dm.glycaemic-documentation-completeness`. | LOCKED | | 1 JSON, fixtures, 1 test | |
| E8 | CCIS | C8 | Analytics `ckd.kidney-function-monitoring-gap`: the interval is a disabled parameter. | LOCKED | | 1 JSON, fixtures, 1 test | |
| E9 | CCIS | C8 | Analytics `ckd.kidney-function-completeness`. | LOCKED | | 1 JSON, fixtures, 1 test | |
| E10 | CCIS | C14 | Analytics `dm-ckd.paired-renal-glycaemic-documentation`: both measured within a window parameter (disabled by default). | LOCKED | | 1 JSON, fixtures, 1 test | |
| E11 | CCIS | B9, E6 | Optional: generic trend-direction analytics for one repeating numeric concept (latest against previous, no threshold). Unlock only if you say so. | LOCKED | | 1 JSON, fixtures, 1 test | |
| E12 | CCIS | E10 | Pathway packages `basic-emr`, `dm`, `ckd`, `dm-ckd` list their analytics; compatibility tests re-run. | LOCKED | | 4 JSON edits, 1 test | |
| E13 | CCIS | E12 | `market-build` includes the analytics; docs for the Marketplace address in the distribution build. | LOCKED | | 1 script edit, docs | |
| E14 | CCIS | E12, D17 | Acceptance: an EMR-exported DM+CKD envelope fixture with hand-calculated results through `npm run accept`. | LOCKED | | fixtures, 1 test | |
| E15 | CCIS | E14 | Docs: LIMITATIONS updates and README ecosystem section. | LOCKED | | docs | |

## PHASE F: Clinical Intelligence Studio (this repo)

Screens follow the CCIS pattern: pure model, then view (jsdom test, not wired), then wiring. Nothing is stored in the browser; work is saved and loaded as one file.

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| F1 | Studio | B21, ED-4 | Scaffold: `package.json`, CI, README with banners, conventions copied from CCIS (check its `package.json` for the test runner). | LOCKED | | small files | |
| F2 | Studio | F1 | `scripts/vendor-ccis.js` pins CCIS v0.2.0 into `vendor/ccis/`; staleness test. | LOCKED | | 1 script, 1 test | |
| F3 | Studio | F2 | Pure SHA-256, esbuild pipeline to one hashed inline script, strict CSP shell page with banners, `--check`. One pinned dependency (esbuild), your OK first. | LOCKED | | 2 modules, 1 script, 1 test | |
| F4 | Studio | F3 | Workspace model: in-memory set of artefacts; save and load as one JSON file; no storage. | LOCKED | | 1 module, 1 test | |
| F5 | Studio | F4 | Catalogue model: load a market folder through the CCIS loader, list artefacts by type and version. | LOCKED | | 1 module, 1 test | |
| F6 | Studio | F5 | Catalogue view and file input; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| F7 | Studio | F6 | Wire the catalogue; rebuild the page. | LOCKED | | small edits, 1 test | |
| F8 | Studio | F7 | Element editor model: form state to SDK `element()` and back; test. | LOCKED | | 1 module, 1 test | |
| F9 | Studio | F8 | Element editor view; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| F10 | Studio | F9 | Wire the element editor; rebuild. | LOCKED | | small edits, 1 test | |
| F11 | Studio | F10, C4 | Concept picker from the `local` bindings (model and view together); jsdom test. | LOCKED | | 1 module, 1 test | |
| F11a | Studio | F11, B2a | Concept ownership report: per concept its owner element and users; duplicate owners and the same concept under different keys flagged; creating an element warns before a second owner is made; jsdom test. | LOCKED | | 1 module, 1 test | |
| F12 | Studio | F10 | Validate panel model: eight layers, compile, checksum; diagnostics by rule id. | LOCKED | | 1 module, 1 test | |
| F13 | Studio | F12, F11a | Validate panel view and wiring, plus the concept ownership report (F11a); rebuild. | LOCKED | | 1 module, small edits, 1 test | |
| F14 | Studio | F13 | Pathway composer model: pick elements and `section` items, order, set contexts, build the pathway through the SDK. | LOCKED | | 1 module, 1 test | |
| F15 | Studio | F14 | Pathway composer view; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| F16 | Studio | F15 | Wire the composer; rebuild. | LOCKED | | small edits, 1 test | |
| F17 | Studio | F16, B8 | Composition model: conflict report, owner choice per shared concept, composed result through `ccis-compose.js`. | LOCKED | | 1 module, 1 test | |
| F18 | Studio | F17 | Composition view; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| F19 | Studio | F18 | Wire composition; rebuild. | LOCKED | | small edits, 1 test | |
| F20 | Studio | F16, E1 | Analytics attach model: static compatibility of a chosen analytics package with the pathway; required or optional. | LOCKED | | 1 module, 1 test | |
| F21 | Studio | F20 | Analytics attach view; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| F22 | Studio | F21 | Wire analytics attach; rebuild. | LOCKED | | small edits, 1 test | |
| F23 | Studio | F16, B17, ED-6 | Rule builder model: a restricted form (inputs, missing, stale, conflicting, interval checks, parameters) to a rule pack through the SDK. | LOCKED | | 1 module, 1 test | |
| F24 | Studio | F23 | Rule builder view; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| F25 | Studio | F24, B14 | Rule tester: run the rule on a synthetic sample context, show the trace and the prompts. | LOCKED | | 1 module, 1 test | |
| F26 | Studio | F25 | Wire the rule builder and tester; rebuild. | LOCKED | | small edits, 1 test | |
| F27 | Studio | F13, F19 | Publish bundle model: artefacts, market index fragment and checksum manifest; no clock; pure. | LOCKED | | 1 module, 1 test | |
| F28 | Studio | F27 | Publish view with a real Blob download; jsdom test. | LOCKED | | 1 module, 1 test | |
| F29 | Studio | F28 | Wire publish; document the pull request step; rebuild. | LOCKED | | small edits, docs, 1 test | |
| F30 | Studio | F29 | Hostile-string sweep over every model-fed view. | LOCKED | | 1 test | |
| F31 | Studio | F30 | Hostile-string sweep over the file-fed views (catalogue, workspace load). | LOCKED | | 1 test | |
| F32 | Studio | F31 | One real-Chrome smoke test. Adds `puppeteer-core`, pinned as in CCIS; your OK first. | LOCKED | | 1 test, 1 dependency | |
| F33 | Studio | F32, C16 | Acceptance: build `dm-ckd` in the Studio flow (jsdom) and compare bytes and checksums with the CCIS-authored content from phase C. | LOCKED | | 1 test | |
| F34 | Studio | F26 | Optional: AI-assisted drafting through the CCIS draft gate. Needs CCIS 4.3b rows DONE. Unlock only if you say so. | LOCKED | | split when reached | |
| F35 | Studio | F33 | `docs/LIMITATIONS.md` (sourced lines) and README for 0.1.0. | LOCKED | | docs | |
| F36 | Studio (YOU) | F35 | Run the full `npm test`, then `git tag v0.1.0` and push. | LOCKED | | none | |

## PHASE G: Integration and hardening

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| G1 | Studio | F36, D24, E15 | `docs/E2E-RUNBOOK.md`: Studio bundle, Marketplace, EMR import, apply, envelope, CCIS run, run record back into the EMR. Native path only; no FHIR anywhere in it (ED-20). | LOCKED | | docs | |
| G2 | Studio | G1 | Script that runs a Studio bundle through the CCIS command line with no EMR; checked against expected outputs. | LOCKED | | 1 script, 1 test | |
| G3 | EMR | G1 | Script and test for the EMR half: import a Studio bundle from file, export an envelope, import a CCIS run record. Test that no FHIR resource is read or written on this path (ED-20). | LOCKED | | 1 script, 1 test | |
| G4 | CCIS | G1 | Unlock CCIS 6.2 (signing) for Marketplace artefacts; its own chunk file first. | LOCKED | | per CCIS split | |
| G5 | Studio | G4 | Governance: `CONTRIBUTING.md` with the independence rule (an author is never the reviewer) and a release checklist; mirror in the Marketplace repo. | LOCKED | | docs | |
| G6 | Studio | G1 | Security review checklist for Marketplace content: text injection in displayed fields, checksum and pin checks, replay of old versions. Review is yours. | LOCKED | | docs | |
| G7 | EMR | D25 | Unlock R7c, R7d, R8a to R8d in the EMR tracker with the import and rule paths added to each. | LOCKED | | per EMR rows | |
| G8 | Studio (YOU) | G6 | Clinician review of all demo content; name the clinical owner (ED-2); decide whether clinical prompts may start (ED-6). | LOCKED | | none | |
| G9 | Studio | G8, ED-8 | Release coordination: version table, tags in every repo built (EMR, CCIS, Studio, Marketplace; CIP and PHR when phases J and K ran), CHANGELOG entries, final name decision. | LOCKED | | docs | |

## PHASE H: External FHIR export (LATE, optional for 0.1.0)

FHIR is only the outward format for people and systems outside the ecosystem (ED-20). Export direction only: nothing here changes how the EMR, CCIS, Studio and Marketplace talk to each other, and internal rows stay the system of record. Starts only when you say "start phase H". Already built in the EMR: pure exporters R6a to R6d (Observation, Condition, AllergyIntolerance, MedicationRequest) and the offline conformance pack R6g-1; none has a route yet.

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| H0 | Studio (YOU) | G3 | Say "start phase H". Decide who the external consumer is and which resources are wanted, and answer ED-21 (keep or retire the FHIR inbound paths). | LOCKED | | none | |
| H1 | Studio | H0 | `docs/FHIR-EXPORT-BOUNDARY.md`: what leaves (native rows through R6a to R6d), export only, how a concept key becomes an external code at export time through the binding slot (a `local` code is never a Coding, as the EMR already rules), what is never exported, loss report, no clinical claim. | LOCKED | | docs | |
| H2 | EMR | H1 | Export routes over the existing exporters: one case or patient to a FHIR Bundle; new permission, audit on every export, matrix and audit tests; reads only. Chunk file first; split if over the guard. | LOCKED | | 1 routes file, permissions, tests | |
| H3 | EMR | H2, I4 | Concept mapping at export: use the verified `terminology-map` content imported in phase I (I4), only for systems you licensed in I0; otherwise codes stay text. Nothing invented. Skip if phase I is not built. | LOCKED | | 1 module edit, 1 test | |
| H4 | EMR (YOU) | H2 | Run the public HL7 validator on the fixtures and one real H2 export (EMR row R6g-2 covers the fixtures; do not duplicate it). Say "done H4" with the result. | LOCKED | | none | |
| H5 | CCIS | H0 | Optional: external export of CCIS results (run record or analytics output). Resource choice is open. CCIS rows 6.1 to 6.4 (FHIR Mode) stay in the CCIS tracker. Unlock only if you say so. | LOCKED | | per CCIS split | |
| H6 | Studio | H4 | Docs: external export step in `docs/E2E-RUNBOOK.md`; `docs/LIMITATIONS.md` lines (not clinically validated, export only, text codes unless H3). | LOCKED | | docs | |

## PHASE I: External terminology maps (LATE, optional for 0.1.0)

Local concept keys stay the identity inside the ecosystem (ED-9, ED-22). SNOMED CT, LOINC or any other system is added later as a separate, versioned `terminology-map` artefact (concept key to external code, status, reviewer, licence note). No element, observation or pathway is rewritten, and local-only always stays valid. Codes are copied from an official release file you supply, never typed from memory. Starts only when you say "start phase I". Native artefacts only (ED-20).

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| I0 | Studio (YOU) | G1, A10 | Say "start phase I". Choose the systems (SNOMED CT, LOINC, others), edition, licence or affiliate status and who holds it, and which concepts go first. This reopens ED-7 for those systems only. | LOCKED | | none | |
| I1 | CCIS | I0, B21 | `terminology-map` artefact schema: local concept key, external system id, version, code, display, mapping status, reviewer, licence note. Additive, so 0.2.0 artefacts still validate. Chunk file first. | LOCKED | | 1 schema, 1 test | |
| I2 | CCIS | I1 | Validator rules: concept key exists in the `local` package, system listed and version pinned, one active map per concept and system, no change to a local key; test. | LOCKED | | 1 module edit, 1 test | |
| I3 | CCIS | I2 | Release-file check: read a code list you supply (for example a SNOMED CT subset or the LOINC table), confirm each map code and display against it. No network, nothing redistributed; test with a tiny synthetic stand-in. | LOCKED | | 1 script, 1 test | |
| I4 | EMR | I2, D3 | Import maps (append-only, migration, matrix tests); the element bindings list gains the external code beside `local`; observations stay keyed by concept; an unmapped concept stays valid. Chunk file first. | LOCKED | | 1 migration, 1 routes file, tests | |
| I5 | Studio | I2, F11a | Map editor, model and view together: pick a concept, choose a code from the imported release file, set status; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| I6 | Studio | I5 | Wire the map editor; rebuild. | LOCKED | | small edits, 1 test | |
| I7 | Marketplace | I2, C2 | CI accepts `terminology-map` artefacts and refuses any file that redistributes licensed code lists. | LOCKED | | 1 script edit, 1 test | |
| I8 | Studio (YOU) | I6, I7 | Licence check and terminologist review of every map. Say "done I8". | LOCKED | | none | |
| I9 | Studio | I8 | Docs: README and `docs/LIMITATIONS.md` lines (local codes by default, maps optional, not clinically verified, licence terms). | LOCKED | | docs | |

## PHASE J: Contextual Interaction Platform, CIP (LATE, optional for 0.1.0)

Repo: https://github.com/avi33tbtt/Vibe-Rounds-Contextual-Interaction-Platform . Scaffold only: README, `docs/00` to `06`, `contracts/` (collection specification, task, response envelope), a synthetic hypertension example, `src/` and `tests/` are placeholders. CIP asks for and collects information (patient forms, AI agents, messaging) and hands it over as proposals. The EMR decides what enters the record. Starts only when you say "start phase J".

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| J0 | Studio (YOU) | G1 | Say "start phase J". Confirm CIP's role and its place as the fifth app; answer ED-23. | LOCKED | | none | |
| J1 | Studio | J0 | `docs/CIP-INTEGRATION.md`: what CIP reads (published pathway and element artefacts, by concept key, checksummed), what it returns (response envelope as a proposed observation with provenance, never a direct write), how a CIP collection specification relates to an element (wording is CIP's, meaning is the element's), how CIP's ADR-0001 ("the EMR owns definitions") fits ED-5, native path only (ED-20), no silent promotion. Read CIP `docs/03-integration-contracts.md`, `contracts/` and the ADR by line range. | LOCKED | | docs | |
| J2 | CIP | J1, B21 | Pin CCIS contract copies into `contracts/` with a commit pin and staleness test (ED-11); add concept key to the response envelope schema. Chunk file first. | LOCKED | | files, 1 schema edit, 1 test | |
| J3 | CIP | J2 | Pure pathway-requirement reader: pathway package and elements to a requirement list by concept key; missing or stale check against an envelope fixture; test. | LOCKED | | 1 module, 1 test | |
| J4 | CIP | J3 | Collection specification generator: element to question item (type, unit, options from the local value set); patient wording is a separate reviewed field; synthetic only; test. | LOCKED | | 1 module, 1 test | |
| J5 | EMR | J4, D8 | Review queue for proposals from CIP: append-only staging, accept or reject through the existing review rules, provenance kept, audit, matrix tests. Chunk file first; split if over the guard. | LOCKED | | 1 migration, 1 routes file, tests | |
| J6 | CIP | J4, J5 | Vertical slice from the CIP README with a mock EMR: requirement, task, form, response, proposal, acknowledgement; synthetic data; test. | LOCKED | | 1 module, 1 test | |
| J7 | Studio | J6 | Docs: CIP section in `docs/E2E-RUNBOOK.md` and `docs/LIMITATIONS.md` line; CIP `CHANGELOG.md` line. | LOCKED | | docs | |

## PHASE K: Patient health record, PHR (LATE, optional for 0.1.0)

Repo: https://github.com/avi33tbtt/Vibe-Rounds---PHR (empty, LICENSE only). Works on the same elements, pathways and concept keys, and helps patients see and record their own information. Shares with a clinic only through CIP and EMR review. Starts only when you say "start phase K".

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| K0 | Studio (YOU) | J1 | Say "start phase K". Answer ED-24: who the patients are, languages, where the data lives, who holds consent, relation to the EMR. | LOCKED | | none | |
| K1 | Studio | K0 | `docs/PHR-DESIGN.md`: which elements a patient may see or enter, patient wording and plain-language help kept apart from clinician labels, trajectory by concept key from the patient's own data (ED-1), sharing through CIP, consent and revocation, privacy, what the PHR never does (diagnose, advise, change a clinician's pathway), patient entries are `patient-reported` and never verified. | LOCKED | | docs | |
| K2 | CCIS | K1, B21 | Optional patient-facing facets on element and pathway (audience, patient label, plain-language help, patient may enter). Additive; unlock only if K1 shows it is needed. Chunk file first. | LOCKED | | 1 schema edit, 1 test | |
| K3 | PHR | K1 | Scaffold: README with banners, conventions file, `CHANGELOG.md`, `.gitattributes`, CI, vendored CCIS pin with staleness test (ED-11). Synthetic data only. Chunk file first. | LOCKED | | small files, 1 test | |
| K4 | PHR | K3 | Pure patient record model: values by concept key, epistemic states, missing never zero; save and load as one file, no browser storage; test. | LOCKED | | 1 module, 1 test | |
| K5 | PHR | K4, K2 | Patient pathway view: published pathway to a patient form model; jsdom test, not wired. | LOCKED | | 1 module, 1 test | |
| K6 | PHR | K5 | Wire the form; trajectory view by concept key; hostile-string test; rebuild. | LOCKED | | small edits, 1 test | |
| K7 | PHR | K6, J5 | Share with a clinic: build a CIP response envelope from entered values with a consent record; the EMR review queue (J5) receives it. | LOCKED | | 1 module, 1 test | |
| K8 | Studio (YOU) | K7 | Privacy, security and clinician review of PHR content, wording and consent text. Say "done K8". | LOCKED | | none | |
| K9 | Studio | K8 | Docs: `docs/LIMITATIONS.md` and release table entries for CIP and PHR. | LOCKED | | docs | |

---

## ITEMS ONLY YOU CAN DO

- **A0:** upload the tracker, conventions file and plan page.
- **A1a:** answer ED-1 to ED-12. ED-2 (clinician owner) and ED-3 (Marketplace host) matter most.
- **B0 and B21:** tag and test CCIS yourself (the full suite is too long for a free session).
- **C0:** the Marketplace repo exists (https://github.com/avi33tbtt/Vibe-Rounds---Marketplace); confirm it is public, enable Pages, say "done C0".
- **H0, H4 and ED-21 (late):** FHIR export is yours to start. Answer ED-21, run the public validator, and decide whether any licensed terminology is wanted (reopens ED-7).
- **I0, I8 (late):** choose terminology systems, hold the licence, review the maps.
- **J0, K0, K8 (late):** confirm CIP's role, define the PHR (users, data custody, consent), review PHR privacy and wording.
- **Dependencies:** approve `esbuild` (F3) and `puppeteer-core` (F32) pins; run `npm ci` after pulling.
- **Before any real use:** clinician review, privacy and security review, licence checks (including any SNOMED CT or LOINC use), EMR go-live gate (R8e). Nothing here is cleared for clinical use.

## OPEN QUESTIONS
See `DECISIONS.md` (ED rows) and `docs/ECOSYSTEM-PLAN.html`.
