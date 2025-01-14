# Aptos Faucet

The Aptos Faucet is a service that runs alongside a test network and mints coins for users to test and develop on Aptos.

## Subdirectories
This is a brief overview of the subdirectories to help you find what you seek. For more information on each of these subdirectories, see the README in that subdirectory.

- `core/`: All core logic, including the server, endpoint handlers, bypassers, checkers, funders, etc.
- `service/`: The entrypoint for running the faucet as a service.
- `cli/`: CLI for executing the core MintFunder code from the service.
- `metrics-server/`: The metrics server for the faucet service.
- `doc/`: OpenAPI spec generated from the server definition.

In all cases, if a directory holds a crate, the name of that crate is `aptos-faucet-<directory>`. For example the name of the crate in `metrics-server/` is `aptos-faucet-metrics-server`.

## Features

Noteworthy features of the faucet include:
- Checkers, which confirm that the given request is valid. Examples include:
  - IP presence in a blocklist.
  - Auth token.
  - Google Captcha.
- Built in rate limiting, e.g. with a [Redis](https://redis.io/) backend, eliminating the need for something like haproxy in front of the faucet. These are also just checkers.
- Bypassers, the opposite of checkers, which allow requests to bypass checkers and rate limits if they meet some criteria. Examples include:
  - IP presence in an allowlist.
- Different funding backends. Examples include:
  - MintFunder: This works like the legacy faucet. By default, on startup we use the root account to delegate minting capability to a new account and use that to create and mint coins for each fund request.
  - TransferFunder: Each faucet has its own account and uses that to create accounts and transfer funds into them. No minting.
- All of these features are configurable using a config file.

## Running
To run the faucet, the simplest way to start is with this command:
```
cargo run -p aptos-faucet-service -- run-simple --key <private_key> --node-url <api_url> --chain-id TESTING
```

This command lets you configure only a subset of the full functionality of the faucet. You cannot enable any checkers / bypassers, and it supports only the MintFunder. Generally it is intended for use with some kind of local swarm-based testnet or other such uses.

For running the faucet in production, you will instead want to build a configuration file and run it like this:
```
cargo run -p aptos-faucet-service -- run -c <path_to_config_file>
```

You can find many examples of different config files in [configs/](configs/).

## Developing
Certain components of the faucet, e.g. the MinterFunder, rely on a Move script to operate. If you change it, compile the Move script like this (from the root of the repo):
```
cd aptos-move/move-examples/scripts/minter
aptos move compile
```

If you have issues with this, try deleting `~/.move`, updating your Aptos CLI, and changing the AptosFramework version.

Then build the tap as normal (from the root of the repo):
```
cargo build
```

## Testing
If you want to run the tests manually, follow these steps. Note that this is **not necessary** for release safety as the tests are run as part of continuous integration (CI) already.

For the test suite to pass, you must spin up a local testnet, notably without a faucet running (since we're testing the faucet here):
```
cargo run -p aptos -- node run-local-testnet --force-restart --assume-yes
```

You must then copy the mint key for that local testnet to the location the tests expect:
```
cp ~/.aptos/testnet/mint.key /tmp
```

As well as spin up a local Redis 6 ([installation guide](https://redis.io/docs/getting-started/)):
```
redis-server
redis-cli flushall
```

Finally you can run the tests:
```
cargo test -p aptos-faucet-core --features integration-tests
```

## Validating configs
To ensure all the configs are valid, run this:
```
cd crates/aptos-faucet/configs
ls . | xargs -I@ cargo run -p aptos-faucet-service -- validate-config -c @
```

## Generating the specs
From the root:
```
cargo run -- generate-openapi -o doc/spec.yaml -f yaml
cargo run -- generate-openapi -o doc/spec.json -f json
```

## Generating the TS client
From `ts-client`:
```
yarn generate-client
```
