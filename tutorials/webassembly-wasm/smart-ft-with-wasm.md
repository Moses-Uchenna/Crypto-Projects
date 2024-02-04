# Smart FT with WASM

The tutorial provides an example of how to develop, deploy and use the WASM fungible smart token with the airdrop functionality.

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

To complete this tutorial, you need to:

* Install [rust and cargo](https://www.rust-lang.org/tools/install).
* Be familiar with the Rust programming language.
* Have a general understanding of how the Coreum blockchain works.
* Follow the [instruction](https://docs.coreum.dev/tutorials/cored.html) to install cored binary.
* Install the required util: `jq`.
* Set the [network variables](https://docs.coreum.dev/tutorials/network-variables.html) for the development (testnet is preferable).

### Source Code <a href="#source-code" id="source-code"></a>

The complete source code is located [here](https://github.com/CoreumFoundation/tutorials/tree/main/wasm/ft-airdrop).

### Getting Started <a href="#getting-started" id="getting-started"></a>

* Clone the smart contract template

```
git clone https://github.com/CoreumFoundation/tutorials.git
```

* Go to the contract directory.

```
cd tutorials/wasm/ft-airdrop
```

* Generate a new account

```
cored keys add wasm-deployer $COREUM_CHAIN_ID_ARGS
```

* Fund account

Use the [faucet](https://docs.coreum.dev/tools-ecosystem/faucet.html) to fund your account

* Check the balance

```
cored q bank balances $(cored keys show wasm-deployer $COREUM_CHAIN_ID_ARGS -a) --denom $COREUM_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

### Build contract <a href="#build-contract" id="build-contract"></a>

* Build the contract

```
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/optimizer:0.15.0
```

This operation might take a significant amount of time.

### Deploy contract <a href="#deploy-contract" id="deploy-contract"></a>

* Deploy the built artifact.

```
RES=$(cored tx wasm store artifacts/ft_airdrop.wasm \
    --from wasm-deployer --gas auto --gas-adjustment 1.3 -y -b block --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS)
echo $RES
CODE_ID=$(echo $RES | jq -r '.logs[0].events[-1].attributes[-1].value')
echo "Code ID: $CODE_ID"
```

* Check the deployed code.

```
cored q wasm code-info $CODE_ID $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

### Instantiate contract <a href="#instantiate-contract" id="instantiate-contract"></a>

* Instantiating the contract.

#### Code:

```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub struct InstantiateMsg {
    pub symbol: String,
    pub subunit: String,
    pub precision: u32,
    pub initial_amount: Uint128,
    pub airdrop_amount: Uint128,
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: InstantiateMsg,
) -> Result<Response<CoreumMsg>, ContractError> {
    set_contract_version(deps.storage, CONTRACT_NAME, CONTRACT_VERSION)?;
    let issue_msg = CoreumMsg::AssetFT(assetft::Msg::Issue {
        symbol: msg.symbol,
        subunit: msg.subunit.clone(),
        precision: msg.precision,
        initial_amount: msg.initial_amount,
        description: None,
        features: Some(vec![0]), // 0 - minting
        burn_rate: Some("0".into()),
        send_commission_rate: Some("0.1".into()), // 10% commission for sending
    });

    let denom = format!("{}-{}", msg.subunit, env.contract.address).to_lowercase();

    let state = State {
        owner: info.sender.into(),
        denom,
        minted_for_airdrop: msg.initial_amount,
        airdrop_amount: msg.airdrop_amount,
    };
    STATE.save(deps.storage, &state)?;

    Ok(Response::new()
        .add_attribute("owner", state.owner)
        .add_attribute("denom", state.denom)
        .add_message(issue_msg))
}
```

In the code snippet above we `instantiate` the deployed contract using the `instantiate` function marked with `entry_point` macro. The contract can be instantiated multiple times and will get a new address each time. On the installation, the contract issues an FT with the `minting` feature enabled, and `send_commission_rate` extension equals `10%`. After the instantiation, the contract will become the FT issuer and will be able to control it. Since we have set up the `minting` feature only, it will be able to `mint`. Read more about [Coreum Fungible Token](https://docs.coreum.dev/modules/assetft.html). Also, on the instantiation we declare the `owner` the address of the instantiation, which we can use conditionally to inherit some contract/issuer permissions.

#### CLI command:

```sh
SUBUNIT=mysubunit
cored tx wasm instantiate $CODE_ID \
 "{\"symbol\":\"mysymbol\",\"subunit\":\"$SUBUNIT\",\"precision\":6,\"initial_amount\":\"1000000000\",\"airdrop_amount\":\"1000000\"}" \
  --amount="10000000$COREUM_DENOM" --no-admin --label "My smart token" --from wasm-deployer --gas auto --gas-adjustment 1.3 -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* On token `instantiation` we can set custom parameters we allow to set in the contract.
  * symbol - FT symbol
  * subunit - FT subunit (used to for the denom creation)
  * precision - FT precision
  * initial\_amount - the amount to mint initially
  * airdrop\_amount - the amount we allow to be received as airdrop
  * `--amount="10000000$COREUM_DENOM"` - the amount we send to the contract to let it mint the FT, that amount will be paid by the contract for the FT creation.
* Capture the contract address.

```sh
cored q wasm list-contract-by-code $CODE_ID --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
CONTRACT_ADDRESS=$(cored q wasm list-contract-by-code $CODE_ID --output json $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS | jq -r '.contracts[-1]')
echo "Contract address: $CONTRACT_ADDRESS"
```

* Build denom and query supply.

```sh
FT_DENOM="$SUBUNIT-$CONTRACT_ADDRESS"
echo "Created denom: $FT_DENOM"
echo "Open the https://explorer.testnet-1.coreum.dev/coreum/accounts/$CONTRACT_ADDRESS to check it on explorer."
```

```
cored q bank total --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

```
cored q bank denom-metadata --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

```
cored q assetft token $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

### Interact with the contract <a href="#interact-with-the-contract" id="interact-with-the-contract"></a>

#### Receive the airdrop. <a href="#receive-the-airdrop" id="receive-the-airdrop"></a>

#### Code:

```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum ExecuteMsg {
    MintForAirdrop { amount: u128 },
    ReceiveAirdrop {},
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(
    deps: DepsMut,
    _env: Env,
    info: MessageInfo,
    msg: ExecuteMsg,
) -> Result<Response<CoreumMsg>, ContractError> {
    match msg {
        ExecuteMsg::MintForAirdrop { amount } => mint_for_airdrop(deps, info, amount),
        ExecuteMsg::ReceiveAirdrop {} => receive_airdrop(deps, info),
    }
}

fn receive_airdrop(deps: DepsMut, info: MessageInfo) -> Result<Response<CoreumMsg>, ContractError> {
    let mut state = STATE.load(deps.storage)?;
    if state.minted_for_airdrop < state.airdrop_amount {
        return Err(ContractError::CustomError {
            val: "not enough minted".into(),
        });
    }
    let send_msg = cosmwasm_std::BankMsg::Send {
        to_address: info.sender.into(),
        amount: vec![Coin {
            amount: state.airdrop_amount,
            denom: state.denom.clone(),
        }],
    };

    state.minted_for_airdrop = state.minted_for_airdrop.sub(state.airdrop_amount);
    STATE.save(deps.storage, &state)?;

    Ok(Response::new()
        .add_attribute("method", "receive_airdrop")
        .add_attribute("denom", state.denom)
        .add_attribute("amount", state.airdrop_amount.to_string())
        .add_message(send_msg))
}
```

In the code snippet above in the `ExecuteMsg` enum, we declare the transactions/messages we allow to execute. The `execute` function marked with `entry_point` macro routes them to the proper handlers. The `receive_airdrop` function is the handler for the `ReceiveAirdrop` message, that checks that we have enough tokens minted for the airdrop and send them to the message sender.

#### CLI command:

```
cored tx wasm execute $CONTRACT_ADDRESS '{"receive_airdrop":{}}' --from wasm-deployer -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

Pay attention, that the messages in enums are decoded using the snake case, and `ReceiveAirdrop` is expected as `receive_airdrop`.

* Check balance

```
cored q bank balances $(cored keys show wasm-deployer $COREUM_CHAIN_ID_ARGS -a) --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

#### Mint for the airdrop. <a href="#mint-for-the-airdrop" id="mint-for-the-airdrop"></a>

* Check the remaining airdrop amount

#### Code:

```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    Token {},
    MintedForAirdrop {},
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn query(deps: Deps<CoreumQueries>, _env: Env, msg: QueryMsg) -> StdResult<Binary> {
    match msg {
        QueryMsg::Token {} => token(deps),
        QueryMsg::MintedForAirdrop {} => minted_for_airdrop(deps),
    }
}

fn minted_for_airdrop(deps: Deps<CoreumQueries>) -> StdResult<Binary> {
    let state = STATE.load(deps.storage)?;
    let res = AmountResponse {
        amount: state.minted_for_airdrop,
    };
    to_binary(&res)
}
```

In the code snippet above in the `QueryMsg` enum, we declare the queries we allow to query. The `query` function marked with `entry_point` macro routes them to the proper handlers. The `minted_for_airdrop` function is the handler for the `MintedForAirdrop` query, which returns the amount minted for the airdrop.

#### CLI command:

```
cored q wasm contract-state smart $CONTRACT_ADDRESS '{"minted_for_airdrop": {}}' $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

Pay attention, that the queries in enums are decoded using the snake case, and `MintedForAirdrop` is expected as `minted_for_airdrop`.

* Mint more for the airdrop

#### Code:

```rust
fn mint_for_airdrop(
    deps: DepsMut,
    info: MessageInfo,
    amount: u128,
) -> Result<Response<CoreumMsg>, ContractError> {
    let mut state = STATE.load(deps.storage)?;
    if info.sender != state.owner {
        return Err(ContractError::Unauthorized {});
    }

    let msg = CoreumMsg::AssetFT(assetft::Msg::Mint {
        coin: Coin::new(amount, state.denom.clone()),
    });

    state.minted_for_airdrop = state.minted_for_airdrop.add(Uint128::new(amount));
    STATE.save(deps.storage, &state)?;

    Ok(Response::new()
        .add_attribute("method", "mint_for_airdrop")
        .add_attribute("denom", state.denom)
        .add_attribute("amount", amount.to_string())
        .add_message(msg))
}
```

In the code snippet above the `mint_for_airdrop` function checks that the sender is the owner (the address which instantiated the contract), and mints the additional amount for the airdrop.

#### CLI command:

```
cored tx wasm execute $CONTRACT_ADDRESS \
  "{\"mint_for_airdrop\":{\"amount\":\"5000000\" }}" \
 --from wasm-deployer -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Check the new airdrop amount

```
cored q wasm contract-state smart $CONTRACT_ADDRESS '{"minted_for_airdrop": {}}' $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

#### Use send commission rate. <a href="#use-send-commission-rate" id="use-send-commission-rate"></a>

The `send commission rate` is the percent of sending amount which will be sent to the issuer in addition to the sending amount.

* Add a new account to receive the issuer token

```
cored keys add recipient $COREUM_CHAIN_ID_ARGS
```

* Check the balance of `wasm-deployer`

```
cored q bank balances $(cored keys show wasm-deployer $COREUM_CHAIN_ID_ARGS -a) --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Check the balance of the smart contract

```
cored q bank balances $CONTRACT_ADDRESS --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Send some FT from `wasm-deployer` to the `recipient`

```
cored tx bank send $(cored keys show wasm-deployer $COREUM_CHAIN_ID_ARGS -a) $(cored keys show recipient $COREUM_CHAIN_ID_ARGS -a) \
  1000$FT_DENOM --from wasm-deployer -b block -y $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

* Check the balance of the `recipient`

```
cored q bank balances $(cored keys show recipient $COREUM_CHAIN_ID_ARGS -a) --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

The `recipient` has received `1000` tokens.

* Check the balance of `wasm-deployer`

```
cored q bank balances $(cored keys show wasm-deployer $COREUM_CHAIN_ID_ARGS -a) --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

The `wasm-deployer` sent `1000` tokens to the recipient and additionally `10%` to the issuer which is the contract address.

* Check the balance of the smart contract

```
cored q bank balances $CONTRACT_ADDRESS --denom $FT_DENOM $COREUM_NODE_ARGS $COREUM_CHAIN_ID_ARGS
```

### Next steps <a href="#next-steps" id="next-steps"></a>

* Read Coreum [modules](https://docs.coreum.dev/modules/main.html) specification, to be familiar with the custom Coreum functionality you can use for your application.
* Read [WASM docs](https://docs.cosmwasm.com/docs/) to understand all supported WASM features.
* Check other [tutorials](https://docs.coreum.dev/tutorials/main.html) to find something you might be interested in additionally.

Last Updated: 12/14/2023, 3:39:40
