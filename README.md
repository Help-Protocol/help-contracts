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

### Environment Setup

- Rust v1.44.1+
- `wasm32-unknown-unknown` target
- Docker

1. Install `rustup` via https://rustup.rs/

2. Run the following:

```sh
rustup default stable
rustup target add wasm32-unknown-unknown
```

3. Make sure [Docker](https://www.docker.com/) is installed

### Unit / Integration Tests

Each contract contains Rust unit tests embedded within the contract source directories. You can run:

```sh
cargo unit-test
```

### Compiling

After making sure tests pass, you can compile each contract with the following:

```sh
RUSTFLAGS='-C link-arg=-s' cargo wasm
cp ../../target/wasm32-unknown-unknown/release/cw1_subkeys.wasm .
ls -l cw1_subkeys.wasm
sha256sum cw1_subkeys.wasm
```

#### Production

For production builds, run the following:

```sh
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/workspace-optimizer:0.11.3
```

This performs several optimizations which can significantly reduce the final size of the contract binaries, which will be available inside the `artifacts/` directory.

## License

Copyright 2021 Help Protocol

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0. Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

See the License for the specific language governing permissions and limitations under the License.
