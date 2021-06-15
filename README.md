<img src="xeno_black.png" alt="Xeno NFT Hub" style="width:200px;"/>

# Xeno NFT Hub Substrate Node

## Getting Started

Follow the steps below to deploy and run a Xeno NFT Hub Node

### Using Nix

Install [nix](https://nixos.org/) and optionally [direnv](https://github.com/direnv/direnv) and [lorri](https://github.com/target/lorri) for a fully plug
and play experience for setting up the development environment. To get all the correct dependencies activate direnv `direnv allow` and lorri `lorri shell`.

### Rust Setup

First, complete the [basic Rust setup instructions](./docs/rust-setup.md).

### Run

Use Rust's native `cargo` command to build and launch the node:

```sh
cargo run --release -- --dev --tmp
```

### Build

The `cargo run` command will perform an initial build. Use the following command to build the node
without launching it:

```sh
cargo build --release
```

### Embedded Docs

Once the project has been built, the following command can be used to explore all parameters and
subcommands:

```sh
./target/release/node-template -h
```

## Run

The provided `cargo run` command will launch a temporary node and its state will be discarded after
you terminate the process. After the project has been built, there are other ways to launch the
node.

### Single-Node Development Chain

This command will start the single-node development chain with persistent state:

```bash
./target/release/node-template --dev
```

Purge the development chain's state:

```bash
./target/release/node-template purge-chain --dev
```

Start the development chain with detailed logging:

```bash
RUST_LOG=debug RUST_BACKTRACE=1 ./target/release/node-template -lruntime=debug --dev
```

### Connect with Polkadot-JS Apps Front-end

Once the node template is running locally, you can connect it with **Polkadot-JS Apps** front-end
to interact with your chain. [Click here](https://polkadot.js.org/apps/#/explorer?rpc=ws://localhost:9944) connecting the Apps to your local node template.

### Multi-Node Local Testnet

If you want to see the multi-node consensus algorithm in action, refer to
[our Start a Private Network tutorial](https://substrate.dev/docs/en/tutorials/start-a-private-network/).

## Repository Structure

A Substrate project such as this consists of a number of components that are spread across a few
directories.

### Node

A blockchain node is an application that allows users to participate in a blockchain network.
Substrate-based blockchain nodes expose a number of capabilities:

-   Networking: Substrate nodes use the [`libp2p`](https://libp2p.io/) networking stack to allow the
    nodes in the network to communicate with one another.
-   Consensus: Blockchains must have a way to come to
    [consensus](https://substrate.dev/docs/en/knowledgebase/advanced/consensus) on the state of the
    network. Substrate makes it possible to supply custom consensus engines and also ships with
    several consensus mechanisms that have been built on top of
    [Web3 Foundation research](https://research.web3.foundation/en/latest/polkadot/NPoS/index.html).
-   RPC Server: A remote procedure call (RPC) server is used to interact with Substrate nodes.

There are several files in the `node` directory - take special note of the following:

-   [`chain_spec.rs`](./node/src/chain_spec.rs): A
    [chain specification](https://substrate.dev/docs/en/knowledgebase/integrate/chain-spec) is a
    source code file that defines a Substrate chain's initial (genesis) state. Chain specifications
    are useful for development and testing, and critical when architecting the launch of a
    production chain. Take note of the `development_config` and `testnet_genesis` functions, which
    are used to define the genesis state for the local development chain configuration. These
    functions identify some
    [well-known accounts](https://substrate.dev/docs/en/knowledgebase/integrate/subkey#well-known-keys)
    and use them to configure the blockchain's initial state.
-   [`service.rs`](./node/src/service.rs): This file defines the node implementation. Take note of
    the libraries that this file imports and the names of the functions it invokes. In particular,
    there are references to consensus-related topics, such as the
    [longest chain rule](https://substrate.dev/docs/en/knowledgebase/advanced/consensus#longest-chain-rule),
    the [Aura](https://substrate.dev/docs/en/knowledgebase/advanced/consensus#aura) block authoring
    mechanism and the
    [GRANDPA](https://substrate.dev/docs/en/knowledgebase/advanced/consensus#grandpa) finality
    gadget.

After the node has been [built](#build), refer to the embedded documentation to learn more about the
capabilities and configuration parameters that it exposes:

```shell
./target/release/node-template --help
```

### Runtime

In Substrate, the terms
"[runtime](https://substrate.dev/docs/en/knowledgebase/getting-started/glossary#runtime)" and
"[state transition function](https://substrate.dev/docs/en/knowledgebase/getting-started/glossary#stf-state-transition-function)"
are analogous - they refer to the core logic of the blockchain that is responsible for validating
blocks and executing the state changes they define. The Substrate project in this repository uses
the [FRAME](https://substrate.dev/docs/en/knowledgebase/runtime/frame) framework to construct a
blockchain runtime. FRAME allows runtime developers to declare domain-specific logic in modules
called "pallets". At the heart of FRAME is a helpful
[macro language](https://substrate.dev/docs/en/knowledgebase/runtime/macros) that makes it easy to
create pallets and flexibly compose them to create blockchains that can address
[a variety of needs](https://www.substrate.io/substrate-users/).

Review the [FRAME runtime implementation](./runtime/src/lib.rs) included in this template and note
the following:

-   This file configures several pallets to include in the runtime. Each pallet configuration is
    defined by a code block that begins with `impl $PALLET_NAME::Config for Runtime`.
-   The pallets are composed into a single runtime by way of the
    [`construct_runtime!`](https://crates.parity.io/frame_support/macro.construct_runtime.html)
    macro, which is part of the core
    [FRAME Support](https://substrate.dev/docs/en/knowledgebase/runtime/frame#support-library)
    library.

### Pallets

The runtime in this project is constructed using many FRAME pallets that ship with the
[core Substrate repository](https://github.com/paritytech/substrate/tree/master/frame) and a
template pallet that is [defined in the `pallets`](./pallets/template/src/lib.rs) directory.

A FRAME pallet is compromised of a number of blockchain primitives:

-   Storage: FRAME defines a rich set of powerful
    [storage abstractions](https://substrate.dev/docs/en/knowledgebase/runtime/storage) that makes
    it easy to use Substrate's efficient key-value database to manage the evolving state of a
    blockchain.
-   Dispatchables: FRAME pallets define special types of functions that can be invoked (dispatched)
    from outside of the runtime in order to update its state.
-   Events: Substrate uses [events](https://substrate.dev/docs/en/knowledgebase/runtime/events) to
    notify users of important changes in the runtime.
-   Errors: When a dispatchable fails, it returns an error.
-   Config: The `Config` configuration interface is used to define the types and parameters upon
    which a FRAME pallet depends.

### Run in Docker

First, install [Docker](https://docs.docker.com/get-docker/) and
[Docker Compose](https://docs.docker.com/compose/install/).

Then run the following command to start a single node development chain.

```bash
./scripts/docker_run.sh
```

This command will firstly compile your code, and then start a local development network. You can
also replace the default command (`cargo build --release && ./target/release/node-template --dev --ws-external`)
by appending your own. A few useful ones are as follow.

```bash
# Run Substrate node without re-compiling
./scripts/docker_run.sh ./target/release/node-template --dev --ws-external

# Purge the local dev chain
./scripts/docker_run.sh ./target/release/node-template purge-chain --dev

# Check whether the code is compilable
./scripts/docker_run.sh cargo check
```

## Implementation Changes

The following sections relflect the ongoing changes that will define the final Xeno Network implementation and its differences from the base Substrate template.

### Consensus
The current codebase uses Aura and Grandpa for block production and finalization (this combination is useful for testnet implementations). In its final state Xeno NFT Hub will implement a standard Babe and Grandpa combination.

### Staking and Block Rewards

Xeno NFT Hub will implement the standard Substrate bonded staking mechanism, with the following changes:

> - A 21 day period for bonding/unbonding.
> - An emission curve implementation for a fixed supply. XNO is a fixed supply asset which means that staking rewards will not increase the total supply but rather will distribute an alloacated tranche of tokens over an extended schedule.
> - Treasury, Staking and Application rewards will all have independent emission logic.
> - Treasury will not implement a burning rule.

### Governance
Following in the footsteps of the Kusama and Polkadot Networks, Xeno governance uses the [Sudo](https://lib.rs/crates/pallet-sudo "Substrate Sudo Pallet") Module giving the development team elevated permissions. However, once the mainnet implementation launches we will phase in the [Collective](https://lib.rs/crates/pallet-collective "Substrate Collective Pallet"), [Treasury](https://lib.rs/crates/pallet-treasury "Substrate Treasury Pallet"), and [Democracy](https://lib.rs/crates/pallet-democracy "Substrate Democracy Pallet") modules for a more decentralized network and protocol upgrading process.

### Generalized NFTs
Xeno NFT Hub want to have an extensible and gneeralized NFT layer that supports various popular NFT standards such as the Ethereum standards [ERC721](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md "ERC721 Interface Specification"), [ERC1155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md "ERC1155 Interface Specification"), and [Tezos FA2 NFT](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-12/tzip-12.md "FA2 Interface Specification")

Along with the NFT base specification Xeno will implement a genralized Metadata schema specification to track and accomodate a JSON based Metadata structure similar to [TZIP-21](https://gitlab.com/tzip/tzip/-/blob/reserve-tzip-21/proposals/tzip-21/tzip-21.md "Multimedia Metadata Specification") but extendable to accomodate other metadata structures as well.

### Off-Chain Worker Interaction
Xeno NFT Hub will track many parts of the application state in a secure merklized state trasition map. Some state transitions are moderated by the application itself in a deterministic manner, some state changes require one or more user inputs (for example creating an auction, bidding, or account based changes). The Off-Chain State Pallet will accept signed inputs for state changing actions. The module's job is to validate the correctness of generalized off-chain state changes along with validation of the various off-chain worker (and user) signatures to ensure system safety. 

Once a general implementation of this module has been completed, application specific logic for the Xeno Marketplace, NFT Management, and Account Management will be built and extended to allow for application layer functionality to be tracked and integrated in a scalable and efficient manner.

### Parachain Integration

After the project implementation of the above changes are settled, Xeno will migrate its structure to a more live network codebase by integrating with the [Substrate Parachain Implementation](https://github.com/substrate-developer-hub/substrate-parachain-template "Substrate Parachain Template repository") whereafter Xeno NFT Hub will be ready for mainnet relay chain integration.
