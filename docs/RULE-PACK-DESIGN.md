# Rule pack design (row A6)

Prototype. Synthetic data only. Not for clinical use.

Status: design for review in A7. Nothing here is built. B10 to B14 turn it into a schema, validator rules and an evaluator. Where this page proposes a number or a name, it says "proposal" and B10 to B13 may change it by adding a row.

Read for this row: the tracker, `plan-chunks/01-core.md`, `DECISIONS.md`, the plan page (rule section and risk table), and rule mentions in `docs/PACKAGE-DESIGN.md` and `docs/VOCABULARY.md`. Not read: the CCIS executor and its operator list, the CCIS validator, the EMR envelope builder. So the exact names of the 18 existing operators and the exact epistemic state names are not copied here. B11 must take them from the executor and the element schema.

## 1. Scope for version 1 (ED-6)

- Version 1 allows one class: `data-quality`.
- The schema may name a second class, `clinical-prompt`, so the format does not change later. In version 1 the Studio will not publish it, the Marketplace CI refuses it, and the EMR import (D19, D20) stages it as blocked. It is unlocked only when ED-2 names a clinician owner and you decide ED-6 again.
- ED-2 names no clinician. So every rule below is demo content, and every interval is a disabled parameter.

## 2. What counts as data-quality

A rule is data-quality when its condition depends only on one of these:

| Kind | The condition asks | Example question (illustrative) |
|------|--------------------|----------------------------------|
| `missing` | Is there any record for this concept? | Is there no creatinine record for the case? |
| `stale` | Is the newest record older than an interval? | Is the newest record older than the interval parameter? |
| `conflicting` | Do two records for the same concept at the same time disagree? | Two measured systolic pressures with the same time and different values. |
| `inconsistent` | Do two fields disagree with each other, by type, unit or order? | Diastolic pressure not lower than systolic pressure. |
| `interval-documentation` | Is a recording that the pathway expects absent within a window? | No glycaemic measurement within the window parameter. |

A rule is NOT data-quality when its condition depends on whether a value is high, low, rising, or inside or outside a band. That is a clinical threshold. It belongs to the `clinical-prompt` class and is blocked.

Two consequences:

1. Bounds on a single value ("plausible range") are not allowed in version 1, because a bound reads as a clinical cut-off. Only comparisons between two fields (`inconsistent`) are allowed. This is a new decision, ED-18.
2. `stale` and `interval-documentation` use a time interval. The interval is a clinical judgement (how often should this be measured), so it is a parameter, disabled by default (section 5).

The validator cannot judge meaning. It enforces the form of the test above (section 8, layer 6). The reviewer judges meaning (section 9).

## 3. Pack shape (proposal for B10 and B11)

A rule pack is a CCIS artefact of type `rule-pack` with:

- `manifest`: id (`group.name`, for example `ckd.data-quality`), version, class, title, description, standard version, author, reviewer record, owner, status, source note, `derivedFrom`.
- `scope`: the pathway ids (and exact versions or version ranges as the pathway reference rules say) the pack may attach to. A pack never runs outside its scope.
- `inputs`: each input is a name, a concept key, an optional element id, and a read mode: `latest`, `all`, `count`. The concept key is the meaning; the element id only narrows. Inputs never match by label (standing rule 5).
- `parameters`: each has a name, a kind (`interval`, `count`), a unit, and `enabled: false`. No value is stored in the published file (section 5).
- `rules`: each has an id, a kind (section 2), an expression tree, a message template, and a severity of `info` or `attention` (proposal; no `critical`, so a rule cannot present itself as urgent).
- `tests`: fixtures with hand-calculated expected outputs (section 10).

A pathway refers to at most one rule pack in `rules` (PACKAGE-DESIGN, B5). A pack may cover several elements of the composed pathway, because it reads by concept.

## 4. Expression language and evaluator

- The language is the existing CCIS three-valued whitelist expression language, plus the five operators from B9 (`first`, `last`, `min`, `max`, `coalesce`). Rules may use only operators on that list. No other operator, no function call, no loop, no string building.
- Three-valued means: true, false, unknown. Missing, unknown and entered-in-error are never turned into zero or false (standing rule 6). A rule whose answer is unknown does not fire and is reported as `not-evaluated` in the test output. Unknown never fires a prompt, and never silences one by being read as false.
- No code, no network, no clock. Time comes from one input, `asOf`, which the caller puts in the context (the EMR envelope builder, D17). The evaluator never reads the system time. The same context always gives the same result.
- Limits (proposal, B11 to confirm): at most 20 rules per pack, depth at most 8 and at most 40 nodes per expression, message at most 200 characters.
- A rule that depends on a disabled parameter is `not-evaluated`. It is never false and it never fires.
- `ccis-rules.js` (B14) is a pure function: pack plus canonical context in, a list of fired prompts and a list of not-evaluated rule ids out. Same input, same bytes (canonical JSON, `ccis-canonical-json/1`).

## 5. Parameters

- Every parameter is published disabled and without a value. Rule 3 in the core conventions: no numbers or thresholds in definitions.
- Enabling is local. A clinician at the EMR site supplies the value and enables it; the EMR records who, when, the pack id and version, the parameter name and the value (D23). That row stays locked until ED-6 and ED-2 allow it.
- Until then a rule that needs the parameter is `not-evaluated`. So version 1 can ship and run with only the rules that need no parameter (`missing`, `conflicting`, `inconsistent`), and the interval rules wait for local enabling.
- The Studio rule builder (F23) never offers a default value.

## 6. Output: the prompt

A fired rule produces one prompt:

| Field | Content |
|-------|---------|
| `ruleId` | pack id plus rule id |
| `packVersion` and `packChecksum` | exact version and checksum that ran |
| `kind` and `severity` | from the rule |
| `message` | the template, rendered |
| `recordRefs` | identifiers of the records the rule read, never their values |
| `asOf` | the context time used |

Message rules (checked by B12):

- Plain text. Not more than 200 characters. No markup, no URLs, no control characters. Text from a package is untrusted (standing rule 9): the EMR escapes it on display and follows nothing in it.
- A template may hold only fixed text plus a small closed set of placeholders: the concept display name, a parameter name, and a record count. No clinical values, no dates, no patient identifiers (proposal, ED-19).
- A message describes an information gap. It never tells anyone to do something to a patient. Validator lint: an error if a message contains an instruction verb from a closed list (for example order, start, stop, give, prescribe, increase, reduce, discontinue). The list is a crude guard; the reviewer is the real control.
- Plain wording example (illustrative): "No record found for creatinine. Check whether it was measured and not yet entered."

A prompt never orders, never changes data, never opens a form on its own. The user can dismiss it with a reason (D22). Nothing else happens.

## 7. Provenance and governance fields

Each pack carries, in the manifest:

- `author`: a person or role name. A named person is required to publish.
- `reviewer`: a record of who reviewed which version, and when. Must differ from the author (section 9).
- `owner`: for `data-quality`, may be `unassigned` while ED-2 is open. For `clinical-prompt`, a named clinician is required and the pack cannot claim `validated` without a validation record (B13).
- `sourceNote`: one plain sentence on why the rule exists. Not a clinical guideline citation unless a clinician owner has added it.
- `derivedFrom`: the plan, SPEC or tracker rows the rule comes from.
- `status`: `draft`, `reviewed`, `published`. A published version is immutable (standing rule 7).

## 8. Validator layers for a rule pack (B12, B13)

| Layer | Row | Check |
|-------|-----|-------|
| 1 to 2 | B10, B11 | Schema shape, manifest and artefact type. (Layer names come from the CCIS validator, not read here.) |
| 3 | B12 | References: every input concept key and element id exists, every parameter named in a rule exists, scope pathways exist. |
| 4 | B12 | Operator types: only whitelist operators, argument types fit, depth and node limits. |
| 5 | B12 | Parameters default disabled and carry no value; messages plain text, closed placeholders, no instruction verbs. |
| 6 | B13 | Executable safety: class `data-quality` allows no comparison of a single value to a constant or parameter; only presence, state, time difference against an interval parameter, and field-to-field comparison. |
| 7 | B13 | Test verification: every rule has at least the cases in section 10, and `ccis-rules.js` reproduces the expected outputs. |
| 8 | B13 | Governance: class, owner, author and reviewer fields; `clinical-prompt` needs a named owner and no unsupported `validated` claim; author differs from reviewer. |

## 9. Review and independence

- The author of a pack is never its reviewer (standing rule 8). The reviewer record names a different person.
- Where it is checked: the Studio publish step refuses an equal author and reviewer (F27); Marketplace CI checks the record in the pull request (C2); the EMR creates a draft on import and the person who accepts is not the one who publishes (D7, existing rule); the release checklist repeats it (G5).
- Limit to state plainly: with one maintainer, these checks compare names in a file. They show intent and catch mistakes. They do not prove independence. Real independence needs a second person (ED-19).
- A reviewer checks two things the validator cannot: that the condition is really about data quality and not a clinical judgement, and that the message wording is an information gap and not advice.

## 10. Tests

Per rule, at least three synthetic fixtures with hand-calculated results:

1. The condition holds: the rule fires, with the expected `recordRefs`.
2. The condition does not hold: no prompt.
3. An input is unknown or missing where that matters: `not-evaluated`, no prompt, and no zero or false substituted.

Rules with a parameter add a fourth: parameter disabled gives `not-evaluated`. Fixtures say they are synthetic. A test file has at most 8 cases (budget rule), so large packs split across rows.

## 11. EMR side (for D19 to D23)

- Import: a pack arrives as a staged draft and is pinned to a pathway version through review (D19). Append-only.
- Run: the runner builds the context with the existing envelope builder, evaluates the pinned packs with the vendored `ccis-rules`, and only for class `data-quality` (D20). Every run is audited.
- Log: `rule_fires`, append-only: pack id, pack version, checksum, rule id, case, record refs, time. No values (D21). A dismissal and its reason are logged. Rules that did not fire are not logged.
- Display: a panel with rule id, version and the records read (D22).
- When a rule fires on records that are later corrected, the old log row stays. A new run may produce a new row. Nothing is rewritten.

## 12. Prohibited

- Any clinical threshold, band, target or range in a rule, a parameter default, a message or a test name (in version 1).
- Advice, orders, doses, or any message that tells someone to act on a patient.
- Silent change of any record. A rule has no write access.
- Code, network, clock, randomness, or a regular expression in a rule.
- Matching by label or by similar text. Inputs are concept keys.
- A value or a patient identifier in a prompt, a log row or a message.
- A published pack that was edited after release. A change is a new version.
- A `clinical-prompt` pack in any version-1 distribution.
- A pack that attaches itself to a pathway outside its `scope`.
- Claims of validation, accuracy or safety without a validation record.

## 13. Illustrative rules (synthetic, not final schema)

These show the form. Concept names are placeholders; the real keys are in `docs/ELEMENT-LIST-V0.md` part 4.

| Rule | Kind | Needs a parameter | Fires when |
|------|------|-------------------|------------|
| `dq-missing-creatinine` | `missing` | no | `count` of records for the creatinine concept is zero. |
| `dq-conflicting-sbp` | `conflicting` | no | two measured records for the systolic concept share a time and differ. |
| `dq-bp-order` | `inconsistent` | no | latest diastolic is not lower than latest systolic at the same time. |
| `dq-stale-creatinine` | `stale` | yes, interval | newest creatinine record is older than the interval, measured against `asOf`. |
| `dq-glycaemic-window` | `interval-documentation` | yes, window | no glycaemic record inside the window ending at `asOf`. |

The two parameterised rules are `not-evaluated` until a clinician enables the interval locally.

## 14. What this design does not settle

- The exact schema property names (B10, B11).
- The exact operator names and the layer names (taken from the CCIS executor and validator).
- How often the runner executes (on save, on open, on demand). D20 decides, and its chunk file says so.
- Clinical prompts. Blocked until ED-2 and a new decision on ED-6.
- Signing of packs. G4.

New decisions from this row: ED-18 (no single-value bounds in version 1) and ED-19 (message placeholders; independence with one maintainer). Both are open until A7.
