# Validator

## Run validator node <a href="#run-validator-node" id="run-validator-node"></a>

* Follow instructions in [full node instruction](https://docs.coreum.dev/validator/run-full.html) first, but set the following config before you start the node. These configurations will reduce the storage size and in turn will make upgrades faster, and the downside is that the node will not have historical data, and cannot be used as a state-sync server.

(Check if `$COREUM_NODE_CONFIG` is set(it should be there, if you haven't exited from the beginning of installation))

```sh
  crudini --set $COREUM_APP_CONFIG state-sync snapshot-interval 0
  crudini --set $COREUM_APP_CONFIG "" pruning \"everything\"
  crudini --set $COREUM_APP_CONFIG "" min-retain-blocks 10
```

**Keep $COREUM\_HOME/config/node\_key.json and $COREUM\_HOME/config/priv\_validator\_key.json files in a safe place, since they can be used to recover the validator node!**

* Set the moniker variable to reuse it in the following instructions.

```sh
export MONIKER="validator"
```

* (Optional) You can set waiting window between validator restart to avoid double signing. If you set this option, the node will not restart if there is another node running with the same key. This option will help avoid double signing, and is particularly useful if you are migrating your node, or are planing to run a back up node. You can learn more about it [here](https://docs.tendermint.com/v0.34/tendermint-core/validators.html#local-configuration)

(Check if `$COREUM_NODE_CONFIG` is set(it should be there, if you haven't exited from the beginning of installation)

```sh
crudini --set $COREUM_NODE_CONFIG consensus double_sign_check_height 10
```

*   Init new account (if you don't have existing), which will be used for validator control, delegation and staking rewards/commission receiving

    ```sh
    cored keys add $MONIKER --keyring-backend os --chain-id=$COREUM_CHAIN_ID
    ```

You will be asked to set the keyring passphrase, set it, and remember/save it, since you will need it to access your private key.

The output example:

```sh
- name: validator
  type: local
  address: testcorevaloper15fr7w6trtx8nzkjp33tcqj922q6r82tp05gdpe
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AwzsffiidUiFtmNng5pLTH6cj1hv4Ufa+zKZpmRVGfNk"}'
  mnemonic: ""

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

nice equal sample cabbage demise online winner lady theory invest clarify organ divorce wheel patient gap group endless security price smoke insane link position
```

**Attention!** _Keep the mnemonic phrase in a safe place, since it can be used to recover the key!_ (Don't be confused with empty mnemonic after pubkey, it is shown in the end of the output)

*   If you have the mnemonic you can import it (skip it if you generated the account at the previous step)

    ```sh
    cored keys add $MONIKER --keyring-backend os --recover --chain-id=$COREUM_CHAIN_ID
    ```
* You will be asked to "Enter keyring passphrase" and "Enter your bip39 mnemonic".
*   Derive the validator-operator address, which will be used for validator creation.

    ```sh
    cored keys show $MONIKER --bech val --address --keyring-backend os --chain-id=$COREUM_CHAIN_ID
    # output example:
    # testcorevaloper15fr7w6trtx8nzkjp33tcqj922q6r82tp05gdpe
    ```

Fund the account.

* You can find the recommended amount with calculation at [this page](https://docs.coreum.dev/validator/how-much-fund-is-needed.html).
* For the Testnet and Devnet you can request test tokens at [discord channel](https://discord.gg/VgkhYeWmTd)
*   Check that you have enough to create the validator(minimum self delegation is 20k)

    ```sh
    cored q bank balances  $(cored keys show $MONIKER --address --keyring-backend os --chain-id=$COREUM_CHAIN_ID) --denom $COREUM_DENOM --node=$COREUM_NODE --chain-id=$COREUM_CHAIN_ID
    ```

Wait until node is fully synced(it's important). To check sync status run next command:

```sh
echo "catching_up: $(echo  $(cored status) | jq -r '.SyncInfo.catching_up')"
```

* If the output is `catching_up: false`, then the node is fully synced.
* Create validator
  *   set up validator configuration

      ```sh
       # COREUM_VALIDATOR_DELEGATION_AMOUNT default is 20k, must be grater or equal COREUM_MIN_DELEGATION_AMOUNT.
       # We suggest setting 30k, in case of slashing you will be able to unjail validator without replenishing your balance.
       # (Otherwise your validator balance will went below 20k and to start it you should transfer tokens first)
       export COREUM_VALIDATOR_DELEGATION_AMOUNT=20000000000 # (Required)
       export COREUM_VALIDATOR_NAME="" # (Required) update it with the name which is visible on the explorer
       export COREUM_VALIDATOR_WEB_SITE="" # (Optional) update with the site
       export COREUM_VALIDATOR_IDENTITY="" # (Optional) update with identity id, which can generated on the site https://keybase.io/
       export COREUM_VALIDATOR_COMMISSION_RATE="0.10" # (Required) Update with your commission rate
       export COREUM_VALIDATOR_COMMISSION_MAX_RATE="0.20" # (Required) Update with your commission max rate
       export COREUM_VALIDATOR_COMMISSION_MAX_CHANGE_RATE="0.01" # (Required) Update with your commission max change rate
       export COREUM_MIN_DELEGATION_AMOUNT=20000000000 # (Required) default 20k, must be grater or equal min_self_delegation parameter on the current chain
      ```

create validator

```sh
cored tx staking create-validator \
--amount=$COREUM_VALIDATOR_DELEGATION_AMOUNT$COREUM_DENOM \
--pubkey="$(cored tendermint show-validator --chain-id=$COREUM_CHAIN_ID)" \
--moniker="$COREUM_VALIDATOR_NAME" \
--website="$COREUM_VALIDATOR_WEB_SITE" \
--identity="$COREUM_VALIDATOR_IDENTITY" \
--commission-rate="$COREUM_VALIDATOR_COMMISSION_RATE" \
--commission-max-rate="$COREUM_VALIDATOR_COMMISSION_MAX_RATE" \
--commission-max-change-rate="$COREUM_VALIDATOR_COMMISSION_MAX_CHANGE_RATE" \
--min-self-delegation=$COREUM_MIN_DELEGATION_AMOUNT \
--gas auto \
--chain-id=$COREUM_CHAIN_ID \
--from=$MONIKER \
--keyring-backend os -y -b block $COREUM_CHAIN_ID_ARGS
```

(Optional) Troubleshooting

*   Error:

    ```sh
    Error: rpc error: code = NotFound desc = rpc error: code = NotFound desc = account testcore15fr7w6trtx8nzkjp33tcqj922q6r82tp077avs not found: key not found
    ```
* It means that your node is not synced yet, wait for the full sync
*   Error:

    ```sh
    Enter keyring passphrase: Error: invalid character 'o' looking for beginning of value Usage: cored tx staking create-validator [flags]
    ```

### &#x20;<a href="#change-pruning-config-of-running-node" id="change-pruning-config-of-running-node"></a>

### Change Pruning Config of Running Node <a href="#change-pruning-config-of-running-node" id="change-pruning-config-of-running-node"></a>

**Attention!** _It possible to be jailed doing this operation, if you run a duplicate node (slashed and jailed forever), or if you take too long to bring your node back online (slashed and jailed)_

It is recommended to prune as much as possible on a validator node. Doing so will reduce the storage size used by validator, which in turn will make upgrades faster. If you have started a validator node with a non-optimized pruning config, you must follow these steps:

1. change the pruning config as described in this page.
2. remove all the state by following these steps:
   * take backup from data folder in $COREUM\_HOME/data
   * remove all the folders but keep this file `priv_validator_state.json`
   * set all the values inside `priv_validator_state.json` to 0
3. sync up using state-sync snapshot.
