# Studio core conventions (read every session)

Prototype. Synthetic data only. Not for clinical use. This file is short on purpose: the target repo's own `01-core.md` adds its rules.

## What this project is
One pathway, four apps. The Clinical Intelligence Studio authors element definitions, pathway packages, analytics references and rule packs. A Marketplace (static, checksummed, immutable) publishes them. The EMR Enhancer pulls pathways and rules and renders the entry form from a pathway. CCIS pulls analytics and runs them. Full plan: `docs/ECOSYSTEM-PLAN.html`.

## Vocabulary (short form; `docs/VOCABULARY.md` is the long one)
- **Element**: a versioned definition of one thing recorded about a patient.
- **Pathway**: a designed, ordered set of elements. The EMR entry form is a pathway.
- **Trajectory**: what accumulates for one patient across pathways, keyed by concept.
- **Concept key**: the shared meaning of a field (for example creatinine), a binding in the `local` code system.
- **Pathway package**: a CCIS pathway artefact with `entryForm`, `contexts`, analytics references and an optional rule pack.
- **Rule pack**: governed rules that run in the EMR and produce visible prompts.

## Standing rules (never break)
1. No patient data in any artefact or Marketplace file. Fixtures are synthetic and say so.
2. No licensed terminology. Local vocabulary only (ED-7). Keep the binding slot.
3. No numbers, bands or thresholds in element definitions. Thresholds are parameters, disabled by default, enabled locally by a clinician.
4. A pathway describes information needed. It never authorises or automates treatment. A rule produces a visible prompt with its id, version and the records it read. It never orders, never changes data silently.
5. Composition is by reference, never by copy. Shared fields are matched by concept key, never by similar label. Each concept has exactly one owner element; any other element that uses it references the owner.
6. Unknown stays unknown. Missing, unknown and entered-in-error are never zero or false.
7. Published versions are immutable. A change is a new version. Checksums use canonical JSON (`ccis-canonical-json/1`).
8. An author is never the reviewer of the same artefact.
9. Text that comes from a package is untrusted: plain text only, escaped on display, no markup, no URLs followed.
10. Every page and doc says: "Prototype. Synthetic data only. Not for clinical use."

## Delivery layout
One zip per row, named `eco-<id>-<date>.zip`, with top-level folders `repo-studio/`, `repo-ccis/`, `repo-emr/`, `repo-marketplace/`. Extract each folder at its repo root.

## Style
Plain sentences, no marketing words. File names lowercase with hyphens. IDs for artefacts use `group.name` (for example `dm.core`). Decision rows in `DECISIONS.md` use `ED-n`; at most 3 new rows per task.
