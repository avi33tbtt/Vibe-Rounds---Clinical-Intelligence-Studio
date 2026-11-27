# Decisions (ED rows)

Prototype. Synthetic data only. Not for clinical use.

Status: ED-1 to ED-12 ANSWERED on 2026-11-27 with "accept all": each recommendation stands as written. ED-13 to ED-15 were added by A4 and are OPEN. Exception in effect: ED-2 names no clinician, so all content stays demo and every threshold or interval stays a disabled parameter.

| ID | Status | Question | Recommendation | Blocks |
|----|--------|----------|----------------|--------|
| ED-1 | ANSWERED 2026-11-27: accept all | What does "trajectory" mean: design time (a pathway) or use time (what accumulates for one patient across pathways)? | Both words stay, with separate meanings. A pathway is designed. A trajectory is what accumulates for a patient and is keyed by concept. | A2, A5, D14 to D16 |
| ED-2 | ANSWERED 2026-11-27: accept all | Who is the named clinician owner for DM and CKD content, analytics parameters and any clinical rule? | Name one person. Until then everything is demo content and every threshold or interval stays a disabled parameter. | C7 to C14, E5 to E11, F12, D23 |
| ED-3 | ANSWERED 2026-11-27: accept all | Where is the Marketplace hosted, what is the repo called, and who can merge? | A public GitHub repo (`clinical-intelligence-marketplace`) with Pages. You merge. | C0 to C3 |
| ED-4 | ANSWERED 2026-11-27: accept all | What is the Studio technology stack? | Static and serverless, same as CCIS: one hashed inline script and a strict CSP. | F1 to F3 |
| ED-5 | ANSWERED 2026-11-27: accept all | What is the source of truth for pathways? | Published CCIS artefacts. The EMR materialises element sets and pathway versions from them. | A5, D4 to D8 |
| ED-6 | ANSWERED 2026-11-27: accept all | What rule scope does version 1 allow? | Data-quality rules only. Clinical prompts wait for ED-2. | A6, B10 to B14, D19 to D23, F12 |
| ED-7 | ANSWERED 2026-11-27: accept all | Which terminology is used? | Local vocabulary only, as decided on 2026-11-25. The binding slot is kept for later. | A3d, C4 |
| ED-8 | ANSWERED 2026-11-27: accept all | What are the final names of "CCIS" and the products? | Decide before any public release. No work needs it until G9. | G9 |
| ED-9 | ANSWERED 2026-11-27: accept all | How are concept keys represented? | As terminology bindings in a `local` code system, not as a new artefact type. | A3d, B1, C4 |
| ED-10 | ANSWERED 2026-11-27: accept all | Where is the first content authored? | In the CCIS repo under `ccis/examples/ecosystem/`, with the SDK and `market-build`, then synced to the Marketplace. The Studio takes over later. | C4 to C16 |
| ED-11 | ANSWERED 2026-11-27: accept all | How do the apps share code? | Copy with a pinned commit and a staleness test, as the EMR does for `contracts/`. No submodules. | D1, D2, F2 |
| ED-12 | ANSWERED 2026-11-27: accept all | Who owns a concept that several elements use? | Exactly one owner element per concept. Other elements reference the owner. A second owner is refused at publish. | A3d, B2a, B15, D5, F11a |
| ED-13 | OPEN | Fields that hold a dose, target, band, treatment text or restriction: `glucose_ss.ss_dose`, `glucose_ss.ss_notes`, `hba1c_mon.target`, `foot_ulcer.ulcer_abx`, `dka_ctx.dka_ins`, `ckd_ctx.renal_diet`. Convert them or leave them out? | Leave them out of elements. They stay EMR-only until a named clinician owner decides (ED-2). Elements hold measurements and facts. | A4a, A5, C5 to C9 |
| ED-14 | OPEN | `dm_ctx.dm_duration` stores a number of years that goes stale. Keep it as entered or replace it with a diagnosis date? | Keep it as entered in v0 (`quantity`, unit `a`) so conversion loses nothing. Consider a diagnosis date later. | A4a, C7 |
| ED-15 | OPEN | The EMR uses mg/dL for glucose and creatinine. Keep EMR units as the one unit per concept, or choose others? | Keep the EMR units in v0, written as UCUM. Nothing converts between units. | A4a, B2, C4 |

## Notes
- A1 writes all twelve rows at once, as the tracker says. The "at most 3 new rows per task" limit in `plan-chunks/01-core.md` applies to later rows.
- ED-2 and ED-3 matter most: they gate content (phase C) and the Marketplace repo (C0).
