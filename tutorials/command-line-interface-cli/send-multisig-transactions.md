# Send Multisig Transactions

The sample below describes the full flow from the creation of the multisig account to the tx broadcast.

* Follow the [instruction](https://docs.coreum.dev/tutorials/cored.html) to install cored binary.
* Verify that [network variables](https://docs.coreum.dev/tutorials/network-variables.html) are set up correctly.
*   Add the keys that we are going to use for the multisig.

    Option 1: generate the keys ourselves.

    {% code lineNumbers="true" %}
    ```sh
    cored keys add k1 $COREUM_CHAIN_ID_ARGS
    cored keys add k2 $COREUM_CHAIN_ID_ARGS
    cored keys add k3 $COREUM_CHAIN_ID_ARGS
    cored keys add recipient $COREUM_CHAIN_ID_ARGS
    ```
    {% endcode %}

Option 2: add the public keys provided by the people who are going to be part of the multisig.

{% code lineNumbers="true" %}
```sh
cored keys add k1 --pubkey='{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A8kOpeBMbmri5rvLjlqN6kOuNzRVUnr2vtinCkKMmwKU"}' $COREUM_CHAIN_ID_ARGS
cored keys add k2 --pubkey='{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Aul+q9bj3zZTADlKbLcpmn/roDj2d0DJIHIQiyCQM8Fk"}' $COREUM_CHAIN_ID_ARGS
cored keys add k3 --pubkey='{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A3AcsNQ+FNwnUovxN/6/sa/vVN+Lc89IksZQKpLyAQ16"}' $COREUM_CHAIN_ID_ARGS
```
{% endcode %}

For this option, each member needs to provide the public key.

```sh
cored keys list
```

Output example:

```sh
- name: k1
  type: local
  address: core1qj7d46j56khz4ysvvgt5elghhu6p3fxepzme7y
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A8kOpeBMbmri5rvLjlqN6kOuNzRVUnr2vtinCkKMmwKU"}'
  mnemonic: ""
```

Generate the multisig account with the 2 signatures threshold.

```sh
cored keys add k1k2k3 --multisig "k1,k2,k3" --multisig-threshold 2 $COREUM_CHAIN_ID_ARGS
```

To set up a 2-of-3 multisig, each member must supply their individual public key. In this example, we already hold all keys.

Output example:

```sh
- name: k1k2k3
  type: multi
  address: devcore13purcatgmnadw3606rcyatmt60ys6e37mcnaar
  pubkey: '{"@type":"/cosmos.crypto.multisig.LegacyAminoPubKey","threshold":2,"public_keys":[{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A8kOpeBMbmri5rvLjlqN6kOuNzRVUnr2vtinCkKMmwKU"},{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Aul+q9bj3zZTADlKbLcpmn/roDj2d0DJIHIQiyCQM8Fk"},{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A3AcsNQ+FNwnUovxN/6/sa/vVN+Lc89IksZQKpLyAQ16"}]}'
  mnemonic: ""
```

* The address here is the multisig account address.
*   Get the address from the keystore.

    ```sh
    cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS
    ```
* Use [faucet](https://docs.coreum.dev/tools-ecosystem/faucet.html) to fund `k1k2k3` account
*   Check the multisig account balances.

    ```sh
    cored q bank balances $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
    ```

Generate the json tx to send some coins to the recipient.

```sh
cored tx bank send $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) $(cored keys show --address recipient $COREUM_CHAIN_ID_ARGS) 700$COREUM_DENOM \
--from $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) \
--generate-only $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS > bank-unsigned-tx.json
```

Check the tx content.

```sh
cat bank-unsigned-tx.json
```

Output example:

```sh
{"body":{"messages":[{"@type":"/cosmos.bank.v1beta1.MsgSend","from_address":"devcore13purcatgmnadw3606rcyatmt60ys6e37mcnaar","to_address":"devcore1lyru5pvjymya9xq0rsg406fss45sama8e9dqrs","amount":[{"denom":"dacore","amount":"700"}]}],"memo":"","timeout_height":"0","extension_options":[],"non_critical_extension_options":[]},"auth_info":{"signer_infos":[],"fee":{"amount":[{"denom":"dacore","amount":"300000000"}],"gas_limit":"200000","payer":"","granter":""}},"signatures":[]}
```

*   Sign the tx from k1 account.

    ```sh
    cored tx sign bank-unsigned-tx.json --multisig $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --from k1 --output-document k1sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
    ```
*   Add the signature to the json tx.

    ```sh
    cored tx multisign bank-unsigned-tx.json k1k2k3 k1sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS > bank-signed-tx.json
    ```
*   Try to send partially signed tx to check that it won't pass.

    ```sh
    cored tx broadcast bank-signed-tx.json -y -b block $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS 
    ```
*   Add one more signature.

    ```sh
    cored tx sign bank-unsigned-tx.json --multisig $(cored keys show --address k1k2k3 $COREUM_CHAIN_ID_ARGS) --from k2 --output-document k2sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
    ```
*   Add the signature to the json tx.

    ```sh
    cored tx multisign bank-unsigned-tx.json k1k2k3 k1sign.json k2sign.json $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS > bank-signed-tx.json
    ```
*   Try to send the tx now.

    ```sh
    cored tx broadcast bank-signed-tx.json -y -b block $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
    ```
*   Check the recipient balance.

    ```shell
    cored q bank balances $(cored keys show --address recipient $COREUM_CHAIN_ID_ARGS) $COREUM_CHAIN_ID_ARGS $COREUM_NODE_ARGS
    ```
