# Deploy First WASM Contract

&#x20;

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

To complete this tutorial, you need to:

* Install [rust and cargo](https://www.rust-lang.org/tools/install).
* Be familiar with the Rust programming language.
* Have a general understanding of how the Coreum blockchain works.
* Follow the [instruction](https://docs.coreum.dev/tutorials/cored.html) to install cored binary.
* Install the required util: `jq`.
* Set the [network variables](https://docs.coreum.dev/tutorials/network-variables.html) for the development (testnet is preferable).

### Source Code <a href="#source-code" id="source-code"></a>

The complete source code is located [here](https://github.com/CoreumFoundation/cw-contracts/tree/main/contracts/nameservice).

### Getting Started <a href="#getting-started" id="getting-started"></a>

* Generate a new wallet for testing.

```
cored keys add wallet $COREUM_CHAIN_ID_ARGS
```

* Use the [faucet](https://docs.coreum.dev/tools-ecosystem/faucet.html) to fund your account
* Clone the smart contract template

```
git clone https://github.com/CoreumFoundation/cw-contracts.git
```

* Go to the template directory.

```
cd cw-contracts/contracts/nameservice
```

### Build contract <a href="#build-contract" id="build-contract"></a>

* Build optimized WASM smart contract:

```sh
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/optimizer:0.15.0
```

This operation might take a significant amount of time.

*
  * If you get the following error:

```
error: could not find `Cargo.toml` in `/code` or any parent directory
```

Check if you are in the right directory (should be `cw-contracts/contracts/nameservice`)

### Deploy contract <a href="#deploy-contract" id="deploy-contract"></a>

* List the already deployed contract codes.

```
cored q wasm list-code $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Deploy the built artifact.

```
RES=$(cored tx wasm store artifacts/cw_nameservice.wasm \
    --from wallet --gas auto --gas-adjustment 1.3 -y -b block --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS)
echo $RES    
CODE_ID=$(echo $RES | jq -r '.logs[0].events[-1].attributes[-1].value')
echo $CODE_ID
```

* Check the deployed code.

```
cored q wasm code-info $CODE_ID $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

### Instantiate contract <a href="#instantiate-contract" id="instantiate-contract"></a>

* Instantiating the contract.

```sh
INIT="{\"purchase_price\":{\"amount\":\"100\",\"denom\":\"$COREUM_DENOM\"},\"transfer_price\":{\"amount\":\"999\",\"denom\":\"$COREUM_DENOM\"}}"
cored tx wasm instantiate $CODE_ID "$INIT" --from wallet --label "name service" -b block -y --no-admin $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Check the contract details and account balance.

```sh
cored q wasm list-contract-by-code $CODE_ID --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
CONTRACT_ADDRESS=$(cored q wasm list-contract-by-code $CODE_ID --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS | jq -r '.contracts[-1]')
echo $CONTRACT_ADDRESS
```

### Interact with the contract <a href="#interact-with-the-contract" id="interact-with-the-contract"></a>

* Register a name for the wallet address on the contract.

```shell
REGISTER='{"register":{"name":"fred"}}'
cored tx wasm execute $CONTRACT_ADDRESS "$REGISTER" --amount 100$COREUM_DENOM --from wallet -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Query the owner of the name record.

```
NAME_QUERY='{"resolve_record": {"name": "fred"}}'
cored q wasm contract-state smart $CONTRACT_ADDRESS "$NAME_QUERY" --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

The owner is the "wallet" now.

* Transfer the ownership of the name record to "new-owner" wallet.

```
cored keys add new-owner $COREUM_CHAIN_ID_ARGS
RECIPIENT_ADDRESS=$(cored keys show --address new-owner $COREUM_CHAIN_ID_ARGS)
TRANSFER="{\"transfer\":{\"name\":\"fred\",\"to\":\"$RECIPIENT_ADDRESS\"}}"
cored tx wasm execute $CONTRACT_ADDRESS "$TRANSFER" --amount 999$COREUM_DENOM --from wallet -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Query the record owner again to see the new owner address.

```
echo "Recipient address: $RECIPIENT_ADDRESS"
NAME_QUERY='{"resolve_record": {"name": "fred"}}'
cored q wasm contract-state smart $CONTRACT_ADDRESS "$NAME_QUERY" --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

### Next steps <a href="#next-steps" id="next-steps"></a>

* Read Coreum [modules](https://docs.coreum.dev/modules/main.html) specification, to be familiar with the custom Coreum functionality you can use for your application.
* Read [WASM docs](https://docs.cosmwasm.com/docs/) to understand all supported WASM features.
* Check other [tutorials](https://docs.coreum.dev/tutorials/main.html) to find something you might be interested in additionally.
