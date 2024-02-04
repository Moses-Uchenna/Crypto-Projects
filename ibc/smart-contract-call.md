# Smart Contract Call

## IBC Smart Contract Call Tutorial <a href="#ibc-smart-contract-call-tutorial" id="ibc-smart-contract-call-tutorial"></a>

This tutorial showcases a simple CosmWasm Smart Contract that will communicate with the same Smart Contract deployed on a different chain. These contracts will 'ping-pong' messages to each other incrementing a counter stored on each contract.

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

To complete this tutorial, you need to:

* Install [rust and cargo](https://www.rust-lang.org/tools/install).
* Be familiar with the Rust programming language.
* Have a general understanding of how the Coreum blockchain works.
* Follow the [instruction](https://docs.coreum.dev/tutorials/cored.html) to install cored binary.
* Set the [network variables](https://docs.coreum.dev/tutorials/network-variables.html). We will use both the mainnet and the testnet.
* Have a general understanding of [IBC](https://ibc.cosmos.network/main/ibc/overview.html)and the [Hermes Relayer](https://hermes.informal.systems/index.html).

### Source Code <a href="#source-code" id="source-code"></a>

The complete source code is located [here](https://github.com/CoreumFoundation/ibc-contract-tutorial/tree/master/ibc-call).

### IBC Channel Overview <a href="#ibc-channel-overview" id="ibc-channel-overview"></a>

To connect our Smart Contracts using IBC we must first create a channel between them. Channels are established through a four-way handshake, in which each step is initiated by a relayer:

1. `ChanOpenInit`: Will set the chain A into `INIT` state. This will call `OnChanOpenInit` so application A can apply the custom callback that it has set on `INIT`, e.g. check if the port has been set correctly, the channel is indeed unordered/ordered as expected, etc. An application version is also proposed in this step.
2. `ChanOpenTry`: Will set chain B into `TRY` state. It will call `OnChanOpenTry` so application B can apply its custom `TRY` callback. Application version negotiation also happens during this step.
3. `ChanOpenAck`: Will set the chain A into `OPEN` state. This will call `OnChanOpenAck` which will be implemented by the application. Application version negotiation is finalised during this step.
4. `ChanOpenConfirm`: Will set chain B into `OPEN` state so application B can apply its `CONFIRM` logic.

Once the handshake has been completed, a channel will created to send IBC messages. To be able to perform this handshake and receive IBC messages our contract must implement the ibc entry points. We define them in [ibc.rs](https://github.com/CoreumFoundation/ibc-contract-tutorial/tree/master/ibc-call/src/ibc.rs):

1. `ibc_channel_open`: Entry point to handle IBC channel opening handshake phases: `OpenInit` and `OpenTry`.
2. `ibc_channel_connect`: Entry point to handle IBC channel connection phase: `OpenAck` and `OpenConfirm`.
3. `ibc_channel_close`: Entry point to handle IBC channel closure. A channel is closed by the counterparty and can be closed during the exceptional circumstances: packet timeout on an ordered channel, 66% takeover by malicious validators or due to a light client attack where the compromised chain can fool the counterparty into thinking that it has closed its end of the channel.
4. `ibc_packet_receive`: Entry point for receiving our IBC messages (packets).
5. `ibc_packet_ack`: Entry point for handling IBC packet acknowledgements.
6. `ibc_packet_timeout`: Entry point for handling IBC packet timeouts.

Once these entry points are defined, when we instantiate our contract, it will be assigned an IBC port. This port will be used as an end-point of the channel that the relayer will create.

### Set up contracts <a href="#set-up-contracts" id="set-up-contracts"></a>

* Build the smart contract

{% code lineNumbers="true" %}
```sh
git clone git@github.com:CoreumFoundation/ibc-contract-tutorial.git
cd ibc-contract-tutorial/ibc-call
make build
```
{% endcode %}

* Deploy the smart contract on two different chains.

We are going to use our Coreum testnet and devnet for the sake of this tutorial, but in a real App we would probably want to use two different chain's mainnets. On each chain:

{% code lineNumbers="true" %}
```sh
RES=$(cored tx wasm store artifacts/ibc_tutorial.wasm \
    --from wallet --gas auto --gas-adjustment 1.3 -y -b block --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS)
echo $RES
CODE_ID=$(echo $RES | jq -r '.logs[0].events[-1].attributes[-1].value')
echo $CODE_ID
```
{% endcode %}

* Instantiate the contract on each chain.

```sh
cored tx wasm instantiate $CODE_ID '{}' --from wallet --label "ibc-contract" -b block -y --no-admin $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Capture the contract address.

{% code lineNumbers="true" %}
```sh
CONTRACT_ADDRESS=$(cored q wasm list-contract-by-code $CODE_ID --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS | jq -r '.contracts[-1]')
echo "Contract address: $CONTRACT_ADDRESS"
```
{% endcode %}

* Obtain IBC port of contract:

{% code lineNumbers="true" %}
```sh
IBC_PORT=$(cored q wasm contract $CONTRACT_ADDRESS --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS | jq -r '.contract_info.ibc_port_id')
echo "IBC Port: $IBC_PORT"
```
{% endcode %}

Example output:

```sh
IBC Port: wasm.testcore120dn2cr7tqnvup0p6qv2gft5zyjuh8nqhjdzyytc0xapcm08hmzsyv6kd6
```

This value is the IBC port of the contract and is what we are going to use as an endpoint of the IBC channel that we are going to create with our relayer. We will need to provide the two IBC ports, one on each chain.

### Set up Relayer <a href="#set-up-relayer" id="set-up-relayer"></a>

To establish a connection between the two contracts you need to set up a relayer. First, we will need to [configure Hermesopen in new window](https://hermes.informal.systems/documentation/configuration/description.html). After configuring Hermes, we can create a channel to establish the IBC connection between the contracts:

```rust
hermes create channel --a-chain coreum-testnet-1 --b-chain coreum-devnet-1 --a-port wasm.testcore120dn2cr7tqnvup0p6qv2gft5zyjuh8nqhjdzyytc0xapcm08hmzsyv6kd6 --b-port wasm.devcore1u8qeahf3aql7xzx25lamtwafrrc63khtwwsg32t9x8azaqa3p6zs2nsekz --channel-version counter-1
```

From this command, we can see that we create a channel between the contracts deployed on our coreum testnet and our coreum devnet (using their IBC ports). The IBC version argument is defined in our contract(in `ibc.rs`):

{% code lineNumbers="true" %}
```rust
// Define the version for IBC
pub const IBC_VERSION: &str = "counter-1";
```
{% endcode %}

Now that the channel is created, it's time to start our relayer:

```sh
hermes start
```

Once the relayer is running, take note of the channel IDs that have been established. We will use this information in our contracts ExecuteMsg to send the packets to the right place. This is necessary because contracts can have multiple channels connected to their port, so we need to provide channel information for our packets.

### [#](https://docs.coreum.dev/tutorials/IBC/ibc-tutorial-contracts.html#execute-contracts)Execute contracts <a href="#execute-contracts" id="execute-contracts"></a>

Let's assume that the channel IDs we got from Hermes are channel-2105 and channel-82 (These numbers will be different for you when you create the channel).

We are going to send a packet from the contract in our testnet to the contract in our devnet over IBC.

{% code lineNumbers="true" %}
```rust
INCREMENT='{"increment": { "channel": "channel-2105" }}'
cored tx wasm execute $CONTRACT_ADDRESS "$INCREMENT" --from wallet --gas auto --gas-adjustment 1.3 -y -b block --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```
{% endcode %}

NOTE: Remember that packets may take a while to be relayed, so we might not see the updated value instantly on the other contract. Additionally, if the relayer was not running, the packets will not be taken by the relayer either.

Let's query the value of the counter on our other contract:

{% code lineNumbers="true" %}
```rust
QUERY='{"get_count": {"channel": "channel-82"}}'
cored q wasm contract-state smart devcore1u8qeahf3aql7xzx25lamtwafrrc63khtwwsg32t9x8azaqa3p6zs2nsekz "$QUERY" --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```
{% endcode %}

Example output:

{% code lineNumbers="true" %}
```rust
data:
  count: 1
```
{% endcode %}

As you can see, the packet was successfully received by the contract in the other chain and the value of the counter was successfully updated. We could do the same as above but in the opposite direction, using the increment on the devnet contract using channel-82 and then querying the count on the testnet contract using channel-2105.

### Conclusion <a href="#conclusion" id="conclusion"></a>

This tutorial provided an easy example of a contract-to-contract IBC communication. It's important to point out that even though we used the same contract on both chains to keep it simple, this is not a requirement. You can have different contracts with different ExecuteMsgs defined and different ways of handling each message in the `ibc_packet_receive` entry point.

In our contract we define:

{% code lineNumbers="true" %}
```rust
// Entry point for receiving IBC packets
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn ibc_packet_receive(
    deps: DepsMut,
    env: Env,
    msg: IbcPacketReceiveMsg,
) -> Result<IbcReceiveResponse, Never> {
    // Handle the packet and ensure we always ACK regardless of success or failure
    match do_ibc_packet_receive(deps, env, msg) {
        Ok(response) => Ok(response),
        Err(error) => Ok(IbcReceiveResponse::new()
            .add_attribute("method", "ibc_packet_receive")
            .add_attribute("error", error.to_string())
            .set_ack(make_ack_fail(error.to_string()))),
    }
}

// Inner logic for handling packet reception
pub fn do_ibc_packet_receive(
    deps: DepsMut,
    _env: Env,
    msg: IbcPacketReceiveMsg,
) -> Result<IbcReceiveResponse, ContractError> {
    let channel = msg.packet.dest.channel_id;
    let msg: IbcExecuteMsg = from_binary(&msg.packet.data)?;

    match msg {
        IbcExecuteMsg::Increment {} => execute_increment(deps, channel),
    }
}
```
{% endcode %}

As you can see, we only have 1 msg defined in our contract (`Increment`), but we can extend this to process as many IBC packets (messages) that our cross-chain application needs.

Additionally, the contracts can be different as long as the IBC logic is defined correctly (packets sent from one contract must be able to be processed by the counterparty).
