# Chunk C2: Marketplace CI

Prototype. Synthetic data only. Not for clinical use. Repo: Marketplace. Needs: C1 (DONE). Next: C3 (publishing guide). Decisions: ED-65 (new, working choice below).

## What exists
The Marketplace root holds `index.json`, `artefacts/`, README, CONTRIBUTING (C1). No code, no workflow, no `package.json`. CCIS (pinned commit `4644d17d0d03ffb6a2f4de8b676f6ef62692c452`, the head when this chunk was written) exports `loadMarket` (every file against its index entry: checksum, canonical bytes, id, type, version, provides and requires; stray files refused M-011), `validateArtefact` (eight layers, `{ known, fixtures }`) and `buildValidationRecord`. The CCIS validator has no runtime dependency (`ajv` is dev only), so no `npm ci` is needed.

## What to build (ED-65, working choice)
1. `scripts/check-market.js` (under 120 lines, Node 22, no dependency): arguments `--ccis <dir>` (the CCIS checkout) and optional `--base <git-ref>`. It imports `ccis-market.js` and `ccis-validator.js` from `<dir>` by file URL. Steps:
   a. Read `index.json` and every file under `artefacts/` (anything not `*.json` is refused).
   b. `loadMarket`: recomputes every checksum, requires canonical bytes, refuses a missing or stray file.
   c. For each artefact, `validateArtefact(doc, { known: all other artefacts })`: no error row in any of the eight layers, and the index entry's `validation.outcome` is `passed`.
   d. With `--base`: read the base index and files with `git show <ref>:<path>`, then call the pure `checkImmutable(base, head)`. Every released file keeps identical bytes; no released file is deleted or renamed; every base index entry equals the head entry. Adds are allowed.
   Exit 1 with one line per finding (rule id, path); never print artefact text.
   `checkImmutable({ indexText, files }, { indexText, files })` is exported and pure so a test needs no git.
2. `.github/workflows/ci.yml`: on `push` and `pull_request`; `permissions: contents: read`; `actions/checkout@v4` with `fetch-depth: 0`; a second checkout of `avi33tbtt/Vibe-Rounds---CCIS` at the pinned commit into `_ccis`; `actions/setup-node@v4` Node 22; run `node --test test/`, then `node scripts/check-market.js --ccis _ccis` (on pull requests add `--base origin/${{ github.base_ref }}`). The pin lives in one `env: CCIS_SHA` line; moving it is a deliberate edit.
3. `test/check-market.test.js` (at most 8 cases).
Not in this row: signing, publishing guide (C3), any artefact change.

## Files to read
CCIS `ccis-market.js` lines 297 to 335 (`loadMarket`) and the header comment; CCIS `ccis-validator.js` export list (grep `export`); CCIS `.github/workflows/ci.yml` (already read: style to copy). Nothing else.

## Tests
1. `checkImmutable`: only additions pass.
2. A changed released file is refused and names its path.
3. A deleted released file is refused.
4. A changed index entry for a released artefact is refused.
5. The repo's own market passes the full check (uses env `CCIS_DIR`; skipped with a printed note when unset; set in CI).
6. A copy of the market with one changed byte is refused by the check (M-010).
7. A stray file under `artefacts/` is refused (M-011).
Run in CI and, if `CCIS_DIR` is set, locally. At most 3 breakage checks (relax the byte compare, skip the stray check, drop the outcome check).

## Size guard
1 workflow, 1 script, 1 test. If it exceeds 120 script lines or needs a dependency, STOP and split (for example the immutability part as C2a).

## Safety notes
The script reads pull-request content but never runs it, never prints artefact text, never uses the network. The workflow has read-only permissions and no secrets. A pass proves bytes, checksums and validation only; it does not prove who published or that content is clinically safe.

## Context impact
Reads about 5 KB; adds about 9 KB of script and workflow and 90 test lines.
