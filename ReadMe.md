# Validator Set
This project has validator-set pallet implemented. It helps with adding validators in an Aura based network.
Pallet is extremely useful and could be utilized to work in a public block chain.

Note: "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0"

## Pallet repo
```
https://github.com/gautamdhameja/substrate-validator-set
```
## Last compiled using the following version
stable-x86_64-unknown-linux-gnu (default)
rustc 1.78.0 (9b00956e5 2024-04-29)

High level Steps:
1) Get the latest version of substrate-node-template (polkadot-v1.9.0)
2) Update runtime/Cargo.toml file
3) Update runtime/src/lib.rs file
4) Update node/src/chain_spec.rs file

```
git clone https://github.com/fzeeshan/ValidatorSet
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

Summary of Changes:
## Update runtime/Cargo.toml file:
open runtime/Cargo.toml

under [dependencies] Add the following 
substrate-validator-set = { git = "https://github.com/gautamdhameja/substrate-validator-set.git", tag = "polkadot-v1.9.0", default-features = false } 
pallet-session = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0", default-features = false } 

Under "[features] default = ["std"] std = [ .." add the following:
"substrate-validator-set/std",
"pallet-session/std",

## Update runtime/src/lib.rs file:

// added OpaqueKeys to traits::

this is what it looks like after change

use sp_runtime::{
	create_runtime_str, generic, impl_opaque_keys,
	traits::{BlakeTwo256, Block as BlockT, Verify, IdentifyAccount, NumberFor, One, OpaqueKeys},
	transaction_validity::{TransactionSource, TransactionValidity},
	ApplyExtrinsicResult, MultiSignature,
};
//added ensureroot
use frame_system::EnsureRoot;

//fz added for substrate-validator-set
parameter_types! {
	pub const MinAuthorities: u32 = 2;
}

impl substrate_validator_set::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
	type AddRemoveOrigin = EnsureRoot<AccountId>;
	type MinAuthorities = MinAuthorities;
	type WeightInfo = substrate_validator_set::weights::SubstrateWeight<Runtime>;
}

parameter_types! {
	pub const Period: u32 = 2 * MINUTES;
	pub const Offset: u32 = 0;
}

impl pallet_session::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
	type ValidatorId = <Self as frame_system::Config>::AccountId;
	type ValidatorIdOf = substrate_validator_set::ValidatorOf<Self>;
	type ShouldEndSession = pallet_session::PeriodicSessions<Period, Offset>;
	type NextSessionRotation = pallet_session::PeriodicSessions<Period, Offset>;
	type SessionManager = ValidatorSet;
	type SessionHandler = <opaque::SessionKeys as OpaqueKeys>::KeyTypeIdProviders;
	type Keys = opaque::SessionKeys;
	type WeightInfo = ();
}
//fz ends

Update Construct runtime.
	pub struct Runtime;

	#[runtime::pallet_index(0)]
	pub type System = frame_system;

	#[runtime::pallet_index(1)]
	pub type Timestamp = pallet_timestamp;

	//balances
	#[runtime::pallet_index(2)]
	pub type Balances = pallet_balances;
	//validatorset added
	#[runtime::pallet_index(3)]
	pub type ValidatorSet = substrate_validator_set;

	//session added
	#[runtime::pallet_index(4)]
	pub type Session = pallet_session;

	#[runtime::pallet_index(5)]
	pub type Aura = pallet_aura;

	#[runtime::pallet_index(6)]
	pub type Grandpa = pallet_grandpa;

	//balances was after grandpa

	#[runtime::pallet_index(7)]
	pub type TransactionPayment = pallet_transaction_payment;

	#[runtime::pallet_index(8)]
	pub type Sudo = pallet_sudo;

	// Include the custom logic from the pallet-template in the runtime.
	#[runtime::pallet_index(9)]
	pub type TemplateModule = pallet_template;


## Update node/src/chain_spec.rs file:
// Add a new function
/// fz adding a new session_key function and updating authority_keys_from_seed
fn session_keys(aura: AuraId, grandpa: GrandpaId) -> SessionKeys {
	SessionKeys { aura, grandpa }
}
// fz ends

// update an existing function
/// Generate an Aura authority key.
pub fn authority_keys_from_seed(s: &str) -> (AccountId, AuraId, GrandpaId) {
	(
		get_account_id_from_seed::<sr25519::Public>(s),
		get_from_seed::<AuraId>(s),
		get_from_seed::<GrandpaId>(s)
	)
}



// update the testnet_genesis function
/// Configure initial storage state for FRAME modules.
fn testnet_genesis(
	initial_authorities: Vec<(AccountId, AuraId, GrandpaId)>,//initial authorities updated
	root_key: AccountId,
	endowed_accounts: Vec<AccountId>,
	_enable_println: bool,
) -> serde_json::Value {
	serde_json::json!({
		"balances": {
			// Configure endowed accounts with initial balance of 1 << 60.
			"balances": endowed_accounts.iter().cloned().map(|k| (k, 1u64 << 60)).collect::<Vec<_>>(),
		},
		"substrate_validator_set": {
			"initial_validators": initial_authorities.iter().map(|x| x.0.clone()).collect::<Vec<_>>(),
		},
		"session": {
			"keys": initial_authorities
				.iter()
				.map(|x| (
					x.0.clone(),
					x.0.clone(),
					session_keys(x.1.clone(), x.2.clone())
				))
				.collect::<Vec<_>>(),
		},
		"aura": {
			"authorities": [],
		},
		"grandpa": {
			"authorities": [],
		},
		"sudo": {
			// Assign network admin rights.
			"key": Some(root_key),
		},
	})
}

