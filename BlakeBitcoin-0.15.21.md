# BlakeBitcoin 0.15.2 Update — Source of Truth

## Overview

Port BlakeBitcoin (BLC) from its current 0.8.9.8 codebase to Bitcoin Core 0.15.2, following the same approach used for the Blakecoin 0.15.2 update (`Blakecoin-0.15.21`).

**Reference codebase:** `../Blakecoin-0.15.21/` — the completed Blakecoin port to 0.15.2
**Original codebase:** `../BlakeBitcoin/` — current 0.8.9.8 source with all coin-specific parameters

---

## QC Status

- QC is now strong enough to treat this repo as the shared 0.15.2 AuxPow reference port, including the modern merged-mining RPC direction. Historical merged-mined header compatibility is now documented well enough for rollout planning; the production pool and Electrium carry-back staging that used to remain is now green.
- The current 0.15.2 tree builds and passes `test/functional/wallet-basic-smoke.py` on regtest with no sends, no funding, and no mining.
- Verified from `../BlakeBitcoin/src`: main/test P2P and RPC ports, `pchMessageStart`, address prefixes, message header, and the standard 210,000-block halving reward model.
- Corrected from source: secret-key prefixes are `128` mainnet and `239` testnet, not `TBD`.
- Regtest stays local-only in this 0.15.2 bootstrap and has AuxPow activation wired at height `0` for local validation.
- The shared AuxPow framework is now integrated: `src/auxpow.{h,cpp}`, `src/primitives/pureheader.{h,cpp}`, AuxPow-aware block/header serialization, disk index persistence, AuxPow-aware PoW validation, and chain-ID-aware block template versions.
- Mainnet keeps strict chain-ID enforcement with chain ID `0x0005` and AuxPow start height `500000`; testnet and regtest use chain ID `0x0005` with start height `0` and strict chain ID disabled for local QA.
- Verified after the compatibility follow-up: `make -C src -j4 blakecoind blakecoin-cli` succeeds and `test/functional/test_runner.py --jobs=1 wallet-basic-smoke.py` passes on regtest.
- Verified historical bootstrap compatibility fix: BlakeBitcoin's legacy chain includes AuxPow-bearing headers before the nominal mainnet AuxPow activation height, so the strict 0.15.2-era `early-auxpow-block` reject had to be removed to preserve historical chain acceptance.
- Verified historical BIP30 compatibility fix: the legacy BlakeBitcoin chain reaches a duplicate-txid overwrite at mainnet height `256205`, so Bitcoin Core's broad 0.15.2-era BIP30 enforcement had to be relaxed back to BlakeBitcoin's original 0.8 behavior or IBD stops with `ConnectBlock(): tried to overwrite transaction`.
- Merged-mining RPC direction is now fixed for this port: primary RPCs are `createauxblock <address>` plus `submitauxblock <hash> <auxpow>`, with `getauxblock` kept only as a compatibility wrapper for older pool software. `getworkaux` is intentionally out of scope unless a real dependency is later proven.
- Private testnet QA correction: the inherited 0.8.x BlakeBitcoin testnet genesis tuple (`nTime=1392351202`, `nNonce=4335147`, hash `00000052...0d5b`) does not satisfy PoW in the 0.15.2 code path, so the isolated AuxPoW QA testnet now uses the same timestamp/message/start bytes with a corrected valid nonce/hash pair (`nNonce=32975318`, hash `0000007381461a5b95ec93210ec6fcf6d3328e7f34113da11a4657d06caef7ad`).
- Keep this repo on a strict no-send / no-mine mainnet rule until the final production pool and Electrium carry-back staging are complete.

### BlakeStream Seed And AuxPoW RPC Policy

- BlakeStream DNS seeds (`seed.blakestream.io`, `seed.blakecoin.org`) are shared across all six coins and serve nodes for ALL coins. A single seed lookup returns peer IPs regardless of which coin is asking; coin separation happens at the wire-protocol layer via each coin's unique `pchMessageStart` and default port.
- The production direction for this repo is a modern merged-mining RPC surface: `createauxblock` to build the child-chain template and `submitauxblock` to submit the solved AuxPoW payload.
- `createauxblock` is address-driven on purpose so a pool can choose the child-chain payout script explicitly instead of depending on wallet mining state inside the daemon.
- `getauxblock` remains only as a compatibility mode for older merged-mining software. It should map onto the same block-template / block-submit flow rather than preserving a separate legacy implementation path.
- `getworkaux` is not part of the planned 0.15.2 target. We are not reviving `getwork`-era RPC unless a live pool or deployment proves it is still required.
- The same 2 DNS seeds (`seed.blakestream.io`, `seed.blakecoin.org`) are used by all six coins. This matches the Blakecoin 0.15.2 reference repo exactly.

### Wire Checksum Policy

- BlakeBitcoin should preserve the legacy `Hashblake` P2P message checksum behavior for current network interoperability.
- Do not normalize BlakeBitcoin to Blakecoin's temporary non-`Hashblake` handshake exception.
- Keep Blakecoin documented as the one current exception; BlakeBitcoin stays on `Hashblake` before go-live unless a fresh compatibility review says otherwise.

## AuxPoW Start And Completed Work

| Network | Chain ID | Nominal AuxPoW Start | Observed Pre-Start AuxPoW Evidence In Current QA | Exact Time/Date Status | 0.15.2 Port Rule |
|---------|----------|----------------------|-----------------------------------------------|------------------------|------------------|
| Mainnet | `0x0005` | `500000` | Seen by block `1` during isolated bootstrap replay | Exact timestamp remains archival-only; compatibility rule is already proven by replay | Keep `500000` as the nominal legacy value, but tolerate earlier historical AuxPoW-bearing blocks during bootstrap / IBD |
| Testnet | `0x0005` | `0` | N/A | Local QA only | AuxPoW enabled for local QA; strict chain ID disabled |
| Regtest | `0x0005` | `0` | N/A | Local QA only | AuxPoW enabled for local QA; strict chain ID disabled |

Interpretation note:
`500000` is the nominal legacy mainnet activation height preserved in `chainparams.cpp`. It is not being treated as proof that earlier historical AuxPoW-bearing blocks cannot exist. The fixed 0.15.2 rule keeps the nominal boundary for documentation and chain-ID policy, while avoiding the blanket `early-auxpow-block` reject that broke real historical bootstrap replay.

- Completed in this repo:
  - Integrated the shared AuxPoW framework with `src/auxpow.{h,cpp}` and `src/primitives/pureheader.{h,cpp}`.
  - Ported AuxPoW-aware block/header serialization, disk index persistence, block version handling, and PoW validation.
  - Kept BlakeBitcoin's nominal mainnet start height at `500000` while removing the modern `early-auxpow-block` reject that broke historical chain acceptance.
  - Implemented the modern merged-mining RPC direction: `createauxblock <address>` and `submitauxblock <hash> <auxpow>`, with `getauxblock` retained only as a compatibility wrapper.
  - Verified no-send regtest wallet smoke coverage and verified isolated bootstrap replay moving past the old pre-start rejection point.
  - Corrected the private testnet genesis nonce/hash so an isolated 0.15.2 BlakeBitcoin testnet can actually boot for AuxPoW QA.
- Operational rule:
  - Keep the strict no-send / no-mine mainnet rule in place while final production pool and Electrium carry-back staging continue.

---

## Coin Identity

| Parameter | Value |
|-----------|-------|
| Coin Name | BlakeBitcoin |
| Ticker | BLC |
| Algorithm | Blake-256 (8 rounds) |
| Merge Mining | Yes (AuxPow — merged with Blakecoin and Photon; shared 0.15.2 port now present) |
| Base Version (current) | 0.8.9.8 (forked from Bitcoin 0.8.6) |
| Target Version | 0.15.2 |

---

## Chain Parameters to Preserve

### Network

| Parameter | Mainnet | Testnet | Regtest |
|-----------|---------|---------|---------|
| P2P Port | 8356 | 18112 | 18444 (local-only) |
| RPC Port | 8243 | 1812 | 18443 (local-only) |
| pchMessageStart | 0xf9, 0xbc, 0xa7, 0xb7 | 0x0b, 0x11, 0x12, 0x09 | 0xfa, 0xbf, 0xb5, 0xda |

### Address Prefixes

| Type | Mainnet | Testnet | Regtest |
|------|---------|---------|---------|
| Pubkey Address | 243 (0xF3) | 142 (0x8E) | 243 (local-only) |
| Script Address | 7 (0x07) | 170 (0xAA) | 7 (local-only) |
| Secret Key | 128 (0x80) | 239 (0xEF) | 128 (local-only) |
| Bech32 HRP | `blb` | `tblb` | `rblb` |

### Block Parameters

| Parameter | Value |
|-----------|-------|
| Block Time | 150 seconds (2.5 minutes) |
| Target Timespan | 1,209,600 seconds (14 days) |
| Retarget Interval | 8,064 blocks |
| Coinbase Maturity | 100 blocks |
| Max Supply | 21,000,000 BLC |
| COIN | 100,000,000 satoshis |
| Initial Block Reward | 50 BLC |
| Halving Interval | 210,000 blocks |
| PoW Limit nBits | 0x1e00ffff |
| PoW Limit | `0x000000ffffffffffffffffffffffffffffffffffffffffffffffffffffffffff` |

### Genesis Block

| Parameter | Value |
|-----------|-------|
| Hash | `0x000000dcb4434e2148558a0a5c71e5c06d864accef97d75ac1c031405deb3371` |
| Merkle Root | `0x0423141660220f9f155a4129a49dcb6431bbed9cd037bba3da34c2baa53ed0ac` |
| nTime | 1399109785 (May 3, 2014) |
| nNonce | 183,893,667 |
| nVersion | 112 |
| nBits | 0x1e00ffff |
| Coinbase Message | "Added to the Blake-256 Merge Mining 12th May 2014" |

### Testnet Genesis Block

| Parameter | Value |
|-----------|-------|
| Hash | `0x0000007381461a5b95ec93210ec6fcf6d3328e7f34113da11a4657d06caef7ad` |
| nTime | 1392351202 |
| nNonce | 32,975,318 |

### DNS Seeds

- blakecrypto.com (single seed; still needs review once AuxPow lands)

### Checkpoints

| Block | Hash |
|-------|------|
| 0 | `0x000000dcb4434e2148558a0a5c71e5c06d864accef97d75ac1c031405deb3371` |
| 145025 | `0x0316c10a202c2bde44628c8cac2d75d61f078a1d961ae1499eaa98eb643b5068` |
| 179266 | `0x7b102e1f37971dcd4311cc64f83fc62da0f75c22270e831be0a6c8cc38ddd5c8` |
| 338643 | `0xdd79a4b1ac2a91d9666d97a2654ee826c84e55495665bacaac2b9a953616f8d6` |
| 406644 | `0xc0f29fe22936216e6a90a4178967ba8ffa9ad78930aa1a369a6fc727a3d2f8e5` |
| 845000 | `0xa32d61133e22687a63c0c2769552a851a484b030cda02f8a1def5a506d368e33` |
| 900000 | `0x1054253e8fd7b596cdddd562619da01022024623ee72ed3b37ea909c3caa5cc7` |
| 950000 | `0xdd4b0d5c8ae8dca9b0fd8ebee3fdf1312c47cdf73f18bff2379d5e4e2d1e59c8` |
| 966300 | `0xb4d0943e70a43256e5a329e7f7450abc42cd1aa9a8f277b0ef7a990dccbbe800` |
| 1004800 | `0xea2e486ef9e96a02e8bfa6782cb2aa36b393783af0b4793dcaa58747f70cd71d` |
| 1352360 | `0xadf9abb289d6e69ca373b4c6dc5853a7c444ea6db5bc1e33b6df2c061eb9444a` |
| 1719100 | `0xe1572e3497370fc796c9722f28c570db392d284f574eb652cb953e39b14a0127` |

---

## What Needs to Be Done

### Phase 1: Copy & Rebrand the Blakecoin 0.15.2 Base

1. **Copy** the entire `Blakecoin-0.15.21` codebase into this directory
2. **Rename** all Blakecoin references → BlakeBitcoin:
   - Binary names: `blakebitcoind`, `blakebitcoin-qt`, `blakebitcoin-cli`, `blakebitcoin-tx`
   - Config file: `blakebitcoin.conf`, config dir `~/.blakebitcoin/`
   - URI scheme: `blakebitcoin://`
   - Desktop entry, icons, window titles
   - `configure.ac`: package name, version
   - All copyright strings to include "BlakeBitcoin developers"

### Phase 2: Apply Coin-Specific Parameters

3. **`src/chainparams.cpp`** — Replace ALL chain parameters with BlakeBitcoin values:
   - Genesis block (hash, merkle root, nTime, nNonce, nBits, coinbase message)
   - Network ports (P2P: 8356, RPC: 8243)
   - Message start bytes (0xf9, 0xbc, 0xa7, 0xb7)
   - Address prefixes (pubkey: 243, script: 7)
   - Block timing (150s block time, 14-day retarget, 8064-block interval)
   - Halving interval: 210,000 blocks
   - Checkpoints (all 12 listed above)
   - DNS seeds
   - Bech32 HRP

4. **`src/amount.h`** — Verify MAX_MONEY = 21,000,000 * COIN (same as Blakecoin)

5. **`src/validation.cpp`** — Block reward logic:
   - Standard halving: 50 BLC → 25 → 12.5 → ... every 210,000 blocks
   - Verify `GetBlockSubsidy()` matches 0.8.x behavior

6. **`src/qt/`** — Update all GUI branding:
   - Window title: "BlakeBitcoin - Wallet"
   - Application name, organization name
   - Icons and splash screen

### Phase 3: AuxPow / Merge Mining

7. **AuxPow integration** — BlakeBitcoin supports merge mining with Blakecoin and Photon
   - Implemented in this repo using modern Namecoin/Dogecoin structure with BlakeBitcoin’s legacy chain constants
   - Added `src/auxpow.{h,cpp}` and `src/primitives/pureheader.{h,cpp}`
   - `src/primitives/block.*`, `src/pow.*`, `src/validation.*`, `src/chain.*`, `src/rest.cpp`, `src/net_processing.cpp`, and `src/rpc/blockchain.cpp` now carry AuxPow-aware header, PoW, and disk-read plumbing
   - Mainnet activation height remains `500000`; testnet and regtest are wired for local activation at height `0`
   - The modern merged-mining RPC path is now implemented and no-send regtest-smoke verified for `createauxblock <address>` plus compatibility `getauxblock`
  - Historical merged-mined header compatibility is now documented from preserved replay evidence; exact activation dating remains archival-only and is no longer the release blocker
   - Mainnet still remains under a strict no-send / no-mine operational rule while final production pool and Electrium staging are incomplete

### Phase 4: Build System

8. **`build.sh`** — Update all variables:
   - COIN_NAME: "blakebitcoin"
   - DAEMON_NAME: "blakebitcoind"
   - QT_NAME: "blakebitcoin-qt"
   - CLI_NAME: "blakebitcoin-cli"
   - TX_NAME: "blakebitcoin-tx"
   - VERSION: "0.15.2"
   - RPC_PORT: 8243
   - P2P_PORT: 8356

9. **Docker configs** — Same Docker images as Blakecoin 0.15.2 (sidgrip/appimage-base:22.04, sidgrip/mxe-base, sidgrip/osxcross-base)

### Phase 5: SegWit Activation

10. **Mainnet SegWit rollout** — Mainnet versionbits signaling starts on May 11, 2026 00:00:00 UTC (`1778457600`) and times out on May 11, 2027 00:00:00 UTC (`1809993600`).
11. **Activation semantics** — May 11, 2026 is the signaling start date, not guaranteed same-day activation. Actual mainnet SegWit enforcement still depends on miner signaling and BIP9 lock-in.
12. **CSV / test networks** — CSV stays `ALWAYS_ACTIVE`, and testnet/regtest keep `ALWAYS_ACTIVE` SegWit for controlled QA and wallet validation.
13. **BIP34/65/66** — Disable version checks (set heights to 100000000) since BlakeBitcoin uses different block versioning

---

## Key Differences from Blakecoin

| Aspect | Blakecoin | BlakeBitcoin |
|--------|-----------|--------------|
| Block Time | 180s (3 min) | 150s (2.5 min) |
| Retarget Interval | 20 blocks | 8,064 blocks |
| Retarget Timespan | 1 hour | 14 days |
| Halving Interval | Disabled (dynamic) | 210,000 blocks |
| Reward | Dynamic (25 + sqrt) | Standard halving (50 → 25 → ...) |
| Max Supply | 21M | 21M |
| Coinbase Maturity | ??? | 100 blocks |
| P2P Port | 8773 | 8356 |
| RPC Port | 8772 | 8243 |
| Pubkey Address | 26 | 243 |
| Merge Mining | No (in donor Blakecoin 0.15.2) | Yes (AuxPow framework now wired) |
| Genesis Date | Oct 2013 | May 2014 |

---

## Potential Issues & Gotchas

1. **AuxPow merge mining** — The shared BlakeBitcoin framework is now in place and fanned out to the other four 0.15.2 update repos. The carry-back staging that used to remain is now green; what remains is rollout discipline, not missing merged-mining RPC support.
2. **Historical AuxPow pre-activation quirk** — The original 0.8.x chain accepts AuxPow-bearing blocks before the configured `GetAuxPowStartBlock()` height. The 0.15.2 port must preserve that behavior for historical bootstrap / IBD compatibility and must not reintroduce an `early-auxpow-block` consensus reject.
3. **Historical BIP30 quirk** — The original 0.8.x chain also does not enforce Bitcoin's broad BIP30 duplicate-txid rule during normal block connection. Re-enabling the stock 0.15.2 logic breaks BlakeBitcoin mainnet sync at height `256205` with `ConnectBlock(): tried to overwrite transaction`.
4. **Retarget interval** — BlakeBitcoin uses Bitcoin's 2-week retarget (8064 blocks) unlike Blakecoin's 20-block retarget. Must preserve this exactly.
5. **Block version** — The conflict is real. This port resolves it by forcing BlakeBitcoin's legacy AuxPow-capable block version format for new block templates instead of Bitcoin Core's BIP9 top-bit pattern.
6. **Halving schedule** — Standard Bitcoin-style halving every 210,000 blocks. Ensure `GetBlockSubsidy()` implements this correctly (not the Blakecoin dynamic formula).
7. **Address prefix 243** — Very high pubkey prefix. Verify Base58 encoding produces valid addresses. Test with existing BlakeBitcoin addresses.
8. **Single DNS seed** — May need additional seeds or hardcoded nodes for network discovery.

---

## Build & Test Plan

1. Rebuild native Linux after any consensus change and keep `blakecoind` / `blakecoin-cli` green
2. Re-run `test/functional/test_runner.py --jobs=1 wallet-basic-smoke.py` on regtest after consensus-touching edits
3. Verify genesis block hash, address generation, and RPC defaults on port `8243`
4. Preserve the archived historical replay notes and do not reintroduce the rejected `early-auxpow-block` rule before any mainnet activity
5. Keep positive `submitauxblock` acceptance testing isolated to regtest or dedicated QA environments
6. Build AppImage, Windows, and macOS artifacts after consensus validation is stable

---

## Verified Snapshot

- Native Linux rebuild succeeded for `blakecoind` and `blakecoin-cli`.
- Fresh regtest no-send smoke passed for `getnewaddress`, `createauxblock <address>`, and compatibility `getauxblock`.
- Verified AuxPow template `chainid` returned as `5` on fresh regtest, matching `consensus.nAuxpowChainId`.
- Verified isolated offline bootstrap import from `~/.blakebitcoin/bootstrap.dat`: the fixed daemon gets past the previous block-1 rejection (`091eb292...`, `early-auxpow-block`) and continued importing through at least height `185839` with no later consensus error seen in the sample run.
- Verified mainnet sync stop root cause from `~/.blakebitcoin/debug.log`: the updated node connects through height `256204`, then rejects block `f05aa8168e7a7cc8d09c9597213f5921a9205ba20e1c6c8d53f6a1e213d207ab` at height `256205` with `ConnectBlock(): tried to overwrite transaction` unless legacy-relaxed BIP30 behavior is preserved.
- Direct `createauxblock` plus `submitauxblock` acceptance is now proven in isolated QA. The production carry-back staging that used to block release is now green.

---

## File Reference

| What | Where |
|------|-------|
| Reference (completed) | `../Blakecoin-0.15.21/` |
| Original coin source | `../BlakeBitcoin/` |
| Original chainparams | `../BlakeBitcoin/src/main.cpp` (0.8.x style) |
| Original build script | `../BlakeBitcoin/build.sh` |
| Blakecoin build notes | `../blakecoin-15.md` |
| SegWit reference | `../Blakecoin-0.15.21/15-2-segwith.md` |

## SegWit Activation Test

- Functional test: `test/functional/segwit-activation-smoke.py`
- Build-server wrapper: `/home/sid/Blakestream-Installer/qa/runtime/run-segwit-activation-suite.sh`
- Direct command used by the wrapper:

```bash
BITCOIND=/path/to/blakebitcoind BITCOINCLI=/path/to/blakebitcoin-cli \
python3 ./test/functional/segwit-activation-smoke.py \
  --srcdir="$(pwd)/src" \
  --tmpdir="<artifact_root>/blakebitcoin/<timestamp>/tmpdir" \
  --nocleanup \
  --loglevel=DEBUG \
  --tracerpc
```

- Expected regtest Bech32 prefix: `rblb1`
- Review artifacts:
  `summary.json`, `state-defined.json`, `state-started.json`, `state-locked_in.json`, `state-active.json`, `address-sanity.json`, `combined-regtest.log`, `tmpdir/test_framework.log`, `tmpdir/node*/regtest/debug.log`
- Successful all-six build-server run:
  `/home/sid/Blakestream-Installer/outputs/segwit-activation/20260412T083423Z/run-summary.md`
- Coin artifact directory:
  `/home/sid/Blakestream-Installer/outputs/segwit-activation/20260412T083423Z/blakebitcoin`
- Harness note:
  the final witness proposal builder now takes the coinbase amount directly from `getblocktemplate()["coinbasevalue"]`, which keeps the activation proof aligned with each chain's real subsidy rules.
- Safety rule:
  regtest only for activation validation; do not mine or send transactions on mainnet while rollout QA is still in progress.

## AuxPoW Testnet Merged-Mining Verification

- Final successful container-built run:
  `/home/sid/Blakestream-Installer/outputs/auxpow-testnet/20260413T003341Z/run-summary.md`
- Wrapper command:
  `bash /home/sid/Blakestream-Installer/qa/auxpow-testnet/run-auxpow-testnet-suite.sh`
- Parent chain:
  Blakecoin testnet only, fully isolated from public peers.
- Live proof result:
  BlakeBitcoin accepted `3` merged-mined child blocks in the pilot, `2` in the 4-child batch, and `1` in the 5-child full run.
- Direct RPC cross-check:
  `createauxblock` plus `submitauxblock` accepted on a fresh BlakeBitcoin testnet pair. Artifact:
  `/home/sid/Blakestream-Installer/outputs/auxpow-testnet/20260413T003341Z/blakebitcoin/rpc-crosscheck.json`
- QC note:
  the batch harness had to promote the proxy merkle size to `8` for the 4-child phase and `16` for the 5-child phase because BlakeBitcoin's chain ID participates in real slot collisions under the Namecoin-style aux index formula.
- Safety rule:
  testnet only for merged-mining QA; do not mine or send transactions on mainnet while AuxPoW rollout validation is still in progress.

## Devnet/Testnet Validation Outcomes

- SegWit activation validation passed on isolated regtest. See:
  `/home/sid/Blakestream-Installer/outputs/segwit-activation/20260412T083423Z/blakebitcoin`
- AuxPoW merged-mining validation passed on isolated testnet, including direct `createauxblock` plus `submitauxblock` cross-checks. See:
  `/home/sid/Blakestream-Installer/outputs/auxpow-testnet/20260413T003341Z/blakebitcoin`
- Mainnet carry-back audit for the devnet copy lives in:
  `mainnet-carryback-audit-2026-04-18.md`
- Audit result:
  the diff between this repo and the devnet `coins/BlakeBitcoin` copy stayed limited to devnet `chainparams*`, Qt network-labeling files, and build cleanup. No new BlakeBitcoin mainnet wallet, consensus, or RPC carry-back was identified from the devnet copy itself.

## Mainnet Carry-Back Decisions

- SegWit rollout remains scheduled, not forced active.
- Mainnet AuxPoW start height remains `500000` as the chain source of truth.
- Do not port devnet network identity, datadir, test shortcuts, or activation shortcuts back into this repo.
- Pool/runtime carry-back work is tracked in the mainnet Eloipool repo.
- Electrium sync and signing carry-back work is tracked in the Electrium repo.
- Mainnet pool integration now depends on the proven multi-miner aux-child payout path in Eloipool, not the old single active mining-key QA shortcut.

## Staging Hygiene

- Keep the intentional autotools and build-system layer in staging for this repo:
  `Makefile.am`, `Makefile.in`, `aclocal.m4`, `autogen.sh`, `configure*`, `build-aux/*`, and `depends/*`.
- Trim generated build junk before review or promotion:
  `.libs/`, `.deps/`, `autom4te.cache/`, `*.o`, `*.lo`, `*.la`, `config.log`, `config.status`, and similar transient outputs.
- April 19, 2026 staging pass explicitly removed staged libtool and univalue build artifacts while preserving the intentional autotools carry-back set.

## Not Carried Back From Devnet

- `src/chainparams.cpp`, `src/chainparamsbase.cpp`, `src/chainparamsbase.h`
- `src/qt/guiconstants.h`, `src/qt/guiutil.cpp`, `src/qt/networkstyle.cpp`
- Any private-testnet `BIP65Height = 1`, `ALWAYS_ACTIVE`, devnet ports, message starts, datadirs, or local-only harness shortcuts
- Pool UI, merged-mine proxy, Electrium, ElectrumX, and builder/runtime scripts

## Pool / Electrium Dependencies

- Mainnet merged-mining now depends on the modern `createauxblock` plus `submitauxblock` direction and the proven multi-miner aux payout model in Eloipool.
- Electrium compatibility now depends on full AuxPoW header support and Blake-family single-SHA signing compatibility.
- Per-coin overlays and branding stay in the Electrium repo and are not folded back into this C++ core tree.

## Safety Rule

- Do not mine on mainnet while carry-back staging is in progress.
- Do not send transactions on mainnet while carry-back staging is in progress.
- Use isolated regtest, testnet, or staging environments until rollout QA is complete.

## April 18, 2026 Devnet Validation Snapshot

- Shared BlakeStream devnet run `20260418T195508Z` proved concurrent multi-miner AuxPoW against the live pool with two mining keys active in the same session.
- Live pooled merged-mined BlakeBitcoin child block proof is green:
  height `449` accepted with `tx_count = 2`, and it contained the queued test tx
  `086ed7d4e0dbea6c2e39d02151d50fe9e96dca64661c7ff78c639cde79c3c1f5`.
- This means the live pool/proxy path is no longer limited to coinbase-only BlakeBitcoin child blocks once mempool transactions are present.

## Mainnet Carry-Back Snapshot

- Keep BlakeBitcoin chain identity, AuxPoW start rules, and scheduled SegWit rollout exactly as already documented in this repo.
- Promote only the proven external dependencies:
  mainnet pool multi-miner mining-key payout plumbing and Electrium full AuxPoW-header plus single-SHA signing compatibility.
- Do not carry back any devnet ports, datadirs, private-testnet activation shortcuts, or runtime wrapper behavior into mainnet chain params.

## April 19, 2026 Broader Electrium Staging Closure

- Broader staged packaged-client proof is now green at:
  `/home/sid/Blakestream-Devnet/outputs/electrium-staging/20260419T053030Z/run-summary.md`
- BlakeBitcoin's packaged Electrium client connected successfully against the
  staged local ElectrumX backend on `127.0.0.1:52001`.
- This run also exposed and closed a shared aux-core startup bug in
  `src/validation.cpp`: height-less disk rereads could treat genesis-like
  headers as regular AuxPoW blocks and fail with a false
  `non-AUX proof of work failed` reject.
- The fix now treats
  `block.GetHash() == consensusParams.hashGenesisBlock || block.hashPrevBlock.IsNull()`
  as genesis-like in the disk-reread path, which keeps standalone staged
  backends honest without relaxing real chained AuxPoW validation.
