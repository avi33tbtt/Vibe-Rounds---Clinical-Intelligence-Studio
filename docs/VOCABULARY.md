# Vocabulary

Prototype. Synthetic data only. Not for clinical use.

Status: draft for review in A7. Decisions applied: ED-1 (trajectory has a use-time meaning), ED-5 (published CCIS artefacts are the source of truth), ED-9 (concept keys are `local` bindings), ED-12 (one owner per concept). Unreviewed by a clinician (ED-2 has no named owner).

## Terms

| Term | Meaning | Time | Notes |
|------|---------|------|-------|
| Element | A versioned, reusable definition of one thing recorded about a patient. Declares fields with type, unit, cardinality, time meaning and allowed epistemic states. ID form `group.name`, for example `dm.core`. | Design | Contains no numbers, bands or thresholds. |
| Element group | A published artefact that lists elements as members (mandatory or optional) and names the owner element of each shared concept. Required by CCIS-REQ-023 and 024. | Design | Composition is by reference, never by copy. |
| Pathway | A designed, ordered set of elements. The EMR entry form is a pathway. | Design | Describes information needed. It never authorises or automates treatment. |
| Pathway package | The publishable unit: a CCIS pathway artefact with `elements`, `entryForm`, `contexts`, `stages` and `nodes`, `analytics` references and an optional `rules` reference. | Design | `entryForm`, `contexts` and `rules` arrive with standard 0.2.0. |
| Trajectory | What accumulates for one patient when elements and pathways are applied in whatever order care takes. Keyed by concept, not by the form a value was typed into. | Use | Never a design artefact. Missing, unknown and entered-in-error are never plotted as zero. |
| Concept key | The shared meaning of a field, for example creatinine. A binding in the `local` code system. | Design | Matched by key, never by similar label. Exactly one owner element per concept. |
| Concept owner | The one element that defines a concept. Any other element using the concept references the owner. | Design | A second owner is refused at publish. |
| Context | A condition tag (for example diabetes, CKD) that says which elements belong to a condition. Declared by the pathway in `contexts`. | Design | Replaces the hard-coded EMR `context-map.js`, which stays as a fallback. |
| Entry form | The layout part of a pathway: sections, order, repeating groups, default visibility. Items are of kind `element` or `section`. | Design | Layout only, no clinical logic. A `section` item is a placeholder for an existing EMR area (problems, allergies and so on), not an element. |
| Rule pack | A governed artefact of rules written in the CCIS three-valued whitelist expression language. Parameters are disabled by default. | Design, runs in use | Version 1 allows the `data-quality` class only (ED-6). Output is a visible prompt with rule id, version and records read. Never an order. |
| Analytics package | A CCIS artefact that computes results from a context envelope. Binds to elements, so it works on any pathway containing them. | Design, runs in use | Thresholds and intervals are disabled parameters. |
| Marketplace | A static, checksummed, immutable repository of published artefacts with an index (`ccis-market/1`). Hosted as a public GitHub repo with Pages (ED-3): https://github.com/avi33tbtt/Vibe-Rounds---Marketplace. | Distribution | No patient data. Checksums use `ccis-canonical-json/1`. |
| Studio | The authoring app for elements, packages, analytics references and rule packs. | Design | Static, serverless, nothing stored in the browser (ED-4). |
| Context envelope, run record | Envelope: patient data exported from the EMR to CCIS. Run record: CCIS results imported back into the EMR. | Use | The only path patient data takes. Never in the Marketplace. |

## Two meanings of "pathway" today

In the EMR a pathway is a list of element set versions. In CCIS a pathway has stages, decision nodes and analytics references. The package above joins them: one CCIS artefact is the source of truth (ED-5) and the EMR materialises its own records from it.

## EMR terms to CCIS terms

The EMR side is taken from the plan page (`docs/ECOSYSTEM-PLAN.html`). Row 3 is inferred and must be checked against EMR source in A4. This doc did not read the EMR repo.

| EMR term | What it is (per plan page) | CCIS term | Difference |
|----------|---------------------------|-----------|------------|
| Element set | A reviewed, versioned form definition, immutable per version. Fields keyed by (set, field key). | Element | The EMR set may bundle several concepts. CCIS splits by meaning (for example `ckd_ctx` into `ckd.core` and `ckd.kidney-function`). No concept key on the EMR side yet (migration D3). |
| Pathway version | An ordered list of exact element set versions. Applying one pins each set to the case. | Pathway package (the `elements` and `entryForm` parts) | EMR has no layout, no contexts, no stages or nodes. Imported versions will carry source artefact id, version and checksum (D4). |
| `case_applied_sets` | Not described in the plan page. Assumed to record which set versions are pinned to a case. | Nothing in the standard. Closest idea: the applied pathway in a context envelope. | Unverified. A4 reads the EMR source. |
| Observation | Append-only value keyed by (set, field key). | A value of a field in an envelope, grouped in a trajectory by concept key | Without concept keys two sets holding creatinine give two unrelated series. |
| Context map (`context-map.js`) | Hard-coded list of set slugs per condition. | `contexts` on the pathway | To be replaced (D15). |
| Staged import and review | Marketplace packages arrive as submitted drafts and use the existing review rules. | Published artefact, immutable | Review stays local to the EMR. |
