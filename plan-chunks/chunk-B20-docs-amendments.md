# Chunk B20: docs for standard 0.2.0 (amendments A-006 to A-009)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B19 (DONE). Decisions: ED-61 (new, OPEN). Next: B21 (you run the full suite and tag).

## Why approval comes first
CCIS rules: a CHANGELOG amendment takes effect only after you approve it, and the entry records the approval (CHANGELOG "Amendment rule" 3). Approving this chunk file by "do next" is the approval, as for A-004 and A-005. Entries above are never edited; ids are never reused.

## Proposed amendments (text is written in the build; this is the scope)
Next free requirement id is CCIS-REQ-095 (SPEC ends at 092; 093 and 094 are used in TRACEABILITY). Next free amendment number is A-006.
- **A-006 Concept keys and ownership** (REQ-095 to 098): a field may carry a `local` concept key; a concept has exactly one owner element, others reference it (`ownedBy`); a second owner is refused at publish (M-014); the local `terminology-package` holds keys, value sets and `replacedBy`. Rows B1 to B4, B15.
- **A-007 Entry form, contexts and composition** (REQ-099 to 101): a pathway may carry layout-only `entryForm` and `contexts`; composition is by reference and matches fields by concept key, never by label; a conflict report names owner, users, unit and blockers. Rows B5 to B8a.
- **A-008 Rule packs** (REQ-102 to 106): a `rule-pack` artefact of class `data-quality` only (ED-6); no number, threshold or parameter value in a rule; a prompt carries rule id, pack version, checksum and record ids, never values; parameters arrive disabled; reviewer is never the author. Rows B10 to B14a.
- **A-009 Operators and standard 0.2.0** (REQ-107, 108): executor operators `min max coalesce` (three-valued; operator set 0.2.0); the validator, compatibility check and importer accept standard 0.2.0; 0.1.0 artefacts still validate; strict concept and group rules start at minor 2 (B19, ED-60).

## What the build changes
- `CHANGELOG.md`: four new entries above A-005, same layout as A-004 (requirements, change, reason, raised by, approved). No older entry is touched.
- `ccis/SPEC.md`: append REQ-095 to 108 rows to the requirements matrix (section 13), status `Implemented`, with the test file named. Body text is not edited (D-214 stays).
- `docs/TRACEABILITY.md`: Appendix A rows for REQ-095 to 108, and one line in section 5 mapping the contract fixtures.
- `README.md`: one short section "Standard 0.2.0" (what is new, one command to run the contract tests, stamps stay 0.1.0). No other edit.
- Mirror row `ECO-B20` in the CCIS tracker.

## Open point for you (ED-61)
`docs/TRACEABILITY.md` calls the checksum amendment "A-005" (REQ-093, 094) while `CHANGELOG.md` uses A-005 for envelope administrations. Working choice: leave both as they are, number new ones A-006 on, and add one note line in the A-006 entry. Say if you want the old clash fixed instead (that edits older text, so it needs a separate row).

## Tests
Docs only. After the edit run `node --test test/ccis-contract-fixtures.test.js test/ccis-contract-fixtures-2.test.js test/ccis-standard-0-2-0.test.js` to prove the named tests exist. Check with `grep` that every new id appears once in SPEC and once in TRACEABILITY. No breakage checks.

## Size guard
4 doc files edited, no code. If an amendment needs a wording decision beyond the scope above, STOP and ask.

## Safety notes
No clinical content. Rules never advise or order (standing rule 4). Prototype banners stay.

## Context impact
Reads about 8 KB (tracker rows, CHANGELOG head, SPEC matrix tail, TRACEABILITY Appendix A tail, README headings).
