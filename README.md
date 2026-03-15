# ⛓️ rust-blockchain

A peer-to-peer blockchain built in Rust. Got curious about how the core mechanics actually work under the hood, so I followed the [LogRocket guide](https://blog.logrocket.com/how-to-build-a-blockchain-in-rust/) and built it out myself.

Nodes discover each other automatically on a local network, mine blocks using proof-of-work, and sync their chains using a longest-valid-chain consensus rule.

---

## What it does

- **Proof-of-work mining**: each block is mined by incrementing a nonce until the SHA-256 hash meets the difficulty target (`DIFFICULTY_PREFIX = "00"`)
- **Chain validation**: every block is checked against its predecessor: previous hash, sequence ID, difficulty, and hash integrity
- **Consensus**: when two peers connect, they exchange chains and keep whichever one is longer and valid
- **P2P networking**: built on [libp2p](https://libp2p.io/). Peers discover each other via mDNS and broadcast new blocks using the Floodsub gossip protocol
- **Encrypted transport**: connections are authenticated using the Noise protocol (`XX` handshake pattern) and multiplexed with mplex

---

## Project structure

```
BC_repl/
└── src/
    ├── main.rs   # App logic, blockchain core, async event loop
    └── p2p.rs    # libp2p network behaviour, peer discovery, message handling
```

---

## Getting started

**Prerequisites:** Rust + Cargo

```bash
git clone https://github.com/monalgupta03/rust-blockchain
cd rust-blockchain
RUST_LOG=info cargo run
```

> `RUST_LOG=info` turns on the logger so you can see mining progress, peer connections, and block broadcasts in real time.

To simulate a P2P network, open multiple terminals and run `cargo run` in each. They'll discover each other automatically via mDNS.

---

## CLI commands

Once running, you can type commands into stdin:

| Command | What it does |
|---|---|
| `ls p` | List connected peers |
| `ls c` | Print your local chain as JSON |
| `create b <data>` | Mine a new block with the given data and broadcast it |

---

## How the mining works

Each block stores an `id`, `timestamp`, `data`, `previous_hash`, `nonce`, and `hash`. When you create a block, `mine_block()` runs a loop hashing the block contents with an incrementing nonce until the binary representation of the SHA-256 hash starts with `"00"`. That's the proof of work.

```rust
if binary_hash.starts_with(DIFFICULTY_PREFIX) {
    return (nonce, hex::encode(hash));
}
nonce += 1;
```

Validity is checked on every incoming block before it gets added to the chain.

---

## Key dependencies

- [`libp2p`](https://crates.io/crates/libp2p) — P2P networking (mDNS, Floodsub, Noise, mplex)
- [`tokio`](https://crates.io/crates/tokio) — async runtime
- [`sha2`](https://crates.io/crates/sha2) — SHA-256 hashing
- [`serde` / `serde_json`](https://crates.io/crates/serde) — serialisation
- [`hex`](https://crates.io/crates/hex) — hash encoding

---

## What I'd add next

- [ ] Configurable difficulty (right now hardcoded to `"00"`)
- [ ] Persist the chain to disk so it survives restarts
- [ ] A proper transaction model instead of raw string data
- [ ] Basic wallet/keypair support for signing transactions

---

## Based on

[How to build a blockchain in Rust — LogRocket Blog](https://blog.logrocket.com/how-to-build-a-blockchain-in-rust/)
