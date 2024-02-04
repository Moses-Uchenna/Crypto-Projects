# Explorer API - Beta

The tutorial provides the instruction on how to use the `Explorer API`.

### Endpoints <a href="#endpoints" id="endpoints"></a>

The primary hasura API is GraphQL API. All GraphQL Query endpoints are public and available for the usage. The list of supported `Explorer APIs` can be found on the page [network-variables.md](https://docs.coreum.dev/tutorials/network-variables.html). For the production usage we advise to deploy your own `Explorer Backend`. The source code can be found [here](https://github.com/CoreumFoundation/bdjuno).

### GraphQL Schema <a href="#graphql-schema" id="graphql-schema"></a>

The GraphQL schema is located [here](https://github.com/CoreumFoundation/bdjuno/tree/chains/coreum/hasura/api/schema.graphql). It describes all supported queries and can be used for the client code generation.

### GraphQL Playground <a href="#graphql-playground" id="graphql-playground"></a>

To run and play with the GraphQL represented as a UI you need:

* Install the [graphql-playground](https://github.com/graphql/graphql-playground).
* Create a folder and copy the [schema.graphql](https://github.com/CoreumFoundation/bdjuno/tree/chains/coreum/hasura/api/schema.graphql) file there.
* Create new file `.graphqlconfig` in the same folder with the content

```graphql
{
  "name": "Explorer Schema",
  "schemaPath": "schema.graphql",
  "extensions": {
    "endpoints": {
      "Remote SWAPI GraphQL Endpoint": {
        "url": " https://hasura.testnet-1.coreum.dev/v1/graphql",
        "headers": {
          "user-agent": "JS GraphQL"
        },
        "introspect": true
      }
    }
  }
}
```

* Run the `graphql-playground` and chose the `.graphqlconfig` as config.
* Execute the query

To get transactions by address, use that query

```graphql
{
    messages_by_address(args: {addresses: "{testcore1sxu4sumja8c53gvyn7cctqlsqm27w6jh0dnvdm}", limit: "50", offset: "0", types: "{}"}) {
        value
        type
        transaction_hash
    }
}
```

The same request can be represented as a raw request:

```
curl --location 'https://hasura.testnet-1.coreum.dev/v1/graphql' \
--header 'Content-Type: text/plain' \
--data '{"operationName":"GetMessagesByAddress","variables":{"limit":50,"offset":0,"types":"{}","address":"{testcore1sxu4sumja8c53gvyn7cctqlsqm27w6jh0dnvdm}"},"query":"query GetMessagesByAddress($address: _text, $limit: bigint = 50, $offset: bigint = 0, $types: _text = \"{}\") {\n  messagesByAddress: messages_by_address(\n    args: {addresses: $address, types: $types, limit: $limit, offset: $offset}\n  ) {\n    transaction {\n      height\n      hash\n      success\n      messages\n      logs\n      block {\n        height\n        timestamp\n        __typename\n      }\n      __typename\n    }\n    __typename\n  }\n}\n"}'
```

Pay attention that examples are for the `testnet` network, so if you want to use different you need to update the address for the corresponding network.
