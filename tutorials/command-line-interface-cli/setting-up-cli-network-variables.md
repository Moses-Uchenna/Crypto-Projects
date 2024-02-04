# Setting up CLI Network variables

If you are looking for Validator Network Variables, go to [this page](https://docs.coreum.dev/validator/network-variables.html)

This document describes the command to set up the environment, depending on the type of network you want to use.

<table data-full-width="true"><thead><tr><th width="155">-</th><th>Mainnet</th><th>Testnet</th><th>Devnet</th><th>znet (localnet)</th></tr></thead><tbody><tr><td><strong>Chain ID</strong></td><td>coreum-mainnet-1</td><td>coreum-testnet-1</td><td>coreum-devnet-1</td><td>coreum-devnet-1</td></tr><tr><td><strong>Denom</strong></td><td>ucore</td><td>utestcore</td><td>udevcore</td><td>udevcore</td></tr><tr><td><strong>RPC URL</strong></td><td>https://full-node.mainnet-1.coreum.dev:26657</td><td>https://full-node.testnet-1.coreum.dev:26657</td><td>https://full-node.devnet-1.coreum.dev:26657</td><td>http://localhost:26657</td></tr><tr><td><strong>GRPC URL</strong></td><td>full-node.mainnet-1.coreum.dev:9090</td><td>full-node.testnet-1.coreum.dev:9090</td><td>full-node.devnet-1.coreum.dev:9090</td><td>localhost:9090</td></tr><tr><td><strong>REST URL</strong></td><td>https://full-node.mainnet-1.coreum.dev:1317</td><td>https://full-node.testnet-1.coreum.dev:1317</td><td>https://full-node.devnet-1.coreum.dev:1317</td><td>http://localhost:1317</td></tr><tr><td><strong>Cored version</strong></td><td>v2.0.2</td><td>v2.0.2</td><td>check the latest devnet release</td><td>already installed via crust</td></tr><tr><td><strong>Explorer API Beta</strong></td><td>https://hasura.mainnet-1.coreum.dev/v1/graphql</td><td>https://hasura.testnet-1.coreum.dev/v1/graphql</td><td>https://hasura.devnet-1.coreum.dev/v1/graphql</td><td>http://localhost:8080/v1/graphql</td></tr></tbody></table>

The following should be noted:

* Keep in mind that our Public RPC Node is stable, but there is always a risk of DDoS attacks, and if you build your own product (wallet, etc.), it is recommended to rely on your own RPC Node.
* Also, having your own RPC node is recommended if you frequently query the Node (for instance, for indexing), since we have rate limiting there.
*   Set the chain environment (env) variables with the values corresponding to the network you want to connect to from the table above.

    {% code lineNumbers="true" %}
    ```sh
    export COREUM_CHAIN_ID="{Chain ID}"
    export COREUM_DENOM="{Denom}"
    export COREUM_NODE="{RPC URL}"
    export COREUM_VERSION="{Cored version}"

    export COREUM_CHAIN_ID_ARGS="--chain-id=$COREUM_CHAIN_ID"
    export COREUM_NODE_ARGS="--node=$COREUM_NODE"

    export COREUM_HOME=$HOME/.core/"$COREUM_CHAIN_ID"

    export COREUM_BINARY_NAME=$(arch | sed s/aarch64/cored-linux-arm64/ | sed s/x86_64/cored-linux-amd64/)
    ```
    {% endcode %}

⚠️**Attention!** _Set those variables globally so it is automatically set after starting a new terminal session._

These commands set up the necessary environment variables for a seamless connection to the specified Coreum network. The variables include information such as the chain ID, denomination, RPC URL, Coreum version, and additional arguments for convenience.\
