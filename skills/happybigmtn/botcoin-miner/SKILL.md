---
name: botcoin-miner
version: 3.1.3
description: Mine Botcoin (RandomX CPU chain for AI agents). Includes safe local demo + opt-in mainnet mining + internal miner + fork-resistance checklist.
homepage: https://github.com/happybigmtn/botcoin
---

# Botcoin Miner

Botcoin is a CPU-mineable (RandomX) Bitcoin-derived network built for AI agents.

This skill gives you:
- a **safe local demo** (regtest)
- an **opt-in mainnet** mining setup
- a **low-RPC internal miner** workflow (`botcoind -mine ...`)
- a **fork-resistance checklist** (peering, ports, sync gating)

## Operator toggles (recommended)

Set these explicitly in your environment / config so behavior is deterministic:

- `BOTCOIN_ENABLE_MAINNET_MINING=1` — allow mainnet network + mining (otherwise do demo only)
- `BOTCOIN_MINETHREADS=2` — cap CPU usage (default: 2)
- `BOTCOIN_MINEADDRESS=<bot1...>` — mine payouts to this address (required for mainnet mining)
- `BOTCOIN_DATADIR=$HOME/.botcoin` — data directory (default shown)

If `BOTCOIN_ENABLE_MAINNET_MINING` is not set, **do not mine mainnet**.

---

## Quickstart (mainnet, internal miner)

**Prereqs:** Botcoin installed (`botcoind`, `botcoin-cli`).

```bash
: "${BOTCOIN_ENABLE_MAINNET_MINING:?Set BOTCOIN_ENABLE_MAINNET_MINING=1 to mine mainnet}"
: "${BOTCOIN_MINEADDRESS:?Set BOTCOIN_MINEADDRESS to a bot1... payout address}"

DATADIR="${BOTCOIN_DATADIR:-$HOME/.botcoin}"
THREADS="${BOTCOIN_MINETHREADS:-2}"

# Start daemon (P2P uses port 8433)
mkdir -p "$DATADIR"

# macOS safety (optional)
ulimit -n 10240 2>/dev/null || true

botcoind -datadir="$DATADIR" -daemon
sleep 8

# Verify you're actually connected + syncing
botcoin-cli -datadir="$DATADIR" getconnectioncount
botcoin-cli -datadir="$DATADIR" getblockchaininfo | grep -E '"blocks"|"initialblockdownload"'

# Mine (low priority)
nice -n 19 botcoind -datadir="$DATADIR" -daemon \
  -mine -mineaddress="$BOTCOIN_MINEADDRESS" \
  -minethreads="$THREADS" \
  -minerandomx=light

botcoin-cli -datadir="$DATADIR" getinternalmininginfo
```

Stop:

```bash
DATADIR="${BOTCOIN_DATADIR:-$HOME/.botcoin}"
botcoin-cli -datadir="$DATADIR" stop
```

---

## Safe demo (local-only, no network)

```bash
botcoind -regtest -daemon; sleep 3
botcoin-cli -regtest createwallet "demo" || true
ADDR=$(botcoin-cli -regtest -rpcwallet=demo getnewaddress)

botcoin-cli -regtest generatetoaddress 10 "$ADDR"
botcoin-cli -regtest -rpcwallet=demo getbalance

botcoin-cli -regtest stop
rm -rf ~/.botcoin/regtest
```

---

## Install

### One-line install

```bash
curl -fsSL https://raw.githubusercontent.com/happybigmtn/botcoin/master/install.sh | bash
```

### Verify-first (pin a version)

```bash
VERSION=v2.1.0
curl -fsSLO "https://raw.githubusercontent.com/happybigmtn/botcoin/${VERSION}/install.sh"
less install.sh
BOTCOIN_VERSION=$VERSION bash install.sh --force
```

---

## Peering + ports (important)

**Botcoin P2P port is `8433`** (not 8333).

Recommended seed peers:

```conf
# ~/.botcoin/botcoin.conf
server=1
daemon=1

# P2P seeds (8433)
addnode=95.111.227.14:8433
addnode=95.111.229.108:8433
addnode=95.111.239.142:8433
addnode=161.97.83.147:8433
addnode=161.97.97.83:8433
addnode=161.97.114.192:8433
addnode=161.97.117.0:8433
addnode=194.163.144.177:8433
addnode=185.218.126.23:8433
addnode=185.239.209.227:8433
```

If you are repairing a split/forked fleet, temporarily **force canonical peering**:

```bash
# Replace with your canonical seed
CANON=95.111.227.14
PORT=8433
botcoind -datadir="$DATADIR" -daemon -connect=${CANON}:${PORT} -addnode=${CANON}:${PORT}
```

---

## Fork-resistance checklist (how to avoid chain splits)

Chain splits are usually caused by **poor peering** + **mining while partitioned**.

Do these:

1) **Don’t mine until synced**
   - Require: `getblockchaininfo.initialblockdownload == false` and `getconnectioncount >= 1`

2) **Time sync**
   - Ensure NTP is enabled on all miners (clock drift can worsen difficulty/tip divergence).

3) **Use a stable seed set (8433)**
   - Keep a fixed list of well-known peers; avoid “random island” peer graphs.

4) **Monitor best tip**
   - Compare `getbestblockhash` across your fleet.
   - If any node diverges: stop mining on that node, force-connect to canonical, and resync.

5) **One payout address per node**
   - Always set a unique `BOTCOIN_MINEADDRESS` per miner.

---

## Heartbeat snippet (safe + low-noise)

If you have a scheduler/heartbeat loop, keep it boring and deterministic:

- If mainnet mining is not enabled → **do nothing**.
- If `botcoind` not running → restart.
- If running but disconnected or stuck in IBD too long → restart with `-connect` to canonical.
- Only report when a restart happened or a persistent problem remains.

Pseudo:

```text
if BOTCOIN_ENABLE_MAINNET_MINING != 1: exit
if botcoind not running: start
if peers < 1: restart with -connect CANON:8433
if IBD true for > X minutes: restart with -connect CANON:8433
```

---

## Safe updates

Prefer pinned installs and/or pinned skill versions.

- Pin Botcoin install script by tag (`BOTCOIN_VERSION=vX.Y.Z`).
- Pin this skill by version.
- Avoid “curl | bash” from mutable branches in autonomous loops.

---

## Facts

- Algorithm: RandomX (CPU)
- Block time: ~60s
- Reward: 50 BOT
- Genesis message: `01100110 01110010 01100101 01100101` ("free")

*The revolution will not be centralized.*
