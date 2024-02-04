# Creating and Managing NFTs

This document gives instructions and examples on how to use our `pkg/client` package to create and manage non-fungible token.

### Complete code <a href="#complete-code" id="complete-code"></a>

Complete code with `go.mod` file you can find [here](https://github.com/CoreumFoundation/tutorials/tree/main/go/create-non-fungible-token)

P.S. If you have issues with `go mod tidy` command, just copy `go.mod` file from the example above.

### Go code skeleton <a href="#go-code-skeleton" id="go-code-skeleton"></a>

#### Imports and main function <a href="#imports-and-main-function" id="imports-and-main-function"></a>

Create standard `main.go` file containing this skeleton importing `pkg/client`:

{% code lineNumbers="true" %}
```sh
package main

import (
	"github.com/CoreumFoundation/coreum/v3/pkg/client"
    coreumconfig "github.com/CoreumFoundation/coreum/v3/pkg/config"
	"github.com/CoreumFoundation/coreum/v3/pkg/config/constant"
)

const (
	senderMnemonic = "" // put mnemonic here

	chainID          = constant.ChainIDTest
	addressPrefix    = constant.AddressPrefixTest
	denom            = constant.DenomTest
	recipientAddress = "testcore1534s8rz2e36lwycr6gkm9vpfe5yf67wkuca7zs"
	nodeAddress      = "full-node.testnet-1.coreum.dev:9090"
)

func main() {

}
```
{% endcode %}

#### Preparing test account <a href="#preparing-test-account" id="preparing-test-account"></a>

Before you may broadcast transactions, you need to have access to a funded account. Normally you would create a private key stored securely in local keystore. Here, for simplicity, we will use private key generated by our faucet. Never ever use mnemonic directly in code and never ever use key generated by faucet in production. It might cause complete funds loss! Please reference keyring documentation to learn on using keyring: https://docs.cosmos.network/v0.47/user/run-node/keyring and https://pkg.go.dev/github.com/cosmos/cosmos-sdk/crypto/keyring.

To get funded account, go to our faucet website: https://docs.coreum.dev/tools-ecosystem/faucet and click on "Generate Funded Wallet" button in "Testnet" section.

In response, you get your wallet address on our testnet chain and mnemonic used to generate the private key. Assign mnemonic to the constant `senderMnemonic` in the code snippet above.

#### Setting Cosmos SDK configuration <a href="#setting-cosmos-sdk-configuration" id="setting-cosmos-sdk-configuration"></a>

First we need to configure Cosmos SDK:

{% code lineNumbers="true" %}
```sh
config := sdk.GetConfig()
config.SetBech32PrefixForAccount(addressPrefix, addressPrefix+"pub")
config.SetCoinType(constant.CoinType)
config.Seal()
```
{% endcode %}

#### Preparing client context and tx factory <a href="#preparing-client-context-and-tx-factory" id="preparing-client-context-and-tx-factory"></a>

Before we are able to broadcast transaction, we must create and configure client context and tx factory:

{% code lineNumbers="true" %}
```sh
modules := module.NewBasicManager(
    auth.AppModuleBasic{},
)

// If you don't use TLS then replace `grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{MinVersion: tls.VersionTLS12}))` with `grpc.WithInsecure()`
grpcClient, err := grpc.Dial(nodeAddress, grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{MinVersion: tls.VersionTLS12})))
if err != nil {
    panic(err)
}

encodingConfig := coreumconfig.NewEncodingConfig(modules)

clientCtx := client.NewContext(client.DefaultContextConfig(), modules).
    WithChainID(string(chainID)).
    WithGRPCClient(grpcClient).
    WithKeyring(keyring.NewInMemory(encodingConfig.Codec)).
	WithBroadcastMode(flags.BroadcastSync)

txFactory := client.Factory{}.
    WithKeybase(clientCtx.Keyring()).
    WithChainID(clientCtx.ChainID()).
    WithTxConfig(clientCtx.TxConfig()).
    WithSimulateAndExecute(true)
```
{% endcode %}

#### Generate private key <a href="#generate-private-key" id="generate-private-key"></a>

To sign a transaction, private key generated from mnemonic stored in `senderMnemonic` is required. We store that key in the temporary keystore. In production you should use any keyring other than `memory` or `test`. Good choice might be `os` or `file`. For more details, refer keyring documentation: https://docs.cosmos.network/v0.47/user/run-node/keyring and https://pkg.go.dev/github.com/cosmos/cosmos-sdk/crypto/keyring.

{% code lineNumbers="true" %}
```sh
senderInfo, err := clientCtx.Keyring().NewAccount(
    "key-name",
    senderMnemonic,
    "",
    sdk.GetConfig().GetFullBIP44Path(),
    hd.Secp256k1,
)
if err != nil {
    panic(err)
}
```
{% endcode %}

### Creating NFT class <a href="#creating-nft-class" id="creating-nft-class"></a>

First we create class which is a container for a set of NFTs having the same purpose:

{% code lineNumbers="true" %}
```sh
senderAddress, err := senderInfo.GetAddress()
if err != nil {
	panic(err)
}
const classSymbol = "NFTClass"
msgIssueClass := &assetnfttypes.MsgIssueClass{
	Issuer:      senderAddress.String(),
	Symbol:      classSymbol,
	Name:        "NFT Class",
	Description: "NFT Class",
	Features:    []assetnfttypes.ClassFeature{assetnfttypes.ClassFeature_freezing},
}

ctx := context.Background()
_, err = client.BroadcastTx(
	ctx,
	clientCtx.WithFromAddress(senderAddress),
	txFactory,
	msgIssueClass,
)
if err != nil {
	panic(err)
}
```
{% endcode %}

### Minting NFT <a href="#minting-nft" id="minting-nft"></a>

Then we mint new NFT for that class:

{% code lineNumbers="true" %}
```sh
classID := assetnfttypes.BuildClassID(classSymbol, senderAddress)
const nftID = "myNFT"
msgMint := &assetnfttypes.MsgMint{
	Sender:  senderAddress.String(),
	ClassID: classID,
	ID:      nftID,
}

_, err = client.BroadcastTx(
	ctx,
	clientCtx.WithFromAddress(senderAddress),
	txFactory,
	msgMint,
)
if err != nil {
	panic(err)
}
```
{% endcode %}

### Querying the owner <a href="#querying-the-owner" id="querying-the-owner"></a>

We query the owner of the NFT to verify that it is owned by the creator:

{% code lineNumbers="true" %}
```sh
nftClient := nft.NewQueryClient(clientCtx)
resp, err := nftClient.Owner(ctx, &nft.QueryOwnerRequest{
	ClassId: classID,
	Id:      nftID,
})
if err != nil {
	panic(err)
}
fmt.Printf("Owner: %s\n", resp.Owner)
```
{% endcode %}

### Sending NFT <a href="#sending-nft" id="sending-nft"></a>

Now we send NFT to someone else:

{% code lineNumbers="true" %}
```sh
recipientInfo, _, err := clientCtx.Keyring().NewMnemonic(
	"recipient",
	keyring.English,
	sdk.GetConfig().GetFullBIP44Path(),
	"",
	hd.Secp256k1,
)
if err != nil {
	panic(err)
}

recipientAddress, err := recipientInfo.GetAddress()
if err != nil {
	panic(err)
}
msgSend := &nft.MsgSend{
	Sender:   senderAddress.String(),
	Receiver: recipientAddress.String(),
	Id:       nftID,
	ClassId:  classID,
}

_, err = client.BroadcastTx(
	ctx,
	clientCtx.WithFromAddress(senderAddress),
	txFactory,
	msgSend,
)
if err != nil {
	panic(err)
}
```
{% endcode %}

Let's verify that recipient is the owner now:

{% code lineNumbers="true" %}
```sh
resp, err = nftClient.Owner(ctx, &nft.QueryOwnerRequest{
	ClassId: classID,
	Id:      nftID,
})
if err != nil {
	panic(err)
}
fmt.Printf("Owner: %s\n", resp.Owner)
```
{% endcode %}

### Freezing <a href="#freezing" id="freezing"></a>

Because issuer enabled `freezing` feature during class issuance, he/she might freeze the NFT:

{% code lineNumbers="true" %}
```sh
msgFreeze := &assetnfttypes.MsgFreeze{
	Sender:  senderAddress.String(),
	ClassID: classID,
	ID:      nftID,
}

_, err = client.BroadcastTx(
	ctx,
	clientCtx.WithFromAddress(senderAddress),
	txFactory,
	msgFreeze,
)
if err != nil {
	panic(err)
}
```
{% endcode %}

After doing this, recipient is not allowed to transfer the NFT from its account.

All the other features may be used in a similar fashion. More info is available in [the documentation](https://docs.coreum.dev/modules/assetnft.html)