# Validator Set
This project has validator-set pallet implemented. It helps with adding validators in an Aura based network.
Pallet is extremely useful and could be utilized to work in a public block chain.

Note: "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0"

## Pallet repo
```
https://github.com/gautamdhameja/substrate-validator-set
```
## last compiled using the following version
stable-x86_64-unknown-linux-gnu (default)
rustc 1.78.0 (9b00956e5 2024-04-29)

High level Steps:
1) Get the latest version of substrate-node-template (polkadot-v1.9.0)
2) Update runtime/Cargo.toml file
3) Update runtime/src/lib.rs file
4) Update node/src/chain_spec.rs file

```
git clone https://github.com/substrate-developer-hub/substrate-node-template](https://github.com/fzeeshan/ValidatorSet
```
build the base version.
for example:
```
cargo build --release
```
or
```
cargo +nightly-2024-04-29 build --release
```
to Launch:
```
./target/release/node-template -d data/node1 --dev
```



