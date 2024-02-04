# Command Line Interface - CLI

## IBC Transfer Using CLI <a href="#ibc-transfer-using-cli" id="ibc-transfer-using-cli"></a>

Inter-Blockchain Communication (IBC) is a protocol that allows different blockchains to communicate with each other, enabling the transfer of tokens and other data across chains. In this guide, we'll dive into how to use the CLI to test IBC transfers, specifically between the Coreum and Osmosis chains.

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

1. **Install the `cored` and `osmosisd` CLI tools**: Ensure that you have both CLI tools installed and accessible from your terminal.
2. **Configure your CLI**: Make sure that you've set up your CLI with the necessary chain configurations, keys, and other required parameters.

### Tutorial <a href="#tutorial" id="tutorial"></a>

#### 1. Sending Tokens to Osmosis from Coreum <a href="#id-1-sending-tokens-to-osmosis-from-coreum" id="id-1-sending-tokens-to-osmosis-from-coreum"></a>

To initiate an IBC transfer from Coreum to Osmosis, use the `cored` CLI tool. Replace `[your-osmosis-address]` with the target Osmosis address and `[amount-in-ucore]` with the amount you wish to send (in `ucore` denomination).

```sh
cored tx ibc-transfer transfer transfer [channel-id] [your-osmosis-address] [amount-in-ucore]
```

**Example**: Let's say you want to send `1000ucore` to the Osmosis address `osmo16q5ca0kz5tl0arxnt4ynzyk5xs5tq24lfrywnx`:

```sh
cored tx ibc-transfer transfer transfer channel-2 osmo16q5ca0kz5tl0arxnt4ynzyk5xs5tq24lfrywnx 1000ucore --from dev-wallet $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
```

#### 2. Sending Tokens back to Coreum from Osmosis <a href="#id-2-sending-tokens-back-to-coreum-from-osmosis" id="id-2-sending-tokens-back-to-coreum-from-osmosis"></a>

To transfer tokens from Osmosis back to Coreum, utilize the `osmosisd` CLI tool. You'll need to specify the token denomination, amount, receiving Coreum address, and other required arguments.

```sh
osmosisd tx ibc-transfer transfer transfer channel-2188 core1msa5mwyvjqlc4nj4ym2q8nqrs0dq9t6nx27mu7 1000ibc/927661F31AA9C5801D58104292A35053097B393CFFA0D9B6CB450A3D66D747FA --fees 875uosmo --from test --chain-id=osmosis-1 --node=https://rpc.osmosis.zone:443
```

In this command:

* `channel-2188` is the IBC channel ID used for the transfer back to Coreum.
* `core1msa5mwyvjqlc4nj4ym2q8nqrs0dq9t6nx27mu7` is the target Coreum address.
* `1000ibc/927661F31AA9C5801D58104292A35053097B393CFFA0D9B6CB450A3D66D747FA` represents the amount and denomination of the token.
* `--fees 875uosmo` specifies the transaction fee in Osmosis tokens.

**Understanding and Generating the IBC Token Denomination (denom)**

In the context of IBC transfers, the token denomination (`denom`) format appears a bit unconventional compared to what most blockchain developers might be accustomed to. Let's understand why it's structured this way and how to determine the correct `denom` for your transfers.

**The IBC Token Denomination Format**

The denom for IBC transfers has a specific format: `ibc/HASH`. Here:

* `ibc/`: It's a prefix indicating that the token follows the IBC token standards.
* `HASH`: It's a SHA-256 hash of the IBC path, typically given in the format `ibc-port/ibc-channel/native-denom`.

The reason behind this hashed format is to ensure a unique identifier for tokens across different chains, preventing any naming conflicts or ambiguities. Since tokens can be sent across multiple chains, each with its native denominations, the IBC protocol adopts this hashing mechanism to generate a unique identifier for every token on every chain.

**Generating the IBC Token Denomination**

The following script can be used to derive the denomination format:

{% code lineNumbers="true" %}
```sh
#!/bin/sh
# Run with "ibc-port/ibc-channel/native-denom" as an argument
IBCD="$1"
echo -n $IBCD | openssl dgst -sha256 | awk '{print "ibc/" toupper($2)}'
```
{% endcode %}

To use the script:

1. Save the above script to a file, e.g., `get_ibc_denom.sh`.
2. Give execute permissions to the script: `chmod +x get_ibc_denom.sh`.
3. Run the script with the appropriate argument: `./get_ibc_denom.sh ibc-port/ibc-channel/native-denom`.

The script will output the correct `denom` for your IBC transfer based on the provided IBC path.

By understanding this unique denom structure and having the means to generate it, you ensure the correct identification of tokens across IBC-enabled blockchains, facilitating smooth and error-free transfers.

## Conclusion <a href="#conclusion" id="conclusion"></a>

You've now learned how to use the CLI to test IBC transfers between Coreum and Osmosis. Remember to double-check addresses, amounts, and other parameters before initiating transfers to ensure successful transactions.
