# Smart FTs with Access Control List (ACL)

### Create Multisig account and issue an asset <a href="#create-multisig-account-and-issue-an-asset" id="create-multisig-account-and-issue-an-asset"></a>

The documentation below explains how to create a Multisig account and grant different permissions to individual addresses using the Authz Module so that each address has different permissions for the management of Coreum assets.

* Follow the [instruction](https://docs.coreum.dev/tutorials/cored.html) to install cored binary.
* Verify that [network variables](https://docs.coreum.dev/tutorials/network-variables.html) are set up correctly.
* The first step is to create a multisig account. Follow the steps in [multisig creation](https://docs.coreum.dev/tutorials/multisig.html) to set it up.
* Generate the json tx to issue an FT from the multisig.

{% code lineNumbers="true" %}
```sh
    # cored tx assetft issue [symbol] [subunit] [precision] [initial_amount] [description] --from [issuer] --features=burning,freezing,minting,whitelisting --burn-rate=0.12 --send-commission-rate=0.2 [flags]
    cored tx assetft issue MYFT umyft 6 1000000 "Multisig token" \
    --from $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) \
    --features=burning,freezing,minting,whitelisting --send-commission-rate=0.02 \
    --generate-only $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS > asset-issuance.json
    # where:
    # MYFT - token symbol, is the display name of a token used mostly for UI purposes. An example of a token symbol is BTC.
    # umyft - subunit, it is used to construct the denom of an FT. All on-chain operations are made using subunits. An example of a subunit is satoshi. This is the minimum amount of a denom that you can operate with.
    # 6 - precision(decimal count, so 1000000umyft=1MYFT).
    # 1000000 - the initial amount of umyft (1 MYFT)
    # --features - optional flag that defines what features the FT will have
    # --send-commission-rate - the value of the commission rate charged when sent.
    # In this particular case we didn't incluse burn-rate so the value of that will be 0.
```
{% endcode %}

*   Check the tx content.

    ```sh
    cat asset-issuance.json
    ```
* **Output:**

```
{"body":{"messages":[{"@type":"/coreum.asset.ft.v1.MsgIssue","issuer":"testcore1ywrydndyqmpx88ch7n3pcsq7g4p7rwrj46rs4c","symbol":"MYFT","subunit":"umyft","precision":6,"initial_amount":"1000000","description":"Multisig token","features":["burning","freezing","minting","whitelisting"],"burn_rate":"0.000000000000000000","send_commission_rate":"0.020000000000000000"}],"memo":"","timeout_height":"0","extension_options":[],"non_critical_extension_options":[]},"auth_info":{"signer_infos":[],"fee":{"amount":[{"denom":"utestcore","amount":"10096"}],"gas_limit":"200000","payer":"","granter":""}},"signatures":[]}
```

Now we will need to first sign the transactions individually by enough multisig members (sign) and then add those signatures to the transaction that we just created with the multisign address (multisign).

*   Sign the tx with 1 of the multisig members.

    ```sh
    cored tx sign asset-issuance.json --multisig $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --from k1 --output-document signerkey1sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
    ```

    NOTE: In a real scenario we will send the unsigned TX to the multisig member for him to sign it.
*   Add the signature to the json tx.

    ```sh
    cored tx multisign asset-issuance.json k1k2k3 signerk1sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS > asset-issuance-signed.json
    ```
*   Try to send partially signed tx to check that it won't pass.

    ```sh
    cored tx broadcast asset-issuance-signed.json -y -b block $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS 
    ```


*   **Output:**

    ```
    raw_log: 'signature verification failed; please verify account number (2937), sequence
    (1) and chain-id (coreum-testnet-1): unauthorized'
    ```



*   Add one more signature.

    ```sh
    cored tx sign asset-issuance.json --multisig $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --from k2 --output-document signerkey2sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
    ```
*   Add the signatures to the json tx.

    ```sh
    cored tx multisign asset-issuance.json k1k2k3 signerkey1sign.json signerkey2sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS  > asset-issuance-signed.json
    ```
*   Try to send the tx now.

    ```sh
    cored tx broadcast asset-issuance-signed.json -y -b block $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
    ```

Now, the asset is correctly issued by the multisig account.

**Grant permission to an address to perform transactions on behalf of the multisig account:**

*   We create an account to which we are going to grant the right to mint assets on behalf of the multisig account. For this step we could also use an account provided by someone else and add it the same way we did in the first step of this document.

    ```sh
    cored keys add minter $COREUM_CHAIN_ID_ARGS
    ```
*   Generate the json tx to give an account permission to mint.

    ```sh
    cored tx authz grant $(cored keys show --address minter $COREUM_CHAIN_ID_ARGS) generic --msg-type=/coreum.asset.ft.v1.MsgMint --from $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --generate-only $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS > authz-mint-unsigned.json
    ```

In this command we are granting the address we created before the right to mint tokens on behalf of the multisig account. If we wanted to give a different kind of permission we would change the --msg-type flag with the corresponding message. Here are other messages for [AssetFTopen in new window](https://github.com/CoreumFoundation/coreum/blob/master/proto/coreum/asset/ft/v1/tx.proto) and for [AssetNFTopen in new window](https://github.com/CoreumFoundation/coreum/blob/master/proto/coreum/asset/nft/v1/tx.proto).

*   We can check the tx content.

    ```sh
    cat authz-mint-unsigned.json
    ```


*   **Output:**

    ```
    {"body":{"messages":[{"@type":"/cosmos.authz.v1beta1.MsgGrant","granter":"testcore1ywrydndyqmpx88ch7n3pcsq7g4p7rwrj46rs4c","grantee":"testcore17dj6sxlcugcs5zugx294dvnsudxvxuykn3s2qg","grant":{"authorization":{"@type":"/cosmos.authz.v1beta1.GenericAuthorization","msg":"/coreum.asset.ft.v1.MsgMint"},"expiration":"2024-07-06T15:23:16Z"}}],"memo":"","timeout_height":"0","extension_options":[],"non_critical_extension_options":[]},"auth_info":{"signer_infos":[],"fee":{"amount":[{"denom":"utestcore","amount":"6875"}],"gas_limit":"200000","payer":"","granter":""}},"signatures":[]}
    ```

As we can see from the output, the authz message grants permissions to the address for 1 year (or it can be revoked using an authz revoke message).

*   Sign the tx with enough multisig members to pass the threshold.

    {% code lineNumbers="true" %}
    ```sh
    cored tx sign authz-mint-unsigned.json --multisig $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --from k1 --output-document signerkey1sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS

    cored tx sign authz-mint-unsigned.json --multisig $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --from k2 --output-document signerkey2sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
    ```
    {% endcode %}
*   Add the signatures to the json tx.

    ```sh
    cored tx multisign authz-mint-unsigned.json k1k2k3 signerkey1sign.json signerkey2sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS > authz-mint-signed.json
    ```
*   Send the tx now.

    ```sh
    cored tx broadcast authz-mint-signed.json -y -b block $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
    ```

And now the address "minter" can mint the AssetFT token "MYFT" (and any other tokens issued by the multisig) on behalf of the multisig by itself without limits. Let's test it.

* Use [faucet](https://docs.coreum.dev/tools-ecosystem/faucet.html) to fund `minter` account
*   Mint token.

    {% code lineNumbers="true" %}
    ```sh
    export FT_ISSUER=$(cored keys show k1k2k3 --address $COREUM_CHAIN_ID_ARGS)
    export FT_DENOM=umyft-$FT_ISSUER
    cored tx assetft mint 200$FT_DENOM --from $(cored keys show k1k2k3 --address $COREUM_CHAIN_ID_ARGS) $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS --generate-only > tx.json && cored tx authz exec tx.json --from $(cored keys show minter --address $COREUM_CHAIN_ID_ARGS) -b block $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
    ```
    {% endcode %}
*   Check that we successfully minted.

    ```sh
    cored q bank balances $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --denom=$FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
    ```


*   Output:

    ```
    amount: "1000200"
    denom: umyft-testcore1tm5fzez64negxc0jl0hg869g35w4c5f9e9qm5c
    ```

As we can see, our 200 tokens have been minted correctly and have been added to the multisig wallet address.

### Creating and managing a Multisig using a UI instead of CLI <a href="#creating-and-managing-a-multisig-using-a-ui-instead-of-cli" id="creating-and-managing-a-multisig-using-a-ui-instead-of-cli"></a>

There are currently multiple implementations of interfaces. In the main Cosmos repository we can find [Cosmos Multisig UIopen in new window](https://github.com/cosmos/cosmos-multisig-ui) which allows us to create and use multisigs more comfortably on any Cosmos SDK based chain. Therefore, it also works with the Coreum blockchain, we must simply select Coreum on the top right of the page and it will fill in all the Coreum mainnet information. If we want to adjust it for testnet we can click on the settings wheel and modify the information there to point towards our testnet or devnet. This information is in [network variables](https://docs.coreum.dev/tutorials/network-variables.html).

It allows us to create multisigs using both public keys (like we did in this tutorial with CLI) or our addresses. If you want to use addresses to create a multisig instead of public keys, these addresses must have been created on chain (they must have received funds before). If these addresses have never received any tokens, they can not be used to create the multisig because the Public Key can not be obtained from the chain.

In general, these multisig implementions have the following workflow.

1. Create the multisig.
2. Use the multisig to create a transaction.
3. Multisig members have to go on the UI, connect with their wallet and sign the transaction that was created using the multisig.
4. Once the threshold has been reached, any member can broadcast the transaction.
