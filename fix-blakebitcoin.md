# BlakeBitcoin Fixes

## 1. Ports — Privileged Port Issue

Ports 243 (RPC) and 356 (P2P) are in the Unix privileged range (1-1024).
On Linux and macOS, binding to these ports requires root — the wallet fails
to start without a config file overriding the ports.

### Current code (matches original BlueDragon747):

**src/protocol.h:21** — P2P port
```c
return testnet ? 18112 : 356;
```

**src/bitcoinrpc.cpp:44** — RPC port
```c
return GetBoolArg("-testnet", false) ?  1812: 243;
```

**src/init.cpp:305** — help text
```
(default: 356 or testnet: 18112)
```

**src/init.cpp:348** — help text
```
(default: 243 or testnet: 1812)
```

**build.sh:32-33**
```bash
RPC_PORT=243
P2P_PORT=356
```

### Change to:

**src/protocol.h:21**
```c
return testnet ? 18112 : 8356;
```

**src/bitcoinrpc.cpp:44**
```c
return GetBoolArg("-testnet", false) ?  1812: 8243;
```

**src/init.cpp:305**
```
(default: 8356 or testnet: 18112)
```

**src/init.cpp:348**
```
(default: 8243 or testnet: 1812)
```

**build.sh:32-33**
```bash
RPC_PORT=8243
P2P_PORT=8356
```

## 2. Consensus Parameters — No Changes

All consensus parameters match the original BlakeBitcoin/BlakeBitcoin repo
(https://github.com/BlakeBitcoin/BlakeBitcoin). No changes made.

**src/main.cpp:1093-1095**
```c
static const int64 nTargetTimespan = 14 * 24 * 60 * 60; // two weeks
static const int64 nTargetSpacing = 150;
static const int64 nInterval = nTargetTimespan / nTargetSpacing;
```
