# Testing Multiple Contracts

The tutorial provides an example of how to test a contract workspace, where we have multi-contract interactions, using the [CosmWasm multitest library](https://github.com/CosmWasm/cw-multi-test).

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

To complete this tutorial, you need to:

* Install [rust and cargo](https://www.rust-lang.org/tools/install).
* Be familiar with the Rust programming language.

### Source Code <a href="#source-code" id="source-code"></a>

The complete source code is located [here](https://github.com/CoreumFoundation/secure-messaging-poc).

### Getting Started <a href="#getting-started" id="getting-started"></a>

* Clone the repository workspace.

```
git clone git@github.com:CoreumFoundation/secure-messaging-poc.git
```

The structure of the workspace is the following:

```
├── contracts
│   ├── controller
│   │   ├── Cargo.toml
│   │   └── src
│   │       └── ...
│   ├── messages
│   │   ├── Cargo.toml
│   │   └── src
│   │       └── ...
│   ├── profiles
│   │   ├── Cargo.toml
│   │   └── src
│   │       └── ...
├── packages
│   ├── utils
│   │   ├── Cargo.toml
│   │   └── src
│   │       └── ...
├── Cargo.lock
├── Cargo.toml
```

As you can see, we have three contracts (controller, messages and profiles) and a package utils that contains code shared by multiple contracts. Since the controller contract is in charge of interacting with the messages and profiles contracts we are going to keep our integration tests there.

* Define dev dependencies.

Since we are going to keep our tests under the controller contract, we are going to define cw-multi-test in the Cargo.toml of the controller contract. We will also use some elements of the other contracts so we will include those contracts in our dev dependencies as well. These dependencies are only used for tests.

```
[dev-dependencies]
cw-multi-test = "0.16.5"
profiles = { path = "../profiles" }
messages = { path = "../messages" }
```

* Let's take a look now at how we can write the integration tests.

1. We create a wrapper around our contracts that we will use to store the code.

```
fn controller_contract() -> Box<dyn Contract<Empty>> {
    let contract = ContractWrapper::new(execute, instantiate, query).with_reply(reply);
    Box::new(contract)
}

fn profiles_contract() -> Box<dyn Contract<Empty>> {
    let contract = ContractWrapper::new(profilesExecute, profilesInstantiate, profilesQuery);
    Box::new(contract)
}

fn messages_contract() -> Box<dyn Contract<Empty>> {
    let contract = ContractWrapper::new(messagesExecute, messagesInstantiate, messagesQuery);
    Box::new(contract)
}
```

In the case of our controller contract, we have a reply entry point so we need to add that as well. In the case that, for example, there was a migrate or a sudo entry point we would need to add it in the same way.

2. We initialize our test App (which will simulate the blockchain).

For some tests we don't need to use balances so we simply use:

```
let mut app = App::default();
```

In the case that we need to initialize some values we can use AppBuilder that will set our parametres. In this example we initialize the balance of an account:

```
    let admin = Addr::unchecked("admin");
    let mut app = AppBuilder::new().build(|router, _api, storage| {
        router
            .bank
            .init_balance(storage, &admin, coins(10000, DENOM))
            .unwrap();
    });
```

This will initialize our test App initializing the balance of an admin address with 10000 ucore.

3. Store the contracts so that we can interact with them.

```
    let code_id_controller = app.store_code(controller_contract());
    let code_id_profiles = app.store_code(profiles_contract());
    let code_id_messages = app.store_code(messages_contract());
```

Using our previously created App, we can store the contracts that we want and obtain their code\_id to initialize them.

4. Now that we have our contracts stored, we can interact with them. For example, we can instantiate our controller contract.

```
let contract_addr = app
        .instantiate_contract(
            code_id_controller,
            admin,
            &ControllerInstantiateMsg {
                code_id_profiles,
                code_id_messages,
                message_max_len: 5000,
                message_query_default_limit: 50,
                message_query_max_limit: 500,
                create_profile_cost: Some(coin(100, DENOM)),
                send_message_cost: Some(coin(10, DENOM)),
            },
            &[],
            "Controller",
            None,
        )
        .unwrap();
```

Since our controller contract also instantiates a profiles and a messages contract (that's why we provide the other code\_ids in the message), we only need to instantiate this one.

5. Query any of our contracts providing their contract address.

```
let resp: Config = app
        .wrap()
        .query_wasm_smart(contract_addr, &ControllerQueryMsg::Config {})
        .unwrap();
```

In this example, we query the Config of our controller by querying it.

Since our controller contract also contains the contract addresses of the profiles and messages contract, we can obtain these addresses to query them. Example:

```
let resp: ContractAddressesResponse = app
        .wrap()
        .query_wasm_smart(contract_addr, &ControllerQueryMsg::ContractAddresses {})
        .unwrap();

let resp_profiles: ProfileInfo = app
        .wrap()
        .query_wasm_smart(
            resp.profiles_contract_addr,
            &ProfilesQueryMsg::AddressInfo { address: admin },
        )
        .unwrap();
```

As you can see, we first obtain the contract address of the profiles contract and then we directly query that contract to obtain the user id of a specific address.

6. Execute a contract.

```
    let msg_create_profile = &ControllerExecuteMsg::CreateProfile {
        user_id: "myuser".to_owned(),
        pubkey: "mypubkey".to_owned(),
    };

    let send_funds = coins(100, DENOM);

    app.execute_contract(
        admin.clone(),
        contract_addr.clone(),
        msg_create_profile,
        &send_funds,
    )
    .unwrap();
```

In this example we create a profile providing the message to the controller contract and attaching funds to it.

* Write and run our tests:

With all these tools we can write all the tests that we deem necessary and tests our contracts in an easy and convenient way.

By simply running

```
cargo test
```

or

```
cargo test --package controller
```

we can test all our integration tests. The first command will run all the tests in all the contracts, but since we only have tests in our controller contract, we can indicate to package where the tests are located so that our output only shows what we want.

### Build all the contracts <a href="#build-all-the-contracts" id="build-all-the-contracts"></a>

Since we have a workspace with multiple contracts instead of a single contract, we can utilize the workspace optimizer that will compile and optimize all our contracts and put them in the /artifacts folder.

```
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/optimizer:0.15.0
```

### Next steps <a href="#next-steps" id="next-steps"></a>

* Read Coreum [modules](https://docs.coreum.dev/modules/main.html) specification, to be familiar with the custom Coreum functionality you can use for your application.
* Read [WASM docs](https://docs.cosmwasm.com/docs/) to understand all supported WASM features.
* Check other [tutorials](https://docs.coreum.dev/tutorials/main.html) to find something you might be interested in additionally.
