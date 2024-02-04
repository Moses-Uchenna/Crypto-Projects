# Getting Started

## Introduction

This tutorial explains how to use [**network variables**](https://github.com/CoreumFoundation/coreum) in Coreum, which are **on-chain storage** for **smart contracts** that can be accessed and modified by any node in the network. Command used to setup an environment (env) depends on the network used; below the table shows information on different networks that can be used on Coreum.

<table data-full-width="false"><thead><tr><th width="163">-</th><th>mainnet</th><th>testnet</th><th>devnet</th><th>znet (localnet)</th></tr></thead><tbody><tr><td><strong>Chain ID</strong></td><td>coreum-mainnet-1</td><td>coreum-testnet-1</td><td>coreum-devnet-1</td><td>coreum-devnet-1</td></tr><tr><td><strong>Denom</strong></td><td>ucore</td><td>utestcore</td><td>udevcore</td><td>udevcore</td></tr><tr><td><strong>RPC URL</strong></td><td>https://full-node.mainnet-1.coreum.dev:26657</td><td>https://full-node.testnet-1.coreum.dev:26657</td><td>https://full-node.devnet-1.coreum.dev:26657</td><td>http://localhost:26657</td></tr><tr><td><strong>GRPC URL</strong></td><td>full-node.mainnet-1.coreum.dev:9090</td><td>full-node.testnet-1.coreum.dev:9090</td><td>full-node.devnet-1.coreum.dev:9090</td><td>localhost:9090</td></tr><tr><td><strong>REST URL</strong></td><td>https://full-node.mainnet-1.coreum.dev:1317</td><td>https://full-node.testnet-1.coreum.dev:1317</td><td>https://full-node.devnet-1.coreum.dev:1317</td><td>http://localhost:1317</td></tr><tr><td><strong>Cored version</strong></td><td>v2.0.2</td><td>v2.0.2</td><td>check the latest devnet release</td><td>already installed via crust</td></tr><tr><td><strong>Explorer API Beta</strong></td><td>https://hasura.mainnet-1.coreum.dev/v1/graphql</td><td>https://hasura.testnet-1.coreum.dev/v1/graphql</td><td>https://hasura.devnet-1.coreum.dev/v1/graphql</td><td>http://localhost:8080/v1/graphql</td></tr></tbody></table>

Set the chain environment (env) using the above values for the network you want to work with.&#x20;

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
