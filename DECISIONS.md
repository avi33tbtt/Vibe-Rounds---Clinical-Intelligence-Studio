# Decisions (ED rows)

Prototype. Synthetic data only. Not for clinical use.

Status: all rows OPEN. Answer in chat ("accept all" is allowed). Claude records each answer at the start of the next row, then marks the row ANSWERED with the date.

| ID | Status | Question | Recommendation | Blocks |
|----|--------|----------|----------------|--------|
| ED-1 | OPEN | What does "trajectory" mean: design time (a pathway) or use time (what accumulates for one patient across pathways)? | Both words stay, with separate meanings. A pathway is designed. A trajectory is what accumulates for a patient and is keyed by concept. | A2, A5, D14 to D16 |
| ED-2 | OPEN | Who is the named clinician owner for DM and CKD content, analytics parameters and any clinical rule? | Name one person. Until then everything is demo content and every threshold or interval stays a disabled parameter. | C7 to C14, E5 to E11, F12, D23 |
| ED-3 | OPEN | Where is the Marketplace hosted, what is the repo called, and who can merge? | A public GitHub repo (`clinical-intelligence-marketplace`) with Pages. You merge. | C0 to C3 |
| ED-4 | OPEN | What is the Studio technology stack? | Static and serverless, same as CCIS: one hashed inline script and a strict CSP. | F1 to F3 |
| ED-5 | OPEN | What is the source of truth for pathways? | Published CCIS artefacts. The EMR materialises element sets and pathway versions from them. | A5, D4 to D8 |
| ED-6 | OPEN | What rule scope does version 1 allow? | Data-quality rules only. Clinical prompts wait for ED-2. | A6, B10 to B14, D19 to D23, F12 |
| ED-7 | OPEN | Which terminology is used? | Local vocabulary only, as decided on 2026-11-25. The binding slot is kept for later. | A3d, C4 |
| ED-8 | OPEN | What are the final names of "CCIS" and the products? | Decide before any public release. No work needs it until G9. | G9 |
| ED-9 | OPEN | How are concept keys represented? | As terminology bindings in a `local` code system, not as a new artefact type. | A3d, B1, C4 |
| ED-10 | OPEN | Where is the first content authored? | In the CCIS repo under `ccis/examples/ecosystem/`, with the SDK and `market-build`, then synced to the Marketplace. The Studio takes over later. | C4 to C16 |
| ED-11 | OPEN | How do the apps share code? | Copy with a pinned commit and a staleness test, as the EMR does for `contracts/`. No submodules. | D1, D2, F2 |
| ED-12 | OPEN | Who owns a concept that several elements use? | Exactly one owner element per concept. Other elements reference the owner. A second owner is refused at publish. | A3d, B2a, B15, D5, F11a |

## Notes
- A1 writes all twelve rows at once, as the tracker says. The "at most 3 new rows per task" limit in `plan-chunks/01-core.md` applies to later rows.
- ED-2 and ED-3 matter most: they gate content (phase C) and the Marketplace repo (C0).
