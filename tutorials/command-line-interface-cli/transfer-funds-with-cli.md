# Transfer Funds with CLI

The section provides a tutorial on how to use `cored` cli. You can follow this flow with any other cli command.

**Note:** we are going to use testnet for this example.

* [Install cored binary](https://docs.coreum.dev/tutorials/cored.html)
* Point`cored` binary to testnet network:

{% code lineNumbers="true" %}
```sh
export CORED_NODE=https://full-node.testnet-1.coreum.dev:26657 
export CHAIN_ID="coreum-testnet-1"
```
{% endcode %}

If you want to target another network than testnet, replace it with values on the [network variables page](https://docs.coreum.dev/tutorials/network-variables.html)

* To check verify that everything is set correctly, you can run `cored status | jq` and check network name. If you don't have jq installed you can run `cored status`, but we strongly recommend using this awesome tool 😃
*   You should have funded account. If you don't have it, do next:

    * Go to [faucet page](https://docs.coreum.dev/tools-ecosystem/faucet.html) and click `Generate Funded Wallet`.
    * Copy `Wallet Mnemonic` and go to your terminal and run:

    {% code lineNumbers="true" %}
    ```sh
    cored keys add my-sender-wallet --recover --chain-id=$CHAIN_ID
    # Where `my-sender-wallet` is your local account name. It will be used later.
    ```
    {% endcode %}
* Enter your mnemonic. Now you imported account into your local machine!
* Store the mnemonic in safe place, this is the only way to recover your account.

`bank` module is responsible for transferring funds:

{% code lineNumbers="true" %}
````sh
```bash
cored tx bank --help
```
````
{% endcode %}

*   There is only one available command called `send`. Let's check its usage:

    ```sh
    cored tx bank send --help
    # output:
    # cored tx bank send [from_key_or_address] [to_address] [amount] [flags]
    ```

Full command should look like this:

```sh
cored tx bank send my-sender-wallet testcore1snn05vrzvnwy7t0g00rr7hva63hmwxuuv7nrj0 1000000utestcore --node=$CORED_NODE --chain-id=$CHAIN_ID
# my-sender-wallet is your local account name, which can be replaced by an address.
# 1000000utestcore is equal to 1testcore
```

If the output code is 0, get the transaction hash, go to [Block Explorer](https://docs.coreum.dev/tools-ecosystem/block-explorer.html) and put it into the search line.

If output code is not 0, your transaction failed local validation and was not broadcasted. Fix the problem and run the command again.

### Send staking tx with CLI <a href="#send-staking-tx-with-cli" id="send-staking-tx-with-cli"></a>

*   To stake your tokens with CLI you should use `staking` module. Let's check available commands:

    ```sh
    cored tx staking --help
    # create-validator create new validator initialized with a self-delegation to it
    # delegate         Delegate liquid tokens to a validator
    # edit-validator   edit an existing validator account
    # redelegate       Redelegate illiquid tokens from one validator to another
    # unbond           Unbond shares from a validator
    ```

Let's delegate some tokens:

```sh
cored tx bank send my-sender-wallet testcore1snn05vrzvnwy7t0g00rr7hva63hmwxuuv7nrj0 1000000utestcore --node=$CORED_NODE --chain-id=$CHAIN_ID
# my-sender-wallet is your local account name, which can be replaced by an address.
# 1000000utestcore is equal to 1testcore
```

*   If the output code is 0, get the transaction hash, go to [Block Explorer](https://docs.coreum.dev/tools-ecosystem/block-explorer.html) and put it into the search line.

    If output code is not 0, your transaction failed local validation and was not broadcasted. Fix the problem and run the command again.

### &#x20;<a href="#send-staking-tx-with-cli-1" id="send-staking-tx-with-cli-1"></a>

### Send staking tx with CLI <a href="#send-staking-tx-with-cli-1" id="send-staking-tx-with-cli-1"></a>

*   To stake your tokens with CLI you should use `staking` module. Let's check available commands:

    ```sh
    cored tx staking --help
    # create-validator create new validator initialized with a self-delegation to it
    # delegate         Delegate liquid tokens to a validator
    # edit-validator   edit an existing validator account
    # redelegate       Redelegate illiquid tokens from one validator to another
    # unbond           Unbond shares from a validator
    ```

Let's delegate some tokens:

```sh
# cored tx staking delegate [validator-addr] [amount] --from [delegator-addr] [flags]
cored tx staking delegate testcorevaloper14x4ux30sadvg90k2xd8fte5vnhhh0uvkxf4rgm 1000000utestcore --from testcore1q07ldrjnr8xtsy3rz82yxqcdrffu3uw3daslrw --node=$CORED_NODE --chain-id=$CHAIN_ID
# 1000000utestcore is equal to 1testcore
```
