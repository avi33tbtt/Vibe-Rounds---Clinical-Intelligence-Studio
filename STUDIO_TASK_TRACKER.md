# Studio: Agent Protocol and Master Task Tracker (EMR + CCIS + Marketplace + Studio)

Repo: https://github.com/avi33tbtt/Vibe-Rounds---Clinical-Intelligence-Studio
Other repos this tracker drives: CCIS https://github.com/avi33tbtt/Vibe-Rounds---CCIS , EMR https://github.com/avi33tbtt/Vibe-Rounds---EMR-Enhancer-App , Marketplace (to be created by you in row C0, proposed name `clinical-intelligence-marketplace`).
Plan: `docs/ECOSYSTEM-PLAN.html`. Conventions: `plan-chunks/01-core.md`. Decisions: `DECISIONS.md`.
Prototype. Synthetic data only. Not for clinical use.

---

## PROTOCOL (instructions for Claude)

Trigger phrase: **"do next"**. Keep the session small: this protocol is written for a free-tier limit.

1. **Refresh first**: `git clone --depth 1` the Studio repo and the ONE other repo named in the next row's Repo column (CCIS, EMR or Marketplace). If a clone fails, stop. Do not pick a task or write anything before both succeed.
2. **Read only these**: this file, `plan-chunks/01-core.md`, the chunk file of the row (if any), and from the target repo only its own conventions file (CCIS: `plan-chunks/01-core.md`; EMR: `emr-plan-chunks/01-core.md`). Do not read the other repo's tracker or `DECISIONS.md` unless the chunk names a range. For source files over 5 KB use `grep -n` and `sed -n` on the ranges the chunk names.
3. **Pick the next row**: the first row in table order with status `TODO` or `IN PROGRESS` whose **Needs** are all `DONE`. ONE row per "do next".
4. **Locks and owner rows**: phases B to G are `LOCKED` until you say **"start phase B"** (and so on). Never start a `LOCKED` row. Rows marked **(YOU)** are yours: Claude never marks them `DONE`; you say "done <id>".
5. **Chunk file first**: a docs-only row is done inline. Any other row without a chunk file: the task is to write `plan-chunks/chunk-<id>-<slug>.md` (docs only, under 5 KB: what to build, files to read with line ranges, files to change, tests, size guard, safety notes, context impact) from the row, the plan page, `DECISIONS.md` and what earlier rows learned. Then stop and ask you to approve it. The next "do next" builds it. Never edit a chunk file after its row has started: add a new row instead.
6. **Size check**: if the row cannot finish within its size guard or the target repo's budget rule, STOP, say which part to split off, add it as a new row (`B7a`). Do not half-build. Do not implement later rows.
7. **The target repo's conventions win**: its `01-core.md`, migration, route and test rules (EMR: next free migration number, permission matrix and audit tests, no DELETE or TRUNCATE grants, targeted tests; CCIS: pure modules, canonical JSON, CHANGELOG amendments with your approval, at most 3 `DECISIONS.md` rows). This file only decides which row runs and how the result is delivered. If the plan is ambiguous, take the most reasonable option, add at most 3 rows to `DECISIONS.md` (prefix `ED-`) and say so in the summary.
8. **Verify lightly**: run only the new or edited test files plus the safety files the target repo names. EMR needs PostgreSQL (`TEST_DATABASE_URL`, a scratch database). CCIS needs Node 22 or newer. At most 3 deliberate-breakage checks (undo a fix, see the new test fail, restore). If tests could not run, write that in the notes. Never claim a pass that was not seen.
9. **Deliver as one zip** named `eco-<id>-<date>.zip` (for example `eco-B1-2026-11-28.zip`). Top-level folders are `repo-studio/`, `repo-ccis/`, `repo-emr/`, `repo-marketplace/`; inside each, the repo-relative paths of new or changed files only. Always include `repo-studio/STUDIO_TASK_TRACKER.md` updated (status `DONE`, date, files, ONE line of notes, 200 characters max) and `repo-studio/DECISIONS.md` if changed. List the zip (`unzip -l`) and present it. Summary: 2 to 4 sentences: what was built, what needs your attention, the next row, and one line: "Extract each folder at its repo root, commit, push."
10. **Mirror rule**: when a row changes CCIS or EMR, also add ONE `DONE` row to that repo's own tracker: ID `ECO-<id>`, plan file `(Studio tracker)`, one-line task and notes. That keeps its tracker truthful. Never edit existing chunk files in any repo.
11. **One change in flight per repo**: commit and push a zip before starting another row on the same repo. The CCIS and EMR chats may keep running their own rows between ecosystem rows; pull first.
12. **Never mark `DONE`** unless implemented and verified. **Never invent clinical content**: all clinical wording is synthetic demo content until a named clinician owner reviews it (ED-2).
13. **Free-tier budget**: read at most 5 files per task besides this tracker; a chunk file stays under 5 KB; a row creates at most 3 new modules, or 1 module plus 1 test plus small assets; a test file has at most 8 cases; no row adds more than one dependency, and a new dependency needs your OK first. Completed rows move to `docs/TRACKER-ARCHIVE.md` when this file passes 40 KB: do not read the archive.

Other commands: `status` (summary only), `redo <id>`, `skip <id>`, `split <id>`, `start phase <X>`, `done <id>`.

If a chat ends midway, nothing is `DONE` until verified: open a new chat, attach this file and say "do next".

---

## STATUS LEGEND
`TODO` | `IN PROGRESS` | `DONE` | `BLOCKED` | `SKIPPED` | `SPLIT` | `LOCKED`

## BUILD ORDER (critical path)

`A` decide and define -> `B` CCIS contract (standard 0.2.0) -> `C` content and Marketplace -> then `D` EMR, `E` CCIS analytics and `F` Studio can run in any order -> `G` integration and hardening.
Why this order: the Studio, the EMR import and the analytics all depend on the contract. Do not scaffold the Studio before B is DONE: it would hard-code a contract that changes.
Known gaps this plan closes: two meanings of "pathway", two element models, no concept key on EMR observations, hard-coded condition contexts (`context-map.js`), no element-group or rule-pack artefact in CCIS, and executor operators `first last min max coalesce` missing (CCIS finding F-9).

## DECISIONS NEEDED (row A1 writes these into `DECISIONS.md`)

| ID | Decision | Recommendation | Blocks |
|----|----------|----------------|--------|
| ED-1 | What "trajectory" means: design time (pathway) vs use time (what accumulates for a patient across pathways) | Yes: pathway is designed, trajectory is keyed by concept | A2, A5, D14 to D16 |
| ED-2 | Clinician owner for DM and CKD content, analytics parameters and any clinical rule | Name one; until then everything is demo and thresholds stay disabled | C7 to C14, E5 to E11, F12, D23 |
| ED-3 | Marketplace host, repo name, who can merge | GitHub repo plus Pages; you merge | C0 to C3 |
| ED-4 | Studio stack | Static, serverless, same as CCIS (hashed script, strict CSP) | F1 to F3 |
| ED-5 | Source of truth for pathways | Published CCIS artefacts; the EMR materialises sets and pathway versions from them | A5, D4 to D8 |
| ED-6 | Rule scope in version 1 | Data-quality rules only; clinical prompts wait for ED-2 | A6, B10 to B14, D19 to D23, F12 |
| ED-7 | Terminology | Local vocabulary only, as decided on 2026-11-25; binding slot kept | A3d, C4 |
| ED-8 | Name "CCIS" and product names | Decide before any public release | G9 |
| ED-9 | Concept keys | Implemented as terminology bindings in a `local` code system, not a new artefact type | A3d, B1, C4 |
| ED-10 | Where first content is authored | In the CCIS repo under `ccis/examples/ecosystem/` with the SDK and `market-build`, then synced to the Marketplace; the Studio takes over later | C4 to C16 |
| ED-11 | How apps share code | Copy with a pinned commit and a staleness test (as the EMR does for `contracts/`), no submodules | D1, D2, F2 |
| ED-12 | Concept ownership | Each concept has exactly one owner element; any other element that uses it references the owner, and a second owner is refused at publish | A3d, B2a, B15, D5, F11a |

---

## PHASE A: Decide and define (Studio repo, docs only)

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| A0 | Studio (YOU) | | Upload this tracker, `plan-chunks/01-core.md` and the plan page as `docs/ECOSYSTEM-PLAN.html` to the Studio repo and commit. Then say "done A0". | DONE | 2026-11-27 | none | Uploads confirmed in repo; you said done A0. |
| A1 | Studio | A0 | Write `DECISIONS.md` with ED-1 to ED-12 as OPEN rows (question, recommendation, what it blocks). Stop and ask you to answer. | DONE | 2026-11-27 | docs | DECISIONS.md written: ED-1 to ED-12 all OPEN, awaiting your answers (A1a). |
| A1a | Studio (YOU) | A1 | Answer ED-1 to ED-12 in chat ("accept all" is allowed). Claude records them at the start of the next row. | DONE | 2026-11-27 | none | You said "accept all"; recorded in DECISIONS.md. ED-2 names no clinician: content stays demo. |
| A2 | Studio | A1a | `docs/VOCABULARY.md`: element, element group, pathway, pathway package, trajectory, concept key, context, rule pack, analytics package, marketplace; plus a table mapping EMR terms (element set, pathway version, case_applied_sets) to CCIS terms. | DONE | 2026-11-27 | docs | docs/VOCABULARY.md written. EMR mapping inferred from the plan page; verify in A4. case_applied_sets not described in the plan. |
| A3a | Studio | A2 | `docs/ELEMENT-LIST-V0.md` part 1: tier 1 elements `core.presentation`, `core.vitals`, `core.examination`, `core.assessment-plan`, `core.event-times`, `core.fluid-balance`: fields, types, units, repeat, time meaning, allowed epistemic states, source EMR set. Synthetic, unreviewed. | DONE | 2026-11-27 | docs | docs/ELEMENT-LIST-V0.md part 1. Fields are proposals from the plan page, not read from EMR sets.js. A4 must reconcile. |
| A3b | Studio | A3a | Part 2: `section` items (problems, allergies, medications, investigations, care episode): what each stores, the matching envelope section and EMR tab. They are placeholders in a form, not elements. | DONE | 2026-11-27 | docs | Part 2 appended to ELEMENT-LIST-V0.md. Envelope section and EMR tab names are marked to confirm in A4; not read from repos. |
| A3c | Studio | A3a | Part 3: `dm.core`, `dm.glycaemic-monitoring`, `htn.core`, optional `dm.foot` and `dm.dka`, from the EMR sets and `longitudinal-demo-content.js`. No numbers or bands in definitions. | DONE | 2026-11-27 | docs | Part 3 appended. Fields are proposals from plan page captions; EMR sets and demo file not read. htn.core references core.vitals for BP (ED-12). |
| A3d | Studio | A3c | Part 4: `ckd.core`, `ckd.kidney-function` (split from set `ckd_ctx`) and the concept key table for the `local` vocabulary: creatinine, eGFR, albumin-creatinine ratio, systolic and diastolic pressure, HbA1c, glucose, weight and every other concept shared by two or more elements. Name ONE owner element per concept (the element that defines it); other elements reference it. | DONE | 2026-11-27 | docs | Part 4: CKD elements and 15 concept keys, one owner each. Fluid-volume and event-time may mix series; open for A4/A5. Units unconfirmed. |
| A4 | Studio | A3d | `docs/FIELD-MAP.md`: EMR field model (id, label, type, unit, options, repeat, derived, flag, bindings) to CCIS field model: conversion per property, what is lost, what must be added (cardinality, time meaning, epistemic states). Read EMR `sets.js` and CCIS `element.schema.json` by line range. | DONE | 2026-11-27 | docs | FIELD-MAP.md written. Real EMR sets differ from ELEMENT-LIST-V0 in many fields. ED-13 to ED-15 added (OPEN). Row A4a added to correct the list. |
| A4a | Studio | A4, ED-13 to ED-15 | Correct `docs/ELEMENT-LIST-V0.md` parts 1 to 4 using `docs/FIELD-MAP.md` section 7: CCIS keys and types, remove invented fields, rebuild from real EMR fields, fix the concept table. Record your answers to ED-13 to ED-15 first. | DONE | 2026-11-27 | docs | ELEMENT-LIST-V0.md rewritten from FIELD-MAP: CCIS keys, real EMR fields, 16 concepts. ED-13 to ED-15 recorded. Section columns still to confirm in A5. |
| A5 | Studio | A4a | `docs/PACKAGE-DESIGN.md`: package members, `entryForm` item kinds (`element`, `section`), `contexts`, composition rules (by reference, match by concept, one owner element per concept and per shared field, conflict report format), materialisation into EMR sets and pathway versions, import and review path. | DONE | 2026-11-27 | docs | PACKAGE-DESIGN.md written. Adds rows B1a, B1b. DM+CKD has no shared concept: A7 to reconsider C14. EMR local-binding check open. ED-16, ED-17 added (OPEN). |
| A6 | Studio | A5 | `docs/RULE-PACK-DESIGN.md`: rule classes, inputs, parameters disabled by default, allowed operators, plain-text message format, provenance, review and independence, logging in the EMR, what is prohibited. | TODO | | docs | |
| A7 | Studio (YOU) | A6 | Review A2 to A6 and say "approve phase A". Changes go in as new rows. | TODO | | none | |

## PHASE B: The contract (CCIS repo, standard 0.2.0)

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| B0 | CCIS (YOU) | A7 | Confirm `git tag v0.1.0` is pushed, run the full `npm test` once, and tell Claude the base commit. | LOCKED | | none | |
| B1 | CCIS | B0 | Element field: optional `concept` (a binding in the `local` system) and `contexts` tags; schema, regenerated validators, test. | LOCKED | | 1 schema edit, 1 test | |
| B1a | CCIS | B1 | Element field `ownedBy` (owner element id and field key) for a field that uses a concept it does not own; type and unit must equal the owner's (ED-16); schema, validator, test. | LOCKED | | 1 schema edit, 1 test | |
| B1b | CCIS | B1 | Local terminology package (artefact type `terminology-package`): code system `local`, concept keys, value sets with display text; check how the validator resolves `valueSet`; schema, test. | LOCKED | | 1 schema, 1 test | |
| B2 | CCIS | B1 | Validator rules for `concept` and `contexts` (concept required on repeating numeric fields, unit consistent per concept, known context tags); test. | LOCKED | | 1 module edit, 1 test | |
| B2a | CCIS | B2, B1a | Concept ownership check: pure `conceptOwners(artefacts)` lists each concept with its owner element and its users; a concept with no owner or a second owner is an error at publish and a warning in `validate`; the same concept under different keys across elements is reported. | LOCKED | | 1 module, 1 test | |
| B3 | CCIS | B1 | Element-group schema (CCIS-REQ-023): members, mandatory or optional, owner per shared concept, version constraints; validators; test. | LOCKED | | 1 schema, 1 test | |
| B4 | CCIS | B3 | Validator rules for element groups (REQ-024): match by concept never by label, unresolved shared concept is an error, no duplicate keys; test. | LOCKED | | 1 module edit, 1 test | |
| B5 | CCIS | B3 | Pathway schema: optional `entryForm` (sections, ordered items of kind `element` or `section`, repeat grouping, default visibility) and `contexts`; validators; test. | LOCKED | | 1 schema edit, 1 test | |
| B6 | CCIS | B5 | Validator rules for `entryForm` and `contexts`: every key exists, each field placed once, known section names; test. | LOCKED | | 1 module edit, 1 test | |
| B7 | CCIS | B4 | `ccis-compose.js` part 1: pure shared-concept matching and conflict report (shared concepts, candidate owners, unit conflicts). | LOCKED | | 1 module, 1 test | |
| B8 | CCIS | B7, B6 | `ccis-compose.js` part 2: build a composed group and pathway from owner choices; deterministic bytes; test. | LOCKED | | 1 module edit, 1 test | |
| B9 | CCIS | B0 | Executor operators `first last min max coalesce` (finding F-9) with an operator-set bump; three-valued; test. | LOCKED | | 1 module edit, 1 test | |
| B10 | CCIS | B1 | Rule-pack schema part 1: manifest, scope, class (`data-quality` or `clinical-prompt`), inputs, parameters; validators; test. | LOCKED | | 1 schema, 1 test | |
| B11 | CCIS | B10, B9 | Rule-pack schema part 2: rules as expression trees from the whitelist, plain-text message templates, provenance, tests member; validators; test. | LOCKED | | 1 schema edit, 1 test | |
| B12 | CCIS | B11 | Validator rules for rule packs, layers 3 to 5: references, operator types, parameters default disabled, messages plain text. | LOCKED | | 1 module edit, 1 test | |
| B13 | CCIS | B12 | Validator rules for rule packs, layers 6 to 8: executable safety, test verification, governance (clinical-prompt class needs a named owner and cannot claim validated without a record). | LOCKED | | 1 module edit, 1 test | |
| B14 | CCIS | B13 | `ccis-rules.js`: pure evaluator of a rule pack over a canonical context; returns fired prompts with rule id, version and record refs, never values; test. | LOCKED | | 1 module, 1 test | |
| B15 | CCIS | B6, B13, B2a | Market: index types `element-group` and `rule-pack`, `requires` and `provides` lookups, `market-build` update; `publishArtefact` refuses a second owner of a concept (B2a); test. | LOCKED | | 1 module edit, 1 script edit, 1 test | |
| B16 | CCIS | B8 | SDK builders for element group and `entryForm`; equivalence with JSON files through `compareRoutes`. | LOCKED | | 1 module edit, 1 test | |
| B17 | CCIS | B14 | SDK builder for rule pack; equivalence test. | LOCKED | | 1 module edit, 1 test | |
| B18 | CCIS | B16, B17 | Contract fixtures: two elements sharing a concept, a group, a pathway with `entryForm`, a data-quality rule pack, a second-owner refusal case; hand-calculated compose and rule outputs; test. | LOCKED | | fixtures, 1 test | |
| B19 | CCIS | B18, B1b | Standard version 0.2.0 accepted by validator, compatibility and importer; all 0.1.0 artefacts and fixtures still validate; test. | LOCKED | | small edits, 1 test | |
| B20 | CCIS | B19 | Docs: SPEC amendments with new CCIS-REQ ids, CHANGELOG amendments, TRACEABILITY, README section. Needs your approval of each amendment. | LOCKED | | docs | |
| B21 | CCIS (YOU) | B20 | Run the full `npm test` and `npm run accept`, then `git tag v0.2.0` and push. Say "done B21". | LOCKED | | none | |

## PHASE C: Content and Marketplace

| ID | Repo | Needs | Task | Status | Date | Size | Notes |
|----|------|-------|------|--------|------|------|-------|
| C0 | Marketplace (YOU) | B21 | Create the repo (public), enable Pages, confirm ED-3. Say "done C0". | LOCKED | | none | |
| C1 | Marketplace | C0 | Scaffold: README with banners, `.gitattributes` (`* -text`), `index.json` copied from CCIS `market/`, LICENSE, CONTRIBUTING stub. | LOCKED | | docs + small files | |
| C2 | Marketplace | C1 | CI: check out CCIS at a pinned commit; validate every artefact (eight layers), recompute checksums, refuse any change to a released file. | LOCKED | | 1 workflow, 1 script | |
| C3 | Marketplace | C2 | `docs/PUBLISHING.md`: how a Studio bundle becomes a pull request, naming and immutability rules. | LOCKED | | docs | |
| C4 | CCIS | B21, A3d | Content: `local` concept bindings with one owner element per concept (SDK, ownership check passes) under `ccis/examples/ecosystem/`. | LOCKED | | 1 script, 1 test | |
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
| G1 | Studio | F36, D24, E15 | `docs/E2E-RUNBOOK.md`: Studio bundle, Marketplace, EMR import, apply, envelope, CCIS run, run record back into the EMR. | LOCKED | | docs | |
| G2 | Studio | G1 | Script that runs a Studio bundle through the CCIS command line with no EMR; checked against expected outputs. | LOCKED | | 1 script, 1 test | |
| G3 | EMR | G1 | Script and test for the EMR half: import a Studio bundle from file, export an envelope, import a CCIS run record. | LOCKED | | 1 script, 1 test | |
| G4 | CCIS | G1 | Unlock CCIS 6.2 (signing) for Marketplace artefacts; its own chunk file first. | LOCKED | | per CCIS split | |
| G5 | Studio | G4 | Governance: `CONTRIBUTING.md` with the independence rule (an author is never the reviewer) and a release checklist; mirror in the Marketplace repo. | LOCKED | | docs | |
| G6 | Studio | G1 | Security review checklist for Marketplace content: text injection in displayed fields, checksum and pin checks, replay of old versions. Review is yours. | LOCKED | | docs | |
| G7 | EMR | D25 | Unlock R7c, R7d, R8a to R8d in the EMR tracker with the import and rule paths added to each. | LOCKED | | per EMR rows | |
| G8 | Studio (YOU) | G6 | Clinician review of all demo content; name the clinical owner (ED-2); decide whether clinical prompts may start (ED-6). | LOCKED | | none | |
| G9 | Studio | G8, ED-8 | Release coordination: version table, tags in all four repos, CHANGELOG entries, final name decision. | LOCKED | | docs | |

---

## ITEMS ONLY YOU CAN DO

- **A0:** upload the tracker, conventions file and plan page.
- **A1a:** answer ED-1 to ED-12. ED-2 (clinician owner) and ED-3 (Marketplace host) matter most.
- **B0 and B21:** tag and test CCIS yourself (the full suite is too long for a free session).
- **C0:** create the Marketplace repo and enable Pages.
- **Dependencies:** approve `esbuild` (F3) and `puppeteer-core` (F32) pins; run `npm ci` after pulling.
- **Before any real use:** clinician review, privacy and security review, licence checks, EMR go-live gate (R8e). Nothing here is cleared for clinical use.

## OPEN QUESTIONS
See `DECISIONS.md` (ED rows) and `docs/ECOSYSTEM-PLAN.html`.
