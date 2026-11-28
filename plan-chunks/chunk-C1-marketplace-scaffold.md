# Chunk C1: Marketplace scaffold

Prototype. Synthetic data only. Not for clinical use. Repo: Marketplace (https://github.com/avi33tbtt/Vibe-Rounds---Marketplace). Needs: C0 (DONE 2026-11-28: public, Pages on, root directory). Next: C2 (CI), C3 (publishing guide). Decisions: ED-3, ED-64 (new, working choice below).

## What exists
The repo holds only `LICENSE` (MIT). Do not recreate or edit it. CCIS `market/` holds the local test market in format `ccis-market/1`: `index.json`, `.gitattributes`, and `artefacts/<id>/<version>.json` for the snake demo (about 100 KB, built by `scripts/market-build.js`).

## What to build (ED-64, working choice)
Because Pages serves the repo root, the market lives at the root: `https://avi33tbtt.github.io/Vibe-Rounds---Marketplace/index.json` and `.../artefacts/<id>/<version>.json`. Files, all new:
1. `.gitattributes` with one line `* -text` (bytes are the checksum; no line-ending conversion).
2. `index.json` and `artefacts/` copied byte for byte from CCIS `market/` (the synthetic snake demo), so the index and the files agree from day one. C16 later replaces them with the built ecosystem content. Do not edit, reformat or re-sort any copied file.
3. `README.md`: banner lines (prototype, synthetic data only, not for clinical use; a checksum proves the same bytes, not who published them or that they are safe); what the repo is; the folder layout; the Pages address above; how a CCIS app is pointed at it (read CCIS README market section by `grep -n` only); "no patient data, ever"; links to the Studio and CCIS repos; status: CI comes in C2, publishing guide in C3.
4. `CONTRIBUTING.md` stub (under 30 lines): synthetic content only; artefacts are immutable once released (a change is a new version); every pull request needs CI green (C2) and a reviewer who is not the author (G5 will expand this); no clinical claim without a named clinician owner (ED-2).
Not in this row: workflow, scripts, tests, LICENSE changes, any new artefact.

## Files to read
CCIS `market/.gitattributes`; CCIS `README.md` market section (grep for "market"); the first 600 bytes of CCIS `market/index.json`. Nothing else.

## Files to change
Marketplace repo only: 3 small text files plus the copied `index.json` and `artefacts/`. Mirror rule: no CCIS or EMR change, so no mirror row. Studio tracker and `DECISIONS.md` updated.

## Check (no code, so no test file)
In the scratch clone: `diff -r` of the copied files against CCIS `market/` shows no difference (the `.gitattributes` differs by design); every `path` in `index.json` exists as a file; every file's sha-256 of canonical bytes is not recomputed here (C2 does that). After you push, open the Pages address for `index.json` once and tell me if it loads (it can take a few minutes).

## Size guard
4 new text files plus a copied folder. If the README needs more than 60 lines, STOP and split.

## Safety notes
Synthetic demo artefacts only. README says nothing is clinically reviewed. No patient data, no real names beyond the repo owner's MIT line already in LICENSE.

## Context impact
Reads about 4 KB; adds about 6 KB of new text and 100 KB of copied files.
