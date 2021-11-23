# Help Protocol Core Contracts

This monorepository contains the source code for the core smart contracts implementing Help Protocol on the [Terra Blockchain](https://terra.money).

You can find information about the architecture, usage, and function of the smart contracts on the official Help Protocol documentation [site](https://docs.help.protocol.onchain.engineer/smart-contracts/architecture).

### Dependencies

The Help Protocol token KARMA depends on [Terraswap](https://terraswap.io) and uses its [implementation](https://github.com/terraswap/terraswap) of the CW20 token specification.

## Contracts

| Contract                                            | Reference                                              | Description                                                                                                                        |
| --------------------------------------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| [`help_collector`](./contracts/help_collector)  | [doc](https://docs.help.protocol.onchain.engineer/smart-contracts/collector) | Gathers protocol fees incurred from CDP withdrawals and liquidations and sends to Gov                                              |
| [`help_community`](../contracts/help_community) | [doc](https://docs.help.protocol.onchain.engineer/smart-contracts/community) | Manages the commuinty pool fund                                                                                                    |
| [`help_factory`](./contracts/help_factory)      | [doc](https://docs.help.protocol.onchain.engineer/smart-contracts/factory)   | Central directory that organizes the various component contracts of Help Protocol                                                       |
| [`help_gov`](./contracts/help_gov)              | [doc](https://docs.help.protocol.onchain.engineer/smart-contracts/gov)       | Allows other Help Protocol contracts to be controlled by decentralized governance, distributes KARMA received from Collector to KARMA pledgers |
| [`help_mint`](./contracts/help_mint)            | [doc](https://docs.help.protocol.onchain.engineer/smart-contracts/mint)      | Handles CDP creation, management and liquidation                                                                                   |

## Development

If you have recently created a contract with this template, you probably could use some
help on how to build and test the contract, as well as prepare it for production. This
file attempts to provide a brief overview, assuming you have installed a recent
version of Rust already (eg. 1.51.0+).

## Prerequisites

Before starting, make sure you have [rustup](https://rustup.rs/) along with a
recent `rustc` and `cargo` version installed. Currently, we are testing on 1.51.0+.

And you need to have the `wasm32-unknown-unknown` target installed as well.

You can check that via:

```sh
rustc --version
cargo --version
rustup target list --installed
# if wasm32 is not listed above, run this
rustup target add wasm32-unknown-unknown
```

## Compiling and running tests

Now that you created your custom contract, make sure you can compile and run it before
making any changes. Go into the repository and do:

```sh
# this will produce a wasm build in ./target/wasm32-unknown-unknown/release/YOUR_NAME_HERE.wasm
cargo wasm

# this runs unit tests with helpful backtraces
RUST_BACKTRACE=1 cargo unit-test

# auto-generate json schema
cargo schema
```

### Understanding the tests

The main code is in `src/contract.rs` and the unit tests there run in pure rust,
which makes them very quick to execute and give nice output on failures, especially
if you do `RUST_BACKTRACE=1 cargo unit-test`.

We consider testing critical for anything on a blockchain, and recommend to always keep
the tests up to date.

## Generating JSON Schema

While the Wasm calls (`instantiate`, `execute`, `query`) accept JSON, this is not enough
information to use it. We need to expose the schema for the expected messages to the
clients. You can generate this schema by calling `cargo schema`, which will output
4 files in `./schema`, corresponding to the 3 message types the contract accepts,
as well as the internal `State`.

These files are in standard json-schema format, which should be usable by various
client side tools, either to auto-generate codecs, or just to validate incoming
json wrt. the defined schema.

## Preparing the Wasm bytecode for production

Before we upload it to a chain, we need to ensure the smallest output size possible,
as this will be included in the body of a transaction. We also want to have a
reproducible build process, so third parties can verify that the uploaded Wasm
code did indeed come from the claimed rust code.

To solve both these issues, we have produced `rust-optimizer`, a docker image to
produce an extremely small build output in a consistent manner. The suggest way
to run it is this:

```sh
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/workspace/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/rust-optimizer:0.11.4
```

We must mount the contract code to `/code`. You can use a absolute path instead
of `$(pwd)` if you don't want to `cd` to the directory first. The other two
volumes are nice for speedup. Mounting `/code/target` in particular is useful
to avoid docker overwriting your local dev files with root permissions.
Note the `/code/target` cache is unique for each contract being compiled to limit
interference, while the registry cache is global.

This is rather slow compared to local compilations, especially the first compile
of a given contract. The use of the two volume caches is very useful to speed up
following compiles of the same contract.

This produces an `artifacts` directory with a `PROJECT_NAME.wasm`, as well as
`checksums.txt`, containing the Sha256 hash of the wasm file.
The wasm file is compiled deterministically (anyone else running the same
docker on the same git commit should get the identical file with the same Sha256 hash).
It is also stripped and minimized for upload to a blockchain (we will also
gzip it in the uploading process to make it even smaller).

## License

Copyright 2021 Help Protocol

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0. Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

See the License for the specific language governing permissions and limitations under the License.
