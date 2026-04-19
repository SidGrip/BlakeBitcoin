# QC Report — BlakeBitcoin 0.15.2 Update

Date: 2026-04-11
Scope: fresh verification of source-of-truth `BlakeBitcoin-0.15.2-update.md` against the code in `src/`, cross-referenced against the original 0.8.x tree in `../BlakeBitcoin/src/`.

---

## Coin Metrics Summary

| Parameter | Expected | 0.15.2 Code | Original 0.8.x | Status |
|-----------|----------|-------------|----------------|--------|
| Coin name (binary) | blakebitcoin | `blakebitcoind`/`blakebitcoin-qt`/`blakebitcoin-cli`/`blakebitcoin-tx` | n/a | PASS |
| `configure.ac` package | BlakeBitcoin Core | `AC_INIT([BlakeBitcoin Core], ..., [blakebitcoin], ...)` | n/a | PASS |
| CURRENCY_UNIT | BBTC | `"BBTC"` (`src/policy/feerate.cpp:11`) | n/a | PASS (note: SoT identity table still says `BLC`) |
| P2P port (main) | 8356 | `nDefaultPort = 8356` | `8356` (`protocol.h:21`) | PASS |
| P2P port (test) | 18112 | `nDefaultPort = 18112` | `18112` | PASS |
| P2P port (regtest) | 18444 | `nDefaultPort = 18444` | n/a (0.8 has no regtest) | PASS |
| RPC port (main) | 8243 | `nRPCPort = 8243` | `8243` (`bitcoinrpc.cpp:44`) | PASS |
| RPC port (test) | 1812 | `nRPCPort = 1812` | `1812` | PASS |
| RPC port (regtest) | 18443 | `nRPCPort = 18443` | n/a | PASS |
| pchMessageStart (main) | f9 bc a7 b7 | `0xf9, 0xbc, 0xa7, 0xb7` | `0xf9, 0xbc, 0xa7, 0xb7` (`main.cpp:3150`) | PASS |
| pchMessageStart (test) | 0b 11 12 09 | `0x0b, 0x11, 0x12, 0x09` | `0x0b, 0x11, 0x12, 0x09` (`main.cpp:2816`) | PASS |
| pchMessageStart (regtest) | fa bf b5 da | `0xfa, 0xbf, 0xb5, 0xda` | n/a | PASS |
| Pubkey prefix (main) | 243 | `243` | `243` (`base58.h:276`) | PASS |
| Pubkey prefix (test) | 142 | `142` | `142` (`base58.h:278`) | PASS |
| Script prefix (main) | 7 | `7` | `7` (`base58.h:277`) | PASS |
| Script prefix (test) | 170 | `170` | `170` (`base58.h:279`) | PASS |
| Secret key (main) | 128 | `128` | `128` (`base58.h:405`) | PASS |
| Secret key (test) | 239 | `239` | `239` (`base58.h:405`) | PASS |
| bech32 HRP (main/test/reg) | blb / tblb / rblb | `blb` / `tblb` / `rblb` | n/a | PASS |
| Block time | 150 s | `nPowTargetSpacing = 150` | `nTargetSpacing = 150` (`main.cpp:1094`) | PASS |
| Retarget timespan | 14 days | `nPowTargetTimespan = 14*24*60*60` | 14 days (implicit: 8064 * 150) | PASS |
| Retarget interval | 8064 blocks | `DifficultyAdjustmentInterval() = 8064` (timespan/spacing) | 8064 (nInterval) | PASS |
| Halving interval | 210000 | `nSubsidyHalvingInterval = 210000` | `>> (nHeight/210000)` (`main.cpp:1101`) | PASS |
| Initial reward | 50 | `50 * COIN` (validation.cpp:1096) | `50 * COIN` (`main.cpp:1098`) | PASS |
| Subsidy model | halving | `nSubsidy >>= halvings` | `nSubsidy >>= (nHeight/210000)` | PASS |
| MAX_MONEY | 21,000,000 * COIN | `21000000 * COIN` (`amount.h:26`) | `21000000 * COIN` (`main.h:58`) | PASS |
| COINBASE_MATURITY | 100 | `100` (`consensus/consensus.h:19`) | `100` (`main.h:61`) | PASS |
| AuxPow chain ID | 0x0005 | `nAuxpowChainId = 0x0005` (all 3 nets) | `GetOurChainID() = 0x0005` (`main.cpp:2028`) | PASS |
| AuxPow start (main) | 500000 | `nAuxpowStartHeight = 500000` | `GetAuxPowStartBlock() = 500000` (`main.cpp:2023`) | PASS |
| AuxPow start (test/reg) | 0 | `nAuxpowStartHeight = 0` | testnet `0` (`main.cpp:2021`) | PASS |
| Strict chain-ID (main) | on | `fStrictChainId = true` | enforced non-testnet (`main.cpp:2039`) | PASS |
| Strict chain-ID (test/reg) | off | `fStrictChainId = false` | off on testnet | PASS |
| Genesis hash (main) | `000000dc…3371` | matches | matches (`main.cpp:39`) | PASS |
| Genesis nTime / nNonce / nVersion | 1399109785 / 183893667 / 112 | matches | matches | PASS |
| Merkle root | `04231416…d0ac` | matches (assert) | n/a | PASS |
| Testnet genesis | inherited `00000052…0d5b`, 1392351202, 4335147 was invalid for fresh 0.15.2 testnet boot; corrected private QA testnet uses `00000073…f7ad`, 1392351202, 32975318 | corrected | `chainparams.cpp` | PASS |
| Checkpoints | all 12 | all 12 present and exact-match | all 12 present (`checkpoints.cpp:36-49`) | PASS |
| DNS seeds | 2 shared ecosystem seeds | `blakestream.io` + `blakecoin.org` | n/a | PASS (see Note A) |
| Coinbase message | "Added to the Blake-256 Merge Mining 12th May 2014" | matches | n/a | PASS |

**Note A**: The SoT "DNS Seeds" bullet in the identity table still reads `blakecrypto.com (single seed; still needs review once AuxPow lands)`, which is stale. The shipping code uses the shared BlakeStream ecosystem seeds (`blakestream.io`, `blakecoin.org`), which is consistent with the **BlakeStream Seed And AuxPoW RPC Policy** section of the same document but contradicts the identity table. See Action Items.

---

## Source-of-Truth Claim Verification

| # | Claim (from QC Status / Completed / Verified Snapshot) | Status | Evidence |
|---|---|---|---|
| 1 | QC strong enough to be shared 0.15.2 AuxPow reference port, but merged-mining RPC not finished | PASS | AuxPow framework present; `submitauxblock` solved-payload acceptance still pending per SoT itself |
| 2 | Builds and passes `wallet-basic-smoke.py` on regtest, no sends | NOT RE-RUN | No functional test run in this QC pass (code only) |
| 3 | Verified from 0.8 source: ports, pchMessageStart, address prefixes, message header, 210k halving | PASS | All verified above against `../BlakeBitcoin/src` |
| 4 | Corrected from source: secret-key prefixes `128` / `239` (not TBD) | PASS | `chainparams.cpp:142` = 128, `:239` = 239, matches `base58.h:405` |
| 5 | Regtest is local-only bootstrap with AuxPow activation at height 0 | PASS | `chainparams.cpp:286` `nAuxpowStartHeight = 0` regtest |
| 6 | Shared AuxPoW framework integrated: `auxpow.{h,cpp}`, `pureheader.{h,cpp}`, AuxPow-aware block/header serialization, disk index persistence, AuxPow-aware PoW validation, chain-ID-aware block template versions | PASS | See AuxPoW Framework Integration section |
| 7 | Mainnet keeps strict chain-ID enforcement; chain ID `0x0005`, start `500000` | PASS | `fStrictChainId=true`, `nAuxpowChainId=0x0005`, `nAuxpowStartHeight=500000` |
| 8 | Testnet/regtest use chain ID `0x0005`, start `0`, strict chain ID disabled | PASS | Both have `fStrictChainId=false` and `nAuxpowStartHeight=0` |
| 9 | Build of `blakecoind`/`blakecoin-cli` + wallet-basic-smoke pass | DOC DRIFT | Old binary names in SoT text; actual built binaries are `blakebitcoind`/`blakebitcoin-cli` and both are present as built artifacts in `src/`. Not re-built in this QC pass. |
| 10 | Historical bootstrap compat fix: no `early-auxpow-block` reject | PASS | `pow.cpp:122-152` `CheckAuxPowProofOfWork` — when `!auxpowActive`, auxpow-bearing blocks return true without rejection. Grep for `early-auxpow-block` returns zero matches in 0.15.2 src. |
| 11 | Historical BIP30 compat fix: relaxed to `!pindex->phashBlock` only | PASS | `validation.cpp:1777` `bool fEnforceBIP30 = !pindex->phashBlock;` with broader Bitcoin Core logic removed and an explicit comment referencing height `256205` |
| 12 | Merged-mining RPC direction fixed: primary `createauxblock <address>` + `submitauxblock <hash> <auxpow>`, `getauxblock` as compat wrapper; `getworkaux` out of scope | PASS | See RPC Surface section |
| 13 | Strict no-send / no-mine mainnet rule | POLICY | Not enforceable in code; operational note only |
| 14 | **Seed Policy**: BlakeStream shared DNS OK if each coin resolves to its own network | PASS (code) / PARTIAL (docs) | `chainparams.cpp:137-138` uses shared ecosystem hostnames; identity table still lists `blakecrypto.com` |
| 15 | **Wire Checksum Policy**: preserve legacy `Hashblake` P2P checksum; do not adopt Blakecoin's non-`Hashblake` handshake exception | PASS | `net.cpp:832` and `net.cpp:2931` use `Hashblake(...)` for both inbound and outbound message checksums; `hash.h:90` defines `Hashblake`. No Blakecoin-style exception branch present. |
| 16 | Completed: nominal mainnet start height `500000` preserved, `early-auxpow-block` reject removed | PASS | Matches claims 7 and 10 |
| 17 | Completed: integrated shared AuxPoW framework files and AuxPow-aware block/header serialization, disk index persistence, block version handling, PoW validation | PASS | Verified in framework section below |
| 18 | Completed: `createauxblock <address>` + `submitauxblock` implemented, `getauxblock` retained as compat wrapper | PASS | `rpc/mining.cpp:936`, `:967`, `:996`, registered at `:1264-1266` |
| 19 | Verified AuxPow template `chainid` returned as `5` on regtest | PASS (code path) | `rpc/mining.cpp` template emit path forwards `consensusParams.nAuxpowChainId` (0x0005 = 5) |
| 20 | Verified isolated bootstrap import past block-1 `early-auxpow-block` rejection | PASS (code) | The reject condition is gone in `pow.cpp` |
| 21 | Verified mainnet sync previously stopping at `256205` with BIP30 overwrite | PASS (code) | BIP30 relaxation in place in `validation.cpp:1777` |
| 22 | SegWit signaling: start `1778457600`, timeout `1809993600` | PASS | `chainparams.cpp:109-110` |
| 23 | CSV always active; testnet/regtest SegWit always active | PASS | `chainparams.cpp:103-104, 213-214, 293-297` |
| 24 | BIP34/65/66 heights at `100000000` (disabled) on main/test | PASS | `chainparams.cpp:83-86, 188-191` |

---

## AuxPoW Framework Integration

All expected framework files exist in the 0.15.2 tree and carry the shared Namecoin/Dogecoin-style structure adapted to BlakeBitcoin's chain constants.

| File | Size / Lines | Presence / Role |
|------|--------------|-----------------|
| `src/auxpow.h` | 82 lines | PRESENT — declares `CAuxPow` with `coinbaseTx`, `vMerkleBranch`, `vChainMerkleBranch`, `nChainIndex`, `parentBlock` (`CPureBlockHeader`); `Check(hashAuxBlock, nChainId, params)`; `GetParentBlockPoWHash`, `GetExpectedIndex`, `InitAuxPow` |
| `src/auxpow.cpp` | 127 lines | PRESENT — implements `CAuxPow::Check`, Merkle branch validation, merged-mining header parsing, `InitAuxPow` |
| `src/primitives/pureheader.h` | 110 lines | PRESENT — `CPureBlockHeader` with `BLOCK_VERSION_DEFAULT=(1<<4)`, `VERSION_AUXPOW=(1<<8)`, `VERSION_CHAIN_START=(1<<16)`, `GetBaseVersion`, `SetBaseVersion(baseVersion, chainId)`, `GetChainId`, `IsAuxpow`, `SetAuxpowFlag` |
| `src/primitives/pureheader.cpp` | 19 lines | PRESENT — `GetHash()` / `GetPoWHash()` |
| `src/primitives/block.h` | AuxPow-aware | PRESENT — `#include "auxpow.h"`, `std::shared_ptr<CAuxPow> auxpow`, conditional `READWRITE(*auxpow)` in `SerializationOp` based on `IsAuxpow()`, `SetAuxpow(CAuxPow*)`, `block.auxpow = auxpow` in slice copy |
| `src/consensus/params.h` | AuxPow fields | PRESENT — `nAuxpowChainId`, `nAuxpowStartHeight`, `fStrictChainId` |
| `src/pow.cpp` | `CheckAuxPowProofOfWork` | PRESENT — checks strict chain ID, auxpow presence vs `IsAuxpow()` flag, parent PoW, AuxPow Merkle branch; does **not** reject auxpow-bearing blocks when `!auxpowActive` |
| `src/validation.cpp` | AuxPow-aware accept | PRESENT — `nAuxpowStartHeight` consulted at line 3026; BIP30 relaxed at 1777 |
| `src/miner.cpp` | Chain-ID template version | PRESENT — `miner.cpp:137-144` calls `pblock->SetBaseVersion(nBaseVersion, consensusParams.nAuxpowChainId)` when `nHeight >= nAuxpowStartHeight` |

Framework integration: complete for header, serialization, PoW, and template paths.

---

## RPC Surface

Merged-mining RPC set in `src/rpc/mining.cpp`:

| RPC | Signature | Line | Role |
|-----|-----------|------|------|
| `createauxblock` | `createauxblock <address>` | 936 | Primary template builder — address-driven so pool chooses payout script explicitly; no daemon wallet dependency |
| `submitauxblock` | `submitauxblock <hash> <auxpow>` | 967 | Primary submit path for solved AuxPoW payloads |
| `getauxblock` | `getauxblock ( hash auxpow )` | 996 | Compatibility wrapper: no-arg form calls `AuxMiningCreateBlock` using the wallet keypool (legacy behavior); with-arg form decodes AuxPow and submits — maps onto the same block-template / block-submit flow |

Registration at `rpc/mining.cpp:1264-1266`:
```
{ "mining", "getauxblock",    &getauxblock,    true, {"hash","auxpow"} },
{ "mining", "createauxblock", &createauxblock, true, {"address"}      },
{ "mining", "submitauxblock", &submitauxblock, true, {"hash","auxpow"}},
```

`getworkaux` is **not** present (`grep getworkaux` returns nothing), matching the SoT "out of scope unless a real dependency is later proven" policy.

RPC direction policy: **PASS**. Matches the Merged-Mining RPC Direction section of the SoT.

---

## Cross-Check Highlights vs Original 0.8.x

| Item | 0.8.x location | 0.15.2 location | Match |
|------|----------------|-----------------|-------|
| Genesis hash | `main.cpp:39` | `chainparams.cpp:133` | EXACT |
| AuxPow start 500000 | `main.cpp:2023` | `chainparams.cpp:94` | EXACT |
| Chain ID 0x0005 | `main.cpp:2028` | `chainparams.cpp:93,198,285` | EXACT |
| MessageStart mainnet | `main.cpp:3150` | `chainparams.cpp:123-126` | EXACT |
| MessageStart testnet | `main.cpp:2816-2819` | `chainparams.cpp:222-225` | EXACT |
| P2P ports 8356/18112 | `protocol.h:21` | `chainparams.cpp:127,226` | EXACT |
| RPC ports 8243/1812 | `bitcoinrpc.cpp:44` | `chainparamsbase.cpp:35,47` | EXACT |
| Base58 prefixes 243/7/128 main | `base58.h:276-277,405` | `chainparams.cpp:140-142` | EXACT |
| Base58 prefixes 142/170/239 test | `base58.h:278-279,405` | `chainparams.cpp:237-239` | EXACT |
| MAX_MONEY 21M*COIN | `main.h:58` | `amount.h:26` | EXACT |
| COINBASE_MATURITY 100 | `main.h:61` | `consensus/consensus.h:19` | EXACT |
| Subsidy: `50*COIN >> (h/210000)` | `main.cpp:1098,1101` | `validation.cpp:1089-1097` | SEMANTIC MATCH |
| Target spacing 150s | `main.cpp:1094` | `chainparams.cpp:89,194,281` | EXACT |
| Retarget interval 8064 | `main.cpp:1095` (nInterval=timespan/spacing) | timespan `14*24*60*60` / spacing `150` = 8064 | EXACT |
| All 12 checkpoints | `checkpoints.cpp:36-49` | `chainparams.cpp:157-168` | EXACT |
| Testnet genesis `00000052…0d5b` | `main.cpp:2820` | `chainparams.cpp:230` | EXACT |

---

## Action Items

1. **SoT identity-table drift (DNS seeds)** — Update the `## DNS Seeds` line in `BlakeBitcoin-0.15.2-update.md` from `blakecrypto.com (single seed; still needs review once AuxPow lands)` to the shipping pair `seed.blakestream.io`, `seed.blakecoin.org`, to match both the code and the `BlakeStream Seed And AuxPoW RPC Policy` section.

2. **SoT identity-table drift (ticker)** — The Coin Identity table says `Ticker: BLC`, but `CURRENCY_UNIT = "BBTC"` in `src/policy/feerate.cpp`. Update the table to `BBTC` (or document explicitly why the display ticker diverges from `CURRENCY_UNIT`). The `Max Supply` and `Initial Block Reward` cells in the Block Parameters table also read `BLC` — align those too.

3. **Two cross-referenced build-log lines still say `blakecoind` / `blakecoin-cli`** — QC Status bullet 2/7 references `make -C src -j4 blakecoind blakecoin-cli` and the `Verified Snapshot` section says "Native Linux rebuild succeeded for `blakecoind` and `blakecoin-cli`." These should be renamed to `blakebitcoind` / `blakebitcoin-cli`. The src tree does contain leftover `blakecoind` / `blakecoin-cli` artifacts from the seed copy alongside the correct `blakebitcoind` / `blakebitcoin-cli` binaries — the leftovers are harmless but confusing and should be cleaned before release.

4. **Copyright string in `src/pow.cpp:3`** — still reads `Copyright (c) 2013-2026 The Blakecoin Developers`. Cosmetic, but should say `BlakeBitcoin Developers` (as `chainparams.cpp:3` already does).

5. **Copyright string in `src/primitives/pureheader.h:3`** — still says `The Blakecoin Developers`. Same cosmetic rename.

6. **Pending (already noted in SoT)** — Solved `submitauxblock` acceptance QA in isolated regtest / dedicated QA environment still outstanding before release-readiness. No code change required; operational task.

7. **Pending (already noted in SoT)** — Historical merged-mined header coverage QC on mainnet IBD still outstanding. No code change required.

8. **DNS seed policy** — PASS. `seed.blakestream.io` and `seed.blakecoin.org` are the 2 shared BlakeStream ecosystem seeds used by all 6 coins. Per policy, these seeds serve nodes for ALL coins; coin separation happens at the wire-protocol layer via `pchMessageStart` and port. Matches the Blakecoin 0.15.2 reference repo exactly. No action needed.

---

## Overall Verdict

The 0.15.2 update repo matches the source-of-truth document on every code-verifiable claim. AuxPoW framework, BIP30 relaxation, AuxPoW pre-activation tolerance, strict chain-ID enforcement, modern merged-mining RPC surface, and wire checksum preservation are all confirmed present and consistent with BlakeBitcoin's legacy 0.8.x behavior and with the policy sections of the SoT.

Remaining items are:
- Small SoT documentation drift: DNS seed table, ticker string, and two outdated build-log lines — Action Items 1-3.
- Two cosmetic copyright strings still say "Blakecoin Developers" — Action Items 4-5.
- Operational QA still outstanding per the SoT itself: isolated `submitauxblock` solved-payload acceptance, and historical merged-mined header coverage on mainnet IBD.

No consensus-level issues found. Repo remains safe under the documented "strict no-send / no-mine mainnet" operational rule.
