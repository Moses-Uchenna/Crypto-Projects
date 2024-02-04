# WebAssembly - WASM

## IBC WASM Transfer Tutorial <a href="#ibc-wasm-transfer-tutorial" id="ibc-wasm-transfer-tutorial"></a>

This tutorial covers building an IBC-enabled smart contract to transfer tokens on Coreum using CosmWasm and Rust.

See full repo here: [https://github.com/CoreumFoundation/ibc-contract-tutorial/ibc-transfer](https://github.com/CoreumFoundation/ibc-contract-tutorial/tree/master/ibc-transfer)

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#prerequisites)Prerequisites <a href="#prerequisites" id="prerequisites"></a>

* Rust development environment
* Coreum node running locally
* Coreum CLI `cored` installed
* Docker

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#contract-structure)Contract Structure <a href="#contract-structure" id="contract-structure"></a>

The main contract code is in `contract.rs`, which uses `msg.rs` that defines our `ExecuteMsg` and `InstantiateMsg`. It also imports utilities from `cosmwasm_std`.

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#core-concepts-in-contract-rs)Core Concepts in `contract.rs` <a href="#core-concepts-in-contract-rs" id="core-concepts-in-contract-rs"></a>

The `contract.rs` file is at the heart of our IBC smart contract. This section will dive into some of the primary structures and functionalities defined within it, providing a granular understanding of the contract's inner workings.

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#imports-and-dependencies)Imports and Dependencies <a href="#imports-and-dependencies" id="imports-and-dependencies"></a>

At the beginning of our contract, we import necessary components. Some of the crucial imports include:

```sh
use cosmwasm_std::{
    to_binary, Binary, Coin, CosmosMsg, Deps, DepsMut, Env, MessageInfo,
    Response, StdResult,
};
```

These imports from `cosmwasm_std` library facilitate the creation, execution, and management of the smart contract on the Coreum blockchain.

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#contract-metadata)Contract Metadata <a href="#contract-metadata" id="contract-metadata"></a>

Every contract should specify its name and version. This information often comes from the `Cargo.toml` file.

```sh
const CONTRACT_NAME: &str = env!("CARGO_PKG_NAME");
const CONTRACT_VERSION: &str = env!("CARGO_PKG_VERSION");
```

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#contract-entry-points)Contract Entry Points <a href="#contract-entry-points" id="contract-entry-points"></a>

Here, we review the primary entry points of our contract:

[**#**](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#\_1-instantiation)**1. Instantiation**

Instantiation is the process by which a smart contract is initialized on the blockchain. This is analogous to deploying a contract on the Ethereum blockchain. The `instantiate` function is the entry point that gets called during this initialization process.

```sh
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(
    deps: DepsMut,
    _env: Env,
    _info: MessageInfo,
    _msg: InstantiateMsg,
) -> StdResult<Response> {...}
```

**Function Parameters**:

* **`deps: DepsMut`**: This is a mutable reference to the contract's dependencies. It provides access to the contract's storage, among other things.
* **`_env: Env`**: Provides information about the blockchain's environment, such as the current block height and time.
* **`_info: MessageInfo`**: Contains information about the incoming transaction, such as the sender's address.
* **`_msg: InstantiateMsg`**: This is a custom data type that carries information specific to the contract's instantiation. It can include configuration parameters or initial data to set up the contract.

**Function Logic**:

Inside the `instantiate` function, the contract version is set using the `set_contract_version` function. This ensures that the current version of the contract is stored within the contract's storage.

Finally, the function returns a `Response` object with an attribute indicating that the "instantiate" method was executed. This is primarily for logging and debugging purposes.

[**#**](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#\_2-execution)**2. Execution**

Once a contract is instantiated on the blockchain, interactions with it are done via transactions. Each of these transactions triggers the `execute` function, which serves as the contract's main processing unit.

```sh
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(
    deps: DepsMut,
    _env: Env,
    _info: MessageInfo,
    msg: ExecuteMsg,
) -> StdResult<Response> {...}
```

**Parameters**:

* The first three parameters (`deps`, `_env`, and `_info`) are similar to the `instantiate` function and serve the same purposes.
* **`msg: ExecuteMsg`**: Represents the specific action or instruction the contract should execute. It's a custom data type that can encapsulate various commands, much like an enum in Rust.

**Logic**:

The `execute` function uses a `match` statement to process the incoming `msg`. In this tutorial, there's only one match arm for the `Transfer` variant of the `ExecuteMsg` enum. When a `Transfer` message is received, the contract calls the `transfer` function to handle the IBC token transfer logic.

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#ibc-token-transfer)IBC Token Transfer <a href="#ibc-token-transfer" id="ibc-token-transfer"></a>

The process of transferring tokens between different blockchains requires a protocol that ensures the secure and reliable exchange of assets. In our contract, this is achieved through the Inter-Blockchain Communication (IBC) protocol. Central to this capability is the `transfer` function.

[**#**](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#function-signature)**Function Signature**

```sh
fn transfer(
    channel_id: String,
    to_address: String,
    amount: Coin,
    timeout: IbcTimeout,
) -> StdResult<Response>
```

[**#**](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#parameters)**Parameters:**

1. **`channel_id: String`** - This represents the unique identifier for the IBC channel. The channel ID is essential for routing the transfer correctly between source and destination blockchains.
2. **`to_address: String`** - This is the recipient's address on the destination blockchain. Upon successful completion of the IBC transfer, the tokens will be credited to this address.
3. **`amount: Coin`** - The `Coin` type encapsulates the token denomination and the amount to be transferred. For instance, if you wanted to send 100 tokens of denomination "atom", the `Coin` object would represent this.
4. **`timeout: IbcTimeout`** - This denotes the time or block height by which the transfer should be completed. If the transfer doesn't occur before this timeout, the transaction is considered failed, and any locked assets on the source chain are released.

[**#**](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#function-logic)**Function Logic:**

The `transfer` function starts by creating an IBC transfer message:

```sh
let ibc_transfer_msg: CosmosMsg = IbcMsg::Transfer {
    channel_id,
    to_address,
    amount,
    timeout,
}
.into();
```

Here, the `IbcMsg::Transfer` structure is populated with the provided parameters. This structure represents an IBC transfer request. Once populated, it's converted into a general `CosmosMsg`, which is a more generic message type used within the CosmWasm framework for different blockchain operations.

Subsequently, a response object is created:

```sh
let res = Response::new()
    .add_attribute("method", "transfer")
    .add_message(ibc_transfer_msg);
```

In this step:

* A new response is initialized using `Response::new()`.
* An attribute "method" with a value "transfer" is added to provide context for loggers or any front-end applications.
* The previously constructed `ibc_transfer_msg` is attached to the response. This ensures that the IBC transfer is processed as part of the contract's execution.

Finally, the populated `Response` object is returned:

```sh
Ok(res)
```

By understanding the `transfer` function in detail, developers can gain insights into the IBC token transfer mechanism within the contract. This function serves as a template for facilitating secure and efficient cross-chain asset transfers. With minor modifications, developers can expand on this foundation, introducing additional features or customizing the logic to better suit specific use cases.

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#unit-tests)Unit Tests <a href="#unit-tests" id="unit-tests"></a>

Unit tests ensure the contract's functionality and correctness. For instance, the `transfer` test verifies the IBC token transfer process:

```sh
#[test]
fn transfer() {...}
```

The `contract.rs` file lays the foundation of our IBC smart contract. By understanding its structures and functionalities, developers can modify, extend, or build upon it to create more complex and feature-rich smart contracts for the Coreum blockchain.

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#makefile-commands)Makefile Commands <a href="#makefile-commands" id="makefile-commands"></a>

The Makefile provides commands for development and testing:

```sh
make dev # Build project
make test # Run tests

make build # Build wasm docker
make deploy # Deploy contract
make instantiate # create a new instance of contract

make get_count # retrive stored count value
make get_timeout_count # retrieve stored timeout count value

channels # get channels
ibc_transfer_cli # issue IBC transfer via cli command
ibc_transfer_wasm_timeout # execute IBC transfer using timestamp timeout

```

## [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#makefile-ibc-commands)Makefile IBC Commands <a href="#makefile-ibc-commands" id="makefile-ibc-commands"></a>

In this section, we'll explore various Makefile commands that interact with the IBC protocol using the `cored` CLI tool. These commands facilitate querying IBC channels, initiating IBC transfers, and making calls to WebAssembly (Wasm) smart contracts with specific timeouts.

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#detailed-command-explanations)Detailed Command Explanations: <a href="#detailed-command-explanations" id="detailed-command-explanations"></a>

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#\_1-channels)1. `channels`: <a href="#id-1-channels" id="id-1-channels"></a>

This command queries the IBC channels.

```sh
channels:
	cored query ibc channel channels $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
```

**Description**:

* `cored query ibc channel channels`: This command is used to query all the IBC channels available.
* `$COREUM_CHAIN_ID_ARGS`: Represents the chain ID arguments for the Coreum blockchain.
* `$COREUM_NODE_ARGS`: Specifies the node-related arguments for the Coreum blockchain.

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#\_2-ibc-transfer-wasm-timeout)2. `ibc_transfer_wasm_timeout`: <a href="#id-2-ibc-transfer-wasm-timeout" id="id-2-ibc-transfer-wasm-timeout"></a>

This command calls a Wasm contract to initiate an IBC transfer with a specific timeout.

`NOTE:` using `9999999000001000000` as a timeout will make your transaction expire on Saturday, November 20, 2286 5:30:00.001 PM. This is just for demo purposes.

```sh
ibc_transfer_wasm_timeout:
	cored tx wasm execute $CONTRACT_ADDRESS "{\"transfer\":{\"channel_id\":\"channel-2\",\"to_address\":\"osmo1pwvcapna75slt3uscvupfe52492yuzhflhakem\",\"amount\":{\"amount\":\"2188\",\"denom\":\"udevcore\"}, \"timeout\": { \"timestamp\": \"9999999000001000000\" } }}" --from $DEV_WALLET -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

**Description**:

* `cored tx wasm execute`: This command is used to execute a Wasm contract.
* `$CONTRACT_ADDRESS`: Represents the address of the Wasm contract to be executed.
* The JSON payload specifies the IBC transfer details and the timeout for the transfer.
* `--from $DEV_WALLET`: Specifies the sender's wallet address.
* `-b block`: Indicates that the transaction should be committed in the next block.
* `-y`: Auto-approves the transaction without prompting for confirmation.

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#instantiating-the-ibc-wasm-contract)Instantiating the IBC Wasm Contract <a href="#instantiating-the-ibc-wasm-contract" id="instantiating-the-ibc-wasm-contract"></a>

Once a smart contract has been compiled, the next step is to deploy it to the blockchain from the wasm artifact we stored previously. This process is generally known as "instantiation". In this section, we will overview the `instantiate` command provided in the Makefile, which is designed to deploy (or instantiate) our IBC Wasm contract on the blockchain.

Instantiate the contract on the chain:

```sh
make instantiate
```

This will initialize a new instance of the contract from the stored wasm file.

#### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#the-instantiate-makefile-command)The `instantiate` Makefile Command: <a href="#the-instantiate-makefile-command" id="the-instantiate-makefile-command"></a>

```sh
instantiate:
	@echo "Instantiating the contract..."
	cored tx wasm instantiate $CODE_ID "{}" \
	--amount="10000000$(COREUM_DENOM)" --no-admin --label "dev test" --from $DEV_WALLET --gas auto --gas-adjustment 1.3 -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

**Command Breakdown**:

1. **`cored tx wasm instantiate`**: This is the base command used to instantiate a Wasm contract on the blockchain using the `cored` CLI tool.
2. **`$CODE_ID`**: Represents the unique identifier of the compiled Wasm contract code that you want to instantiate.
3. **`"{}"`**: This part of the command provides the initialization parameters for the contract. In this case, an empty JSON object is passed, indicating that no specific initialization parameters are required.
4. **`--amount="10000000$(COREUM_DENOM)"`**: Specifies the initial funds to be transferred to the contract upon instantiation. Here, `10000000` units of the `$(COREUM_DENOM)` token are transferred.
5. **`--no-admin`**: This flag indicates that the contract should not have an admin. Without an admin, certain administrative functions (like updating the contract) might be disabled.
6. **`--label "dev test"`**: Provides a label for the contract, which can be useful for identification purposes. In this instance, the label "dev test" is used.
7. **`--from $DEV_WALLET`**: Specifies the wallet address that will be used to pay for the gas fees and provide the initial funds. Here, the `DEV_WALLET` variable holds the address.
8. **`--gas auto`**: This allows the blockchain to automatically estimate the required gas for the transaction.
9. **`--gas-adjustment 1.3`**: Provides a multiplier to the estimated gas. This ensures that if the estimation is slightly off, there's still enough gas provided to complete the transaction.
10. **`-b block`**: Specifies that the transaction will wait until it's included in a block.
11. **`-y`**: This flag confirms that the command should proceed without manual confirmation.
12. **`$COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS`**: These are additional arguments for the Coreum blockchain, specifying details like the node to connect to and the chain ID.

The `instantiate` command is essential for deploying your compiled Wasm contract to the blockchain. Understanding each part of the command ensures you have full control and awareness of the deployment process. Before running this command in a production environment, always double-check the provided parameters, especially the wallet address and the amount of tokens being transferred.

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#execute-messages)Execute Messages <a href="#execute-messages" id="execute-messages"></a>

Execute contract functions:

```sh
make ibc_transfer_wasm_timeout
```

This will send an IBC transfer:

```sh
let ibc_transfer_msg: CosmosMsg = IbcMsg::Transfer {
    channel_id,
    to_address,
    amount,
    timeout,
}
```

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#verifying-successful-ibc-transfer)Verifying Successful IBC Transfer <a href="#verifying-successful-ibc-transfer" id="verifying-successful-ibc-transfer"></a>

Succeful logs will have these messages (seen in explorer.coreum.dev):

```sh
{
    "msg": {
        "transfer": {
            "amount": {
                "denom": "ucore",
                "amount": "1"
            },
            "timeout": {
                "timestamp": "9999999000001000000"
            },
            "channel_id": "channel-2",
            "to_address": "osmo16mwdyj2mmujsf39w0cd82389hlhp82qnzw6fda"
        }
    },
    "@type": "/cosmwasm.wasm.v1.MsgExecuteContract",
    "funds": [],
    "sender": "core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74",
    "contract": "core14lr9zdfn0d5gxjwafh3mg5nrrculj4dndunynve452zws2lzyd3smqrkpz"
}
```

The transaction will also have logs in this format:

```sh
[
    {
        "events": [
            {
                "type": "coin_received",
                "attributes": [
                    {
                        "key": "receiver",
                        "value": "core12k2pyuylm9t7ugdvz67h9pg4gmmvhn5vvgafk0"
                    },
                    {
                        "key": "amount",
                        "value": "1ucore"
                    }
                ]
            },
            {
                "type": "coin_spent",
                "attributes": [
                    {
                        "key": "spender",
                        "value": "core14lr9zdfn0d5gxjwafh3mg5nrrculj4dndunynve452zws2lzyd3smqrkpz"
                    },
                    {
                        "key": "amount",
                        "value": "1ucore"
                    }
                ]
            },
            {
                "type": "execute",
                "attributes": [
                    {
                        "key": "_contract_address",
                        "value": "core14lr9zdfn0d5gxjwafh3mg5nrrculj4dndunynve452zws2lzyd3smqrkpz"
                    }
                ]
            },
            {
                "type": "ibc_transfer",
                "attributes": [
                    {
                        "key": "sender",
                        "value": "core14lr9zdfn0d5gxjwafh3mg5nrrculj4dndunynve452zws2lzyd3smqrkpz"
                    },
                    {
                        "key": "receiver",
                        "value": "osmo16mwdyj2mmujsf39w0cd82389hlhp82qnzw6fda"
                    }
                ]
            },
            {
                "type": "message",
                "attributes": [
                    {
                        "key": "action",
                        "value": "/cosmwasm.wasm.v1.MsgExecuteContract"
                    },
                    {
                        "key": "module",
                        "value": "wasm"
                    },
                    {
                        "key": "sender",
                        "value": "core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74"
                    }
                ]
            },
            {
                "type": "send_packet",
                "attributes": [
                    {
                        "key": "packet_data",
                        "value": "{\"amount\":\"1\",\"denom\":\"ucore\",\"receiver\":\"osmo16mwdyj2mmujsf39w0cd82389hlhp82qnzw6fda\",\"sender\":\"core14lr9zdfn0d5gxjwafh3mg5nrrculj4dndunynve452zws2lzyd3smqrkpz\"}"
                    },
                    {
                        "key": "packet_data_hex",
                        "value": "7b22616d6f756e74223a2231222c2264656e6f6d223a2275636f7265222c227265636569766572223a226f736d6f31366d7764796a326d6d756a73663339773063643832333839686c68703832716e7a7736666461222c2273656e646572223a22636f726531346c72397a64666e30643567786a77616668336d67356e727263756c6a34646e64756e796e76653435327a7773326c7a796433736d71726b707a227d"
                    },
                    {
                        "key": "packet_timeout_height",
                        "value": "0-0"
                    },
                    {
                        "key": "packet_timeout_timestamp",
                        "value": "9999999000001000000"
                    },
                    {
                        "key": "packet_sequence",
                        "value": "18"
                    },
                    {
                        "key": "packet_src_port",
                        "value": "transfer"
                    },
                    {
                        "key": "packet_src_channel",
                        "value": "channel-2"
                    },
                    {
                        "key": "packet_dst_port",
                        "value": "transfer"
                    },
                    {
                        "key": "packet_dst_channel",
                        "value": "channel-2188"
                    },
                    {
                        "key": "packet_channel_ordering",
                        "value": "ORDER_UNORDERED"
                    },
                    {
                        "key": "packet_connection",
                        "value": "connection-3"
                    }
                ]
            },
            {
                "type": "transfer",
                "attributes": [
                    {
                        "key": "recipient",
                        "value": "core12k2pyuylm9t7ugdvz67h9pg4gmmvhn5vvgafk0"
                    },
                    {
                        "key": "sender",
                        "value": "core14lr9zdfn0d5gxjwafh3mg5nrrculj4dndunynve452zws2lzyd3smqrkpz"
                    },
                    {
                        "key": "amount",
                        "value": "1ucore"
                    }
                ]
            },
            {
                "type": "wasm",
                "attributes": [
                    {
                        "key": "_contract_address",
                        "value": "core14lr9zdfn0d5gxjwafh3mg5nrrculj4dndunynve452zws2lzyd3smqrkpz"
                    },
                    {
                        "key": "method",
                        "value": "transfer"
                    }
                ]
            }
        ]
    }
]
```

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-wasm-transfer.html#conclusion)Conclusion <a href="#conclusion" id="conclusion"></a>

This covers a basic IBC transfer contract flow.

Last Updated: 12/14/2023, 3:39:40 PM

[ IBC Smart Contract Call Tutorial](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-contracts.html)[IBC Transfer Using CLI ](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-cli.html)

\
