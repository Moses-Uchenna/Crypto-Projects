# Create and Manage FTs

## Create Fungible Token with CLI <a href="#create-fungible-token-with-cli" id="create-fungible-token-with-cli"></a>

This tutorial will guide you through the process of using the [asset FT](https://docs.coreum.dev/modules/assetft.html) module to create and manage Fungible Tokens.

Please note that each subsequent section depends on the previous one.

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

* Since we are going to use the Testnet network in this tutorial, let's export its variables:

```
# CHAIN_ID we are going to use as a value for the --chain-id flag to target specific network
export CHAIN_ID="coreum-testnet-1"
# RPC_URL we are going to use as a value for the --node flag to target specific node within the given network
export RPC_URL="https://full-node.testnet-1.coreum.dev:26657"
```

\* If you want to use other network, find CHAIN\_ID and RPC\_URL values on the [network variables](https://docs.coreum.dev/tutorials/network-variables.html) page.

* [Install cored binary](https://docs.coreum.dev/tutorials/cored.html)
*   You should have two accounts on your local keychain. If you do not have them, you can go to the [faucet page](https://docs.coreum.dev/tools-ecosystem/faucet.html) and generate funded accounts for the testnet. Once you have their mnemonics, import them by running the following command:

    ```
    cored keys add ft-issuer --recover --chain-id=$CHAIN_ID
    # put the first mnemonic here
    cored keys add ft-receiver-1 --recover --chain-id=$CHAIN_ID
    # put the second mnemonic here
    ```

    \* If you already have accounts, you can bind them to environment variables in the next section.

    \* You may skip the `--chain-id` flag, but the output will be printed for the mainnet network as a default one.
*   Export commonly used, throughout the tutorial, environment variables.

    ```
    # The keyring holds the private/public keypairs used to interact with a node.
    # The private key can be stored in different locations, called "backends".
    # Available values for cored are os|file|kwallet|pass|test|memory. We are going to use "os".
    export KEYRING_BACKEND=os

    # Bank send requires a raw address as a recipient, let's export it to the environment variables:
    export FT_ISSUER=$(cored keys show ft-issuer --address --keyring-backend=$KEYRING_BACKEND --chain-id=$CHAIN_ID)
    export FT_RECEIVER_1=$(cored keys show ft-receiver-1 --address --keyring-backend=$KEYRING_BACKEND --chain-id=$CHAIN_ID)
    ```

### Issue your first FT <a href="#issue-your-first-ft" id="issue-your-first-ft"></a>

* Use the following command to issue your first FT:

```
  # cored tx assetft issue [symbol] [subunit] [precision] [initial_amount] [description] --uri --uri_hash --from [issuer] --features=burning,freezing,minting,whitelisting --burn-rate=0.12 --send-commission-rate=0.2 [flags]
  cored tx assetft issue MYFT cmyft 2 100 "My first FT token"  --uri https://my-token-meta.invalid/1 --uri_hash e000624 --from $FT_ISSUER --features=burning,freezing,minting,whitelisting --send-commission-rate=0.02 --node=$RPC_URL --chain-id=$CHAIN_ID
  # where:
  # MYFT - token symbol, is the display name of a token used mostly for UI purposes. An example of a token symbol is BTC.
  # cmyft - subunit, it is used to construct the denom of an FT. All on-chain operations are made using subunits. An example of a subunit is satoshi.
  # 2 - precision(decimal count, so 100cmyft=1myft)
  # 100 - the initial amount of cmyft
  # uri - Uniform Resource Identifier(example: https://my-token-meta.invalid/1)(flag is optional)
  # uri_hash - example: e000624 (flag is optional)
  # --features flag is optional
```

<details>

<summary>"c" prefix in subunit name stays for `centi`, which means 10^(-2). Expand this section to see more details.</summary>

<img src="https://docs.coreum.dev/assets/submultiples_prefixes.8b21cb2c.png" alt="Figure 1 - Smart Tokens" data-size="original">

</details>

The output of the command should provide a transaction hash. Copy this hash and go to the [Block explorer](https://docs.coreum.dev/tools-ecosystem/block-explorer.html) to see the transaction's status. Remember that your transaction may appear in block explorer with some delay since it should be indexed first.

Please note that you can only issue one unique FT within one account address.

* Your new token has a unique denom that consists of a subunit and your account address.
*   Let's export this denom to an environment variable for further use:

    ```
    export FT_DENOM=cmyft-$FT_ISSUER
    ```

    Let's check our FT balance:

    ```
    cored q bank balances $FT_ISSUER --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # You should see the output similar to this:
    # amount: "100"
    # denom: cmyft-testcore1z0f5qlw5k90qn0ll5m6d7k8802j3qntylnt6mv
    ```
*   Let's retrieve FT details. Obtain your full token denom (including the account address) and replace it with the following command:

    ```
    cored q assetft token $FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # token:
    # burn_rate: "0.000000000000000000"
    # denom: cmyft-testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7
    # description: My first FT token
    # features:
    # - burning
    # - freezing
    # - minting
    # - whitelisting
    # globally_frozen: false
    # issuer: testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7
    # precision: 2
    # send_commission_rate: "0.020000000000000000"
    # subunit: cmyft
    # symbol: MYFT
    # uri: https://my-token-meta.invalid/1
    # uri_hash: e000624
    ```

### Minting <a href="#minting" id="minting"></a>

*   Since we have enabled the `minting` feature, the token issuer can mint additional tokens by following these steps:

    ```
    # You should use the issuer's address for this command, otherwise you will get an unauthorized error
    cored tx assetft mint 100$FT_DENOM --from $FT_ISSUER --node=$RPC_URL --chain-id=$CHAIN_ID
    # Let's check the balance:
    cored q bank balances $FT_ISSUER --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # amount: "200"
    # denom: cmyft-testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7

    ```

    Note: minting is not affected by the global freeze. You can also check the total supply using this command:

    ```
    cored q bank total --denom=$FT_DENOM --node=$RPC_URL
    # amount: "200"
    # denom: cmyft-testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7
    ```

### Whitelisting and bank send <a href="#whitelisting-and-bank-send" id="whitelisting-and-bank-send"></a>

*   Since we have enabled the `whitelisting` feature, you should whitelist the account before sending tokens, otherwise, the transaction will fail with the following error `0: balance whitelisted error`. Here's how to do it:

    ```
    # cored tx assetft set-whitelisted-limit [account_address] [amount] --from [sender] [flags]
    cored tx assetft set-whitelisted-limit $FT_RECEIVER_1 1000$FT_DENOM --from $FT_ISSUER --node=$RPC_URL --chain-id=$CHAIN_ID

    # Now we can check the account's whitelist balance:
    cored q assetft whitelisted-balance $FT_RECEIVER_1 $FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # balance:
    # amount: "1000"
    # denom: cmyft-testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7
    ```

    Note that each set-whitelisted-limit command overrides the existing whitelisting limit settings. You can whitelist more tokens than issued.
*   Send tokens to another account:

    ```
    cored tx bank send $FT_ISSUER $FT_RECEIVER_1 200$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID

    # Let's check if we received tokens:
    cored q bank balances $FT_RECEIVER_1 --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # amount: "200"
    ```

### Freezing <a href="#freezing" id="freezing"></a>

Freezing is the process of locking up a Fungible Token on a specific account or globally. This can be done using two sets of commands: `freeze` and `globally-freeze`.

The `freeze` command can be used to freeze any amount for a specific account, even if the amount is greater than the account's balance. The opposite command is `unfreeze`, which unfreezes the specified amount of frozen tokens.

The `globally-freeze` command is used to freeze all Fungible Tokens so that no operations could be performed on them by anyone except the issuer until they are unfrozen using the `globally-unfreeze` command.

#### Freeze <a href="#freeze" id="freeze"></a>

Here's an example of using the `freeze` command:

```
# Usage: cored tx assetft freeze [account_address] [amount] --from [sender] [flags]
cored tx assetft freeze $FT_RECEIVER_1 100$FT_DENOM --from $FT_ISSUER  --node=$RPC_URL --chain-id=$CHAIN_ID
```

In this example, we are freezing 100 tokens for the account $FT\_RECEIVER\_1. Note that we can freeze any amount for an account, even if it is greater than the account's balance.

If we try to send frozen tokens, we will receive an error message indicating that the tokens are not available for sending:

```
cored tx bank send $FT_RECEIVER_1 $FT_ISSUER 101$FT_DENOM  --node=$RPC_URL --chain-id=$CHAIN_ID
# failed to execute message;
# message index: 0: 101cmyft-testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7 is not available,
# available 100cmyft-testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7: insufficient funds
```

Here, we are trying to send 101 tokens, but we can only spend 100(the rest exceeds the applied freezing limit).

We can check the number of frozen tokens for a specific account using the `frozen-balance` command:

```
# Usage: cored q assetft frozen-balance [account] [denom] [flags]
cored q assetft frozen-balance $FT_RECEIVER_1 $FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
# balance:
#  amount: "100"
#  denom: cmyft-testcore1m4mm44zh9unlpg74nnqxmvwrkm5a20tmaes2z7
```

In this example, we can see that the frozen balance for $FT\_RECEIVER\_1 is 100 tokens.

To unfreeze tokens, we can use the `unfreeze` command:

```
cored tx assetft unfreeze $FT_RECEIVER_1 100$FT_DENOM --from $FT_ISSUER  --node=$RPC_URL --chain-id=$CHAIN_ID

# Let's check the frozen balance
cored q assetft frozen-balance $FT_RECEIVER_1 $FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
# amount: "0"
```

We can see that the frozen balance is now 0. Note that we cannot unfreeze more tokens than the account has frozen.

#### Globally-freeze <a href="#globally-freeze" id="globally-freeze"></a>

To globally freeze tokens, you can use the following command:

```
cored tx assetft globally-freeze $FT_DENOM --from $FT_ISSUER  --node=$RPC_URL --chain-id=$CHAIN_ID
```

Once this command is executed, all token operations will be disabled, including sending tokens on behalf of the common account. Even sending tokens to the issuer will not be possible, as the coins will be marked as not spendable.

To illustrate this, you can try sending tokens using the following command:

```
cored tx bank send $FT_RECEIVER_1 $FT_ISSUER 100$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
# On the explorer page you can see the following output:
# coins are not spendable: cmyft-testcore1f2dyj8dhdv62ytrkuvn832ezzjdcpg2jhrtzvy is globally frozen
```

This command will not work, and you will receive an error message stating that the coins are globally frozen.

To unfreeze the tokens and resume normal operations, you can use the following command:

```
cored tx assetft globally-unfreeze $FT_DENOM --from $FT_ISSUER --node=$RPC_URL --chain-id=$CHAIN_ID
```

Note that even after the global freeze, the issuer of the tokens will still be able to perform operations with their tokens.

### Send-Commission-Rate <a href="#send-commission-rate" id="send-commission-rate"></a>

The send-commission-rate determines the commission applied to tokens transferred between accounts, except when one of the participants is the token issuer. To illustrate how this works, follow the steps below:

*   First, we need a third account to transfer tokens to, let's [generate](https://docs.coreum.dev/tools-ecosystem/faucet.html) it and prepare:

    ```
    # import the third account
    cored keys add ft-receiver-2 --recover --chain-id=$CHAIN_ID
    # export the third account address to an environment variable
    export FT_RECEIVER_2=$(cored keys show ft-receiver-2 --address --keyring-backend=$KEYRING_BACKEND --chain-id=$CHAIN_ID)
    # whitelist the third account
    cored tx assetft set-whitelisted-limit $FT_RECEIVER_2 200$FT_DENOM --from $FT_ISSUER --node=$RPC_URL --chain-id=$CHAIN_ID
    ```
*   Next, check the initial balances of the sender and issuer using the following commands:

    ```
    cored q bank balances $FT_RECEIVER_1 --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # amount: "200"

    cored q bank balances $FT_ISSUER --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # amount: "0"
    ```
*   Afterward, send 100 tokens from the sender to the third account using the following command:

    ```
    cored tx bank send $FT_RECEIVER_1 $FT_RECEIVER_2 100$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    ```
*   Let's check account balances:

    ```
    cored q bank balances $FT_RECEIVER_1 --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # amount: "98"

    cored q bank balances $FT_RECEIVER_2 --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # amount: "100"

    # check the token issuer address
    cored q bank balances $FT_ISSUER --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
    # amount: "2"
    ```

    Note that the sender's balance is now less than 100 tokens because the send-commission-rate was applied, and 2 tokens were transferred to the token issuer.

### Burn-Rate <a href="#burn-rate" id="burn-rate"></a>

The `burn-rate` works similarly to the [send-commission-rate](https://docs.coreum.dev/tutorials/smart-tokens/asset-ft.html#send-commission-rate). The only difference is that when the `burn-rate` is applied, the token issuer does not receive any tokens. Instead, a certain amount of tokens is burned and removed from circulation.

### Burning <a href="#burning" id="burning"></a>

To burn tokens, you can use the command shown below:

```
cored tx assetft burn 10$FT_DENOM --from $FT_RECEIVER_1 --node=$RPC_URL --chain-id=$CHAIN_ID
```

After executing this command, you can check the account balance with the following command:

```
cored q bank balances $FT_RECEIVER_1 --denom=$FT_DENOM --node=$RPC_URL --chain-id=$CHAIN_ID
# amount: "88"
```

You can also check the total supply of the token by running the following command:

```
cored q bank total --denom=$FT_DENOM --node=$RPC_URL
# amount: "190"
```

Note that neither burn-rate nor send-commission-rate is applied in this case.

### Further Reading <a href="#faq" id="faq"></a>

You can read more about Fungible Tokens at [Coreum Fungible Token](../../developer-options/coreum/fungible-tokens.md) page
