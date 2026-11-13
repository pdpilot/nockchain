# AGENTS.md — security review mode

Read this before any security task in this repository. It adds a *process*; it does not replace your
own method. The documents it asks you to keep are the only progress record — the chat is not.

Pinned target: **`ed1edfc`** (2026-10-26). State the commit in every verdict; re-verify if the
branch has moved.

## 0. The task contract (as published by the programme)

- **Only CRITICAL findings are reviewed.** Critical is: theft or unauthorised movement/destruction of
  funds; unbacked issuance (NOCK or wrapped NOCK minted outside consensus rules); an extreme
  proof-of-work bypass (valid blocks at ≤10% of honest cost, or credit for work not performed,
  without majority hashrate or extreme luck); or arbitrary proof forgery accepted on the **production
  verification path**.
- **Out of scope:** majority-hashrate attacks; third-party dependencies (report upstream);
  previously known issues and findings from published audits; social engineering; **automated
  scanner output without a developed, reproducible exploit**.
- One finding per report, one root cause each, with a PoC that runs on the production path against
  a clean checkout. Unreproducible reports are not eligible.

A seam that can only reach "a real bug, not critical" is recorded and parked, not written up.

## 1. Read order, before analysis

There is no separate spec spine in this tree (it was removed in `#225`); the entry points are:

1. `README.md` — layout, build, and how node/miner/wallet fit together.
2. `docs/protocol/README.md` and everything in `docs/protocol/upgrades/` — consensus rules and every
   rule change by height. **Read all of them** before reasoning about acceptance.
3. `docs/architecture/README.md` (+ `docs/architecture/tx-engine/`) and `docs/AI_POW_CUTOVER.md`.
4. `docs/fixes/*` and `docs/pma/*` — known-hard areas and previous findings.
5. The `BUILD.bazel`/`README` of every crate you touch; `hoon/apps/dumbnet/inner.hoon` is the
   consensus entry point, `hoon/apps/dumbnet/lib/consensus.hoon` its rule library.

Do not write a finding whose specification you only inferred from code. A missing spec is itself a
line in `SPEC.md`.

## 2. Prior art, before any new seam

- **`#221` "Security batch 3: coordinated consensus upgrade at height 170,000"**, **`#222` "honk:
  /dat kernel parity"**, **`#225` layout reorg**. Read all three diffs first: the first is a security
  fix in the exact area this programme targets, the second pins a jet/kernel parity claim.
- `git log --all --grep='security\|advisory\|fix\|CVE\|exploit'`, `docs/fixes/*`, `docs/pma/*`, and
  the upgrade notes in `docs/protocol/upgrades/`.
- The test suite: a behaviour pinned by a test is covered. Name the test, then move on.
- Your own earlier reviews are **not** in this repo. Keep them in the private archive
  (`SECURITY-REVIEW/PRIOR.md`, git-excluded) so nothing is reported twice.

## 3. The two documents you maintain (git-excluded, never pushed)

Create `SECURITY-REVIEW/` if it does not exist (in a fresh checkout it will not). It is working
paper: never `git add` it, never commit it, never open a PR with it.

- **`SECURITY-REVIEW/SPEC.md`** — components, the invariant each claims, the seams where they
  disagree, and the attack angle per seam.
- **`SECURITY-REVIEW/LEDGER.md`** — one append-only row per seam, with status and commit.

Statuses: `TODO` · `READING` · `HYPOTHESIS` · `TESTING` · `REFUTED` · `FINDING` ·
`PARKED(non-critical)` · `KNOWN(disclosed/pinned)`.

## 4. Per seam: the loop

1. Quote the invariant the code claims, with its location (doc, comment, or test).
2. State the attack angle and the *observable* difference it would make.
3. Check prior art (§2). Pinned by a test or already disclosed → `KNOWN`, one line why, stop.
4. Build the smallest **runnable** PoC on the production path. It must show the effect, not exercise
   a mock: red before a fix, or red against the deployed rule.
5. **Quantify.** For work claims: cost per valid block versus honest, or work credited without being
   performed. An unquantified PoW claim drops to a lower band by the programme's own rules.
6. Record verdict, artifact path, commit, and the one-line reason in the ledger.

## 5. Rules that are not negotiable

- **No finding without a runnable PoC.** A plausible seam is a ledger row, not a report.
- **One root cause per report.** Split by root, never by symptom; a second report of one root earns a
  credit share at most.
- **Never run anything against mainnet.** Local or private networks only.
- **Never push review material.** `SECURITY-REVIEW/` (spec, ledger, findings, PoCs) stays out of git;
  it is excluded via `.git/info/exclude`. Findings go to the maintainers through a **private**
  advisory, by the human — not by you, not through a branch or PR.
- **Never edit code to make a PoC pass.** Reproduce on a clean checkout of the pinned commit.
- **Third-party code is out of scope** (`third_party/`, vendored crates). Note it, move on.
- **Time-box.** No verdict after roughly an hour → `PARKED` with what blocked you (GPU, keys,
  unbuildable target, a service that does not exist). Record it; never drop it silently.
- **Check the tree before judging:** `git status` must be clean, and the commit must equal the one you
  wrote down. A finding that needs local edits is not a finding.

## 6. Where the critical classes live in this layout

- **Consensus / kernel:** `hoon/apps/dumbnet/inner.hoon`, `hoon/apps/dumbnet/lib/consensus.hoon`,
  `crates/lib/kernels`, `crates/lib/nockapp`, `crates/lib/nockchain-types`.
- **AI proof-of-work (programme focus):** `crates/lib/ai-pow`, `crates/lib/ai-pow-jets`,
  `crates/lib/ai-pow-zk`, `crates/apps/ai-pow-miner`, `crates/apps/ai-pow-miner-cuda`,
  `crates/apps/zk-pow-miner`, plus the verify path in `inner.hoon`.
- **Jets versus the interpreter:** `crates/tools/honk`, `crates/tools/honk-tools`,
  `crates/lib/ai-pow-jets` — a jet that disagrees with Nock semantics is a consensus split.
- **Funds / issuance:** `crates/bridge`, `crates/lib/wallet-tx-builder`, `crates/apps/nockchain-wallet`.
- **Node surface:** `crates/lib/nockapp-grpc*`, `crates/apps/nockchain-api`, `crates/tools/chaff`.

Starting points, not a scope limit.

## 7. Handoff

A `FINDING` is written in the programme's structure — Summary / Affected component with commit and
version / Impact with the quantified advantage / Proof of concept with environment and commands /
Supporting material — under `SECURITY-REVIEW/findings/<ID>/`, with the PoC and its output beside it.
Report to the human; do not submit anything yourself.
