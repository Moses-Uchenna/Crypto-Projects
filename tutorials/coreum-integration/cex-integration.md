# CEX Integration

This document provides the information required by exchanges to integrate the trading of CORE.

## Links <a href="#links" id="links"></a>

* [Official website](https://www.coreum.com/)
* [Source code](https://github.com/CoreumFoundation/coreum)
* [Block explorers](https://docs.coreum.dev/tools-ecosystem/block-explorer.html)
* [Documentation](https://docs.coreum.dev/)
* [Faucet](https://docs.coreum.dev/tools-ecosystem/faucet.html)

## Project information <a href="#project-information" id="project-information"></a>

* Project name: `Coreum`
* Token name: `Coreum`
* Token symbol: `COREUM`
* Token decimal (precision): `6 digits`

### Initial and total supply <a href="#initial-and-total-supply" id="initial-and-total-supply"></a>

Initial supply: `500,000,000 COREs`. It is set in genesis block.

Total supply will grow, there is no ceiling. Inflation is proportional to the difference between current TVL and our target TVL (67%). Initial inflation is set to 10%, maximum inflation is 20%, minimum inflation is 0% (when TVL is equal to or higher than 67%).

### Functions of CORE token <a href="#functions-of-core-token" id="functions-of-core-token"></a>

* staking and delegating to the validators
* voting
* paying fees

### CORE allocation plan <a href="#core-allocation-plan" id="core-allocation-plan"></a>

* SOLO community airdrop: 20%
* CORE community airdrop: 30%
* Validators' reward pool: 10%
* dApp developers: 10%
* Operation (maintenance, teams, investors): 30%

### Swap <a href="#swap" id="swap"></a>

We swap from token present on XRP Ledger chain. This process does not require any special action taken by the exchanges. There is the bridge used to convert tokens present on XRPL to tokens present on our mainnet. Exchanges should care only about tokens on our mainnet.

### Transaction model <a href="#transaction-model" id="transaction-model"></a>

We use the Account Model where each account holds the balance of COREs. It is implemented by standard Bank module of Cosmos SDK.

### Consensus <a href="#consensus" id="consensus"></a>

* algorithm: BPoS (Bonded Proof of Stake), implemented by Tendermint.
* block time: 1.6 seconds, not fixed
* number of confirmations required: 1

### Base implementation <a href="#base-implementation" id="base-implementation"></a>

Based on Cosmos SDK, with added custom functionalities. Transferring COREs work the same way like in standard bank module provided by Cosmos SDK

### Validators and voting <a href="#validators-and-voting" id="validators-and-voting"></a>

There are 32 validators. Voting power of each validator is proportional to its stake.

### Dividends and/or interests <a href="#dividends-and-or-interests" id="dividends-and-or-interests"></a>

Validators get rewards for validating blocks. They also earn commission on stake delegated to them by other accounts.

### Smart contract <a href="#smart-contract" id="smart-contract"></a>

Our chain supports WASM smart contracts by integrating CosmWASM module.

### Token issuance <a href="#token-issuance" id="token-issuance"></a>

Coreum blockchain supports token issuance. It also may receive tokens from other chains by using IBC protocol.

### Account activation <a href="#account-activation" id="account-activation"></a>

Account does not require activation. Its existence starts when it receives any funds.

### Signatures <a href="#signatures" id="signatures"></a>

* We support multi-signature but only for amino-encoded transactions which are legacy.
* It is possible to sign transactions in offline mode and broadcast later.
* Each account maintains an incremental sequence number. A valid, expected sequence number must be included in a transaction. Otherwise the transaction will fail.

## Transactions <a href="#transactions" id="transactions"></a>

Transactions never expire. They are executed in FIFO order. We don't support delayed transactions.

Every transaction executed by the account changes its balance due to fees or implemented logic. Those transactions require a private key to sign them so only the exchange may create them.

There are also messages which might change the balance of the account without its interaction (account receives some funds):

* `bank.MsgMultiSend`
* `bank.MsgSend`
* `authz.MsgExec`
* `distribution.MsgWithdrawDelegatorReward`
* `distribution.MsgWithdrawValidatorCommission`
* `ibc.MsgTransfer`
* `vesting.MsgCreateVestingAccount`
* `wasm.MsgExecuteContract`

The transaction included in a block might fail. It is indicated by a non-zero status code of the transaction. Keep in mind that in this case fee is charged anyway.

### Fees <a href="#fees" id="fees"></a>

To understand Coreum chain fees read about the [coreum gas price](https://docs.coreum.dev/tutorials/gas-price.html).

### Coinbase transactions <a href="#coinbase-transactions" id="coinbase-transactions"></a>

Blocks don't start with BTC-like coinbase transaction.

### Audit <a href="#audit" id="audit"></a>

Our code is being audited. Once the process is completed we will share the results.

### Code examples <a href="#code-examples" id="code-examples"></a>

Here we present examples on how to connect to our chain, broadcast transactions and query the state. Examples are written in Golang and TypeScript.

### Preparing test account <a href="#preparing-test-account" id="preparing-test-account"></a>

Before you may broadcast transactions, you need to have access to a funded account. Normally you would create a private key stored securely in keystore. Here, for simplicity, we will use the private key generated by our faucet. Never ever use mnemonic directly in code and never ever use key generated by faucet in production. It might cause complete funds loss! Please reference keyring documentation to learn how to store your keys securely using keyring: https://docs.cosmos.network/v0.47/user/run-node/keyring and https://pkg.go.dev/github.com/cosmos/cosmos-sdk/crypto/keyring.

To get funded account, go to our faucet website: https://docs.coreum.dev/tools-ecosystem/faucet and click on "Generate Funded Wallet" button in "Testnet" section.

In response, you get your wallet address on our testnet chain and mnemonic used to generate the private key. Assign mnemonic to the constant `senderMnemonic` in the code snippets below.

### Skeleton <a href="#skeleton" id="skeleton"></a>

#### **Golang**

Create standard `main.go` file containing this skeleton importing `pkg/client`:

{% code lineNumbers="true" %}
```sh
package main

import (
	"github.com/CoreumFoundation/coreum/v3/pkg/client"
	"github.com/CoreumFoundation/coreum/pkg/config/constant"
)

func main() {
	const (
		walletMnemonic = "" // put mnemonic here

		chainID          = constant.ChainIDTest
		addressPrefix    = constant.AddressPrefixTest
		denom            = constant.DenomTest
		recipientAddress = "testcore1534s8rz2e36lwycr6gkm9vpfe5yf67wkuca7zs"
		nodeAddress      = "full-node.testnet-1.coreum.dev:9090"
	)
}
```
{% endcode %}

#### **TypeScript**

Initialize new project:

```sh
npm init
```

Install required dependencies:

```sh
npm install @cosmjs/proto-signing @cosmjs/stargate typescript
```

Create file `main.ts` containing the skeleton:

{% code lineNumbers="true" %}
```sh
import { StdFee } from "@cosmjs/amino";
import { stringToPath } from "@cosmjs/crypto";
import {DirectSecp256k1HdWallet, AccountData, parseCoins, Coin} from "@cosmjs/proto-signing";
import {
    calculateFee,
    GasPrice,
    SigningStargateClient,
    DeliverTxResponse,
} from "@cosmjs/stargate";
import {IndexedTx, isDeliverTxSuccess} from "@cosmjs/stargate/build/stargateclient";
import { Event, Attribute } from "@cosmjs/stargate";

const coreumAccountPrefix = "testcore"; // the address prefix (different for different chains/environments)
const coreumHDPath = "m/44'/990'/0'/0/0"; // coreum HD path (same for all chains/environments)
const coreumDenom = "utestcore"; // core denom (different for different chains/environments)
const coreumRpcEndpoint = "https://full-node.testnet-1.coreum.dev:26657"; // rpc endpoint (different for different chains/environments)
const recipientAddress = "testcore1534s8rz2e36lwycr6gkm9vpfe5yf67wkuca7zs"
const senderMnemonic = ""; // put mnemonic here

const main = (async function() {

})();

export default main;
```
{% endcode %}

#### Configure the client <a href="#configure-the-client" id="configure-the-client"></a>

To sign a transaction, or use standard bank module operations you need to set up the new client.

To sign a transaction, private key generated from mnemonic stored in `senderMnemonic` is required. We generate the private key on the fly. In production you should keep the generated private key in safe location and not store the mnemonic inside the source code.

#### **Golang**

{% code lineNumbers="true" %}
```sh
config := sdk.GetConfig()
config.SetBech32PrefixForAccount(addressPrefix, addressPrefix+"pub")
config.SetCoinType(constant.CoinType)
config.Seal()
	
modules := module.NewBasicManager(
    auth.AppModuleBasic{},
)

// If you don't use TLS then replace `grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{MinVersion: tls.VersionTLS12}))` with `grpc.WithInsecure()`
grpcClient, err := grpc.Dial(nodeAddress, grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{MinVersion: tls.VersionTLS12})))
if err != nil {
    panic(err)
}

encodingConfig := coreumconfig.NewEncodingConfig(modules)

clientCtxConfig := client.DefaultContextConfig()
// These are default values, but we provide them explicitly so users are aware that those settings exist.
clientCtxConfig.GasConfig.GasAdjustment = 1.0
clientCtxConfig.GasConfig.GasPriceAdjustment = sdk.MustNewDecFromStr("1.1")
clientCtx := client.NewContext(clientCtxConfig, modules).
    WithChainID(string(chainID)).
    WithGRPCClient(grpcClient).
    WithKeyring(keyring.NewInMemory(encodingConfig.Codec)).
	WithBroadcastMode(flags.BroadcastSync)

txFactory := client.Factory{}.
    WithKeybase(clientCtx.Keyring()).
    WithChainID(clientCtx.ChainID()).
    WithTxConfig(clientCtx.TxConfig()).
    WithSimulateAndExecute(true)

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

fmt.Println(senderInfo.GetAddress().String())

ctx := context.Background()
```
{% endcode %}

**TypeScript**

{% code lineNumbers="true" %}
```sh
console.log("preparing sender wallet");
const senderWallet = await DirectSecp256k1HdWallet.fromMnemonic(senderMnemonic, {
    prefix: coreumAccountPrefix,
    hdPaths: [stringToPath(coreumHDPath)],
});
const [sender] = await senderWallet.getAccounts();
console.log(`sender address: ${sender.address}`);

const senderClient = await SigningStargateClient.connectWithSigner(
    coreumRpcEndpoint,
    senderWallet
);
```
{% endcode %}

#### Send coins <a href="#send-coins" id="send-coins"></a>

Now we are ready to broadcast transaction. As an example we send `9000000utestcore` tokens from `sender` wallet to `recipient`. After executing the transaction you may copy transaction hash and paste it in the search box of our block explorer (https://explorer.testnet-1.coreum.dev/coreum) to confirm the transaction execution and check its properties.

**Golang**

{% code lineNumbers="true" %}
```sh
// Validate addresses
senderAddr, err := sdk.AccAddressFromBech32(sender)
if err != nil {
    return "", err
}
if _, err := sdk.AccAddressFromBech32(recipient); err != nil {
    return "", err
}

// Broadcast transaction transferring funds
msg := &banktypes.MsgSend{
    FromAddress: sender,
    ToAddress:   recipient,
    Amount:      sdk.NewCoins(amount),
}

result, err := client.BroadcastTx(
    ctx,
    clientCtx.WithFromAddress(senderAddr),
    txFactory,
    msg,
)
if err != nil {
    return "", err
}
fmt.Printf("Tx hash: %s\n", result.TxHash)

return result.TxHash, nil
```
{% endcode %}

#### Output:

```
Tx hash: 8C694A92A2208DB8CE733D54C22A3C7F945D54867B9078D08686DC7DBF0F44DC
```

**TypeScript**

{% code lineNumbers="true" %}
```sh
console.log(`sending ${amount.amount} to recipient:`);
// Initial gas price is hardcoded for now here, because client doesn't support querying for current gas price.
const gasPrice = GasPrice.fromString(`0.0625${coreumDenom}`);
// TODO: USe gas estimation once https://github.com/CoreumFoundation/coreum/pull/403 is merged and released to testnet
const singleBankSendGas = 111_000;
const bankSendFee: StdFee = calculateFee(singleBankSendGas, gasPrice);
const bankSendResult = await senderClient.sendTokens(
    sender,
    recipient,
    [amount],
    bankSendFee
);
isDeliverTxSuccess(bankSendResult);
console.log(`successfully sent, tx hash: ${bankSendResult.transactionHash}`);
```
{% endcode %}

#### Output:

```
successfully sent, tx hash: 197916E8FBE5E1CFF93A5C54BD2C149E4BED6F216ABBCFA740E2735629E9E6A9
```

### Querying the balance <a href="#querying-the-balance" id="querying-the-balance"></a>

Now you may query the balance of your account.

#### **Golang**

{% code lineNumbers="true" %}
```sh
bankClient := banktypes.NewQueryClient(clientCtx)
balances, err := bankClient.AllBalances(ctx, &banktypes.QueryAllBalancesRequest{
    Address: address,
})
if err != nil {
    return err
}
fmt.Printf("Balances: %s\n", balances.Balances)

return nil
```
{% endcode %}

#### Output:

```
Balances: 9000000utestcore
```

#### **TypeScript**

**Code:**

{% code lineNumbers="true" %}
```sh
const balance = await senderClient.getBalance(address, coreumDenom);
console.log(`balance: ${balance.amount}`);
```
{% endcode %}

#### Output:

```
balance: 9000000utestcore
```

### Querying the latest block <a href="#querying-the-latest-block" id="querying-the-latest-block"></a>

#### **Golang**

**Code:**

{% code lineNumbers="true" %}
```shell
tmClient := tmservice.NewServiceClient(clientCtx)
latestBlock, err := tmClient.GetLatestBlock(ctx, &tmservice.GetLatestBlockRequest{})
if err != nil {
    return err
}
fmt.Printf("Latest block:\n%+v\n", latestBlock.Block.Header)
```
{% endcode %}

#### Output:

```sh
Latest block:
{Version:{Block:11 App:0} ChainID:coreum-testnet-1 Height:514698 Time:2023-02-09 08:41:27.633469239 +0000 UTC LastBlockId:{Hash:[183 68 105 60 2 100 116 232 232 78 209 210 127 226 111 200 6 139 142 20 246 83 17 200 70 131 24 106 111 214 109 67] PartSetHeader:{Total:1 Hash:[64 86 200 156 134 226 62 253 121 109 77 250 78 228 243 104 77 133 82 42 233 63 123 195 235 8 102 90 17 239 156 168]}} LastCommitHash:[162 163 235 199 125 217 181 58 101 90 244 190 254 55 18 255 74 204 61 50 4 191 213 64 160 221 124 80 27 154 215 37] DataHash:[85 177 235 42 15 19 222 173 164 224 203 108 20 39 5 99 60 176 193 15 178 153 84 173 45 236 7 134 147 181 49 21] ValidatorsHash:[87 36 242 74 140 244 140 232 251 130 220 161 226 183 56 16 107 93 158 72 78 157 201 97 104 170 33 141 124 55 9 119] NextValidatorsHash:[87 36 242 74 140 244 140 232 251 130 220 161 226 183 56 16 107 93 158 72 78 157 201 97 104 170 33 141 124 55 9 119] ConsensusHash:[34 216 212 200 185 84 158 169 2 198 106 26 24 247 36 248 247 151 11 142 76 95 30 179 176 181 241 180 225 222 5 49] AppHash:[114 54 187 74 194 75 37 81 6 227 116 106 110 226 84 193 232 85 38 115 61 95 144 226 128 7 115 23 246 31 137 142] LastResultsHash:[227 176 196 66 152 252 28 20 154 251 244 200 153 111 185 36 39 174 65 228 100 155 147 76 164 149 153 27 120 82 184 85] EvidenceHash:[227 176 196 66 152 252 28 20 154 251 244 200 153 111 185 36 39 174 65 228 100 155 147 76 164 149 153 27 120 82 184 85] ProposerAddress:[218 151 165 124 109 33 146 94 158 5 239 124 209 81 182 35 69 4 148 16]}
```

**TypeScript**

**Code:**

{% code lineNumbers="true" %}
```sh
const latestBlock = await senderClient.getBlock();
console.log(`latest block: `, latestBlock.header);
```
{% endcode %}

#### Output:

{% code lineNumbers="true" %}
```sh
latest block:  {
  version: { block: '11', app: '0' },
  height: 730020,
  chainId: 'coreum-testnet-1',
  time: '2023-02-13T12:11:43.761122499Z'
}
```
{% endcode %}

### Querying block by height <a href="#querying-block-by-height" id="querying-block-by-height"></a>

#### **Golang**

**Code:**

{% code lineNumbers="true" %}
```sh
block, err := tmClient.GetBlockByHeight(ctx, &tmservice.GetBlockByHeightRequest{Height: latestBlock.Block.Header.Height})
if err != nil {
    return err
}
fmt.Printf("Block:\n%+v\n", block.Block.Header)
```
{% endcode %}

#### Output:

```sh
Block:
{Version:{Block:11 App:0} ChainID:coreum-testnet-1 Height:514698 Time:2023-02-09 08:41:27.633469239 +0000 UTC LastBlockId:{Hash:[183 68 105 60 2 100 116 232 232 78 209 210 127 226 111 200 6 139 142 20 246 83 17 200 70 131 24 106 111 214 109 67] PartSetHeader:{Total:1 Hash:[64 86 200 156 134 226 62 253 121 109 77 250 78 228 243 104 77 133 82 42 233 63 123 195 235 8 102 90 17 239 156 168]}} LastCommitHash:[162 163 235 199 125 217 181 58 101 90 244 190 254 55 18 255 74 204 61 50 4 191 213 64 160 221 124 80 27 154 215 37] DataHash:[85 177 235 42 15 19 222 173 164 224 203 108 20 39 5 99 60 176 193 15 178 153 84 173 45 236 7 134 147 181 49 21] ValidatorsHash:[87 36 242 74 140 244 140 232 251 130 220 161 226 183 56 16 107 93 158 72 78 157 201 97 104 170 33 141 124 55 9 119] NextValidatorsHash:[87 36 242 74 140 244 140 232 251 130 220 161 226 183 56 16 107 93 158 72 78 157 201 97 104 170 33 141 124 55 9 119] ConsensusHash:[34 216 212 200 185 84 158 169 2 198 106 26 24 247 36 248 247 151 11 142 76 95 30 179 176 181 241 180 225 222 5 49] AppHash:[114 54 187 74 194 75 37 81 6 227 116 106 110 226 84 193 232 85 38 115 61 95 144 226 128 7 115 23 246 31 137 142] LastResultsHash:[227 176 196 66 152 252 28 20 154 251 244 200 153 111 185 36 39 174 65 228 100 155 147 76 164 149 153 27 120 82 184 85] EvidenceHash:[227 176 196 66 152 252 28 20 154 251 244 200 153 111 185 36 39 174 65 228 100 155 147 76 164 149 153 27 120 82 184 85] ProposerAddress:[218 151 165 124 109 33 146 94 158 5 239 124 209 81 182 35 69 4 148 16]}
```

#### **TypeScript**

**Code:**

{% code lineNumbers="true" %}
```sh
const block = await senderClient.getBlock(latestBlock.header.height);
console.log(`block: `, block.header);
```
{% endcode %}

#### Output:

{% code lineNumbers="true" %}
```sh
block:  {
  version: { block: '11', app: '0' },
  height: 730126,
  chainId: 'coreum-testnet-1',
  time: '2023-02-13T12:14:41.672493387Z'
}
```
{% endcode %}

### Querying transaction by hash <a href="#querying-transaction-by-hash" id="querying-transaction-by-hash"></a>

#### **Golang**

Code:

{% code lineNumbers="true" %}
```sh
txClient := sdktx.NewServiceClient(clientCtx)

tx, err := txClient.GetTx(ctx, &sdktx.GetTxRequest{Hash: txHash})
if err != nil {
    return nil, err
}
fmt.Printf("Tx:\n%+v\n", tx.TxResponse)
```
{% endcode %}

#### Output:

```
Tx:
code: 0
codespace: ""
data: 0A1E0A1C2F636F736D6F732E62616E6B2E763162657461312E4D736753656E64
...
```

**TypeScript**

**Code:**

{% code lineNumbers="true" %}
```sh
const tx = await senderClient.getTx(txHash);
console.log(`tx: `, tx);
```
{% endcode %}

#### Output:

```
tx:  {
  height: 730242,
  hash: 'A7C99B4B7374C9B892577A2943FE8D18D102B036EF988615188E54DB4535F059',
  code: 0,
  events: [
    { type: 'coin_spent', attributes: [Array] },
    { type: 'coin_received', attributes: [Array] },
    { type: 'transfer', attributes: [Array] },
...
```

### Check if Transaction Succeeded <a href="#check-if-transaction-succeeded" id="check-if-transaction-succeeded"></a>

#### **Golang**

**Code:**

```sh
fmt.Println(tx.TxResponse.Code == 0)
```

#### Output:

```
true
```

#### **TypeScript**

**Code:**

```sh
console.log(tx.code === 0);
```

#### Output:

```
true
```

### Detecting balance changes <a href="#detecting-balance-changes" id="detecting-balance-changes"></a>

Whenever account receives or spends tokens, event is generated by the transaction. So to get information about balance updates it is needed to search for those events in all the transactions in all the incoming blocks:

**Golang**

{% code lineNumbers="true" %}
```sh
for _, event := range tx.TxResponse.Events {
    switch event.Type {
    case "coin_received":
        var receiver string
        var amount string
        for _, attr := range event.Attributes {
            switch string(attr.Key) {
            case "receiver":
                receiver = string(attr.Value)
            case "amount":
                amount = string(attr.Value)
            }
        }
        coins, err := sdk.ParseCoinsNormalized(amount)
        if err != nil {
            panic(err)
        }
        denomAmount := coins.AmountOf(denom)
        if denomAmount.IsZero() {
            continue
        }
        fmt.Printf("%s received %s\n", receiver, sdk.NewCoin(denom, denomAmount))
    case "coin_spent":
        var spender string
        var amount string
        for _, attr := range event.Attributes {
            switch string(attr.Key) {
            case "spender":
                spender = string(attr.Value)
            case "amount":
                amount = string(attr.Value)
            }
        }
        coins, err := sdk.ParseCoinsNormalized(amount)
        if err != nil {
            panic(err)
        }
        denomAmount := coins.AmountOf(denom)
        if denomAmount.IsZero() {
            continue
        }
        fmt.Printf("%s spent %s\n", spender, sdk.NewCoin(denom, denomAmount))
    }
}
```
{% endcode %}

#### Output:

```
testcore1zuelfk5fz02v9x7gnsy2t7ps83m8vljx5wqdfq spent 2544utestcore
testcore17xpfvakm2amg962yls6f84z3kell8c5l4rqxrs received 2544utestcore
testcore1zuelfk5fz02v9x7gnsy2t7ps83m8vljx5wqdfq spent 9000000utestcore
testcore1534s8rz2e36lwycr6gkm9vpfe5yf67wkuca7zs received 9000000utestcore
```

**TypeScript**

{% code lineNumbers="true" %}
```sh
tx.events.forEach((event: Event) => {
    switch (event.type) {
        case 'coin_received':
            var receiver: string;
            var amount: string;
            event.attributes.forEach((attr: Attribute) => {
                switch (attr.key) {
                    case "receiver":
                        receiver = attr.value;
                        break;
                    case "amount":
                        amount = attr.value;
                        break;
                }
            });

            parseCoins(amount).forEach((coin: Coin) => {
                if (coin.denom != coreumDenom) {
                    return;
                }

                console.log(`${receiver} received ${coin.amount}${coin.denom}`);
            });
            break;
        case 'coin_spent':
            var spender: string;
            var amount: string;
            event.attributes.forEach((attr: Attribute) => {
                switch (attr.key) {
                    case "spender":
                        spender = attr.value;
                        break;
                    case "amount":
                        amount = attr.value;
                        break;
                }
            });

            parseCoins(amount).forEach((coin: Coin) => {
                if (coin.denom != coreumDenom) {
                    return;
                }

                console.log(`${spender} spent ${coin.amount}${coin.denom}`);
            })
            break;
    }
});
```
{% endcode %}

#### **Output:**

```
testcore1ucyj07jehc7xu9h9v3dhfh4ssv0vp9a2wvp9pq spent 6938utestcore
testcore17xpfvakm2amg962yls6f84z3kell8c5l4rqxrs received 6938utestcore
testcore1ucyj07jehc7xu9h9v3dhfh4ssv0vp9a2wvp9pq spent 9000000utestcore
testcore1ecngv3mg38r4wxr8qxzu6uzyptaa79as2gzp4u received 9000000utestcore
```

### Complete code <a href="#complete-code" id="complete-code"></a>

Here is the complete code listing with all the features implemented above.

#### **Golang**

{% code lineNumbers="true" %}
```sh
package main

import (
	"context"
	"crypto/tls"
	"fmt"

	"github.com/cosmos/cosmos-sdk/client/flags"
	"github.com/cosmos/cosmos-sdk/client/grpc/tmservice"
	"github.com/cosmos/cosmos-sdk/crypto/hd"
	"github.com/cosmos/cosmos-sdk/crypto/keyring"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/types/module"
	sdktx "github.com/cosmos/cosmos-sdk/types/tx"
	"github.com/cosmos/cosmos-sdk/x/auth"
	banktypes "github.com/cosmos/cosmos-sdk/x/bank/types"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"

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
	// Configure Cosmos SDK
	config := sdk.GetConfig()
	config.SetBech32PrefixForAccount(addressPrefix, addressPrefix+"pub")
	config.SetCoinType(constant.CoinType)
	config.Seal()

	// List required modules.
	// If you need types from any other module import them and add here.
	modules := module.NewBasicManager(
		auth.AppModuleBasic{},
	)

	// Configure client context and tx factory
	// If you don't use TLS then replace `grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{MinVersion: tls.VersionTLS12}))` with `grpc.WithInsecure()`
    grpcClient, err := grpc.Dial(nodeAddress, grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{MinVersion: tls.VersionTLS12})))
	if err != nil {
		panic(err)
	}

    encodingConfig := coreumconfig.NewEncodingConfig(modules)

	clientCtxConfig := client.DefaultContextConfig()
	// These are default values, but we provide them explicitly so users are aware that those settings exist.
	clientCtxConfig.GasConfig.GasAdjustment = 1.0
	clientCtxConfig.GasConfig.GasPriceAdjustment = sdk.MustNewDecFromStr("1.1")
	clientCtx := client.NewContext(clientCtxConfig, modules).
		WithChainID(string(chainID)).
		WithGRPCClient(grpcClient).
        WithKeyring(keyring.NewInMemory(encodingConfig.Codec)).
		WithBroadcastMode(flags.BroadcastSync)

	txFactory := client.Factory{}.
		WithKeybase(clientCtx.Keyring()).
		WithChainID(clientCtx.ChainID()).
		WithTxConfig(clientCtx.TxConfig()).
		WithSimulateAndExecute(true)

	// Generate private key and add it to the keystore
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

	fmt.Println(senderInfo.GetAddress().String())

	ctx := context.Background()

	txHash, err := sendCoins(
		ctx, clientCtx, txFactory,
		senderInfo.GetAddress().String(), recipientAddress,
		sdk.NewInt64Coin(denom, 9_000_000),
	)
	if err != nil {
		panic(err)
	}

	err = queryBalance(ctx, clientCtx, recipientAddress)
	if err != nil {
		panic(err)
	}

	err = queryBlock(ctx, clientCtx)
	if err != nil {
		panic(err)
	}

	tx, err := queryTx(ctx, clientCtx, txHash)
	if err != nil {
		panic(err)
	}

	balanceUpdates(tx)
}

func sendCoins(
	ctx context.Context,
	clientCtx client.Context,
	txFactory client.Factory,
	sender, recipient string,
	amount sdk.Coin,
) (string, error) {
	// Validate addresses
	senderAddr, err := sdk.AccAddressFromBech32(sender)
	if err != nil {
		return "", err
	}
	if _, err := sdk.AccAddressFromBech32(recipient); err != nil {
		return "", err
	}

	// Broadcast transaction transferring funds
	msg := &banktypes.MsgSend{
		FromAddress: sender,
		ToAddress:   recipient,
		Amount:      sdk.NewCoins(amount),
	}

	result, err := client.BroadcastTx(
		ctx,
		clientCtx.WithFromAddress(senderAddr),
		txFactory,
		msg,
	)
	if err != nil {
		return "", err
	}
	fmt.Printf("Tx hash: %s\n", result.TxHash)

	return result.TxHash, nil
}

func queryBalance(ctx context.Context, clientCtx client.Context, address string) error {
	bankClient := banktypes.NewQueryClient(clientCtx)
	balances, err := bankClient.AllBalances(ctx, &banktypes.QueryAllBalancesRequest{
		Address: address,
	})
	if err != nil {
		return err
	}
	fmt.Printf("Balances: %s\n", balances.Balances)

	return nil
}

func queryBlock(ctx context.Context, clientCtx client.Context) error {
	// Query latest block
	tmClient := tmservice.NewServiceClient(clientCtx)
	latestBlock, err := tmClient.GetLatestBlock(ctx, &tmservice.GetLatestBlockRequest{})
	if err != nil {
		return err
	}
	fmt.Printf("Latest block:\n%+v\n", latestBlock.Block.Header)

	// Query block by height
	block, err := tmClient.GetBlockByHeight(ctx, &tmservice.GetBlockByHeightRequest{Height: latestBlock.Block.Header.Height})
	if err != nil {
		return err
	}
	fmt.Printf("Block:\n%+v\n", block.Block.Header)

	return nil
}

func queryTx(ctx context.Context, clientCtx client.Context, txHash string) (*sdktx.GetTxResponse, error) {
	// Query tx by hash
	txClient := sdktx.NewServiceClient(clientCtx)

	tx, err := txClient.GetTx(ctx, &sdktx.GetTxRequest{Hash: txHash})
	if err != nil {
		return nil, err
	}
	fmt.Printf("Tx:\n%+v\n", tx.TxResponse)

	// Check if transaction succeeded
	fmt.Println(tx.TxResponse.Code == 0)

	return tx, nil
}

func balanceUpdates(tx *sdktx.GetTxResponse) {
	// Detecting balance changes
	for _, event := range tx.TxResponse.Events {
		switch event.Type {
		case "coin_received":
			var receiver string
			var amount string
			for _, attr := range event.Attributes {
				switch string(attr.Key) {
				case "receiver":
					receiver = string(attr.Value)
				case "amount":
					amount = string(attr.Value)
				}
			}
			coins, err := sdk.ParseCoinsNormalized(amount)
			if err != nil {
				panic(err)
			}
			denomAmount := coins.AmountOf(denom)
			if denomAmount.IsZero() {
				continue
			}
			fmt.Printf("%s received %s\n", receiver, sdk.NewCoin(denom, denomAmount))
		case "coin_spent":
			var spender string
			var amount string
			for _, attr := range event.Attributes {
				switch string(attr.Key) {
				case "spender":
					spender = string(attr.Value)
				case "amount":
					amount = string(attr.Value)
				}
			}
			coins, err := sdk.ParseCoinsNormalized(amount)
			if err != nil {
				panic(err)
			}
			denomAmount := coins.AmountOf(denom)
			if denomAmount.IsZero() {
				continue
			}
			fmt.Printf("%s spent %s\n", spender, sdk.NewCoin(denom, denomAmount))
		}
	}
}
```
{% endcode %}

#### **TypeScript**

{% code lineNumbers="true" %}
```sh
import { StdFee } from "@cosmjs/amino";
import { stringToPath } from "@cosmjs/crypto";
import {DirectSecp256k1HdWallet, AccountData, parseCoins, Coin} from "@cosmjs/proto-signing";
import {
    calculateFee,
    GasPrice,
    SigningStargateClient,
    DeliverTxResponse,
} from "@cosmjs/stargate";
import {IndexedTx, isDeliverTxSuccess} from "@cosmjs/stargate/build/stargateclient";
import { Event, Attribute } from "@cosmjs/stargate";

const coreumAccountPrefix = "testcore"; // the address prefix (different for different chains/environments)
const coreumHDPath = "m/44'/990'/0'/0/0"; // coreum HD path (same for all chains/environments)
const coreumDenom = "utestcore"; // core denom (different for different chains/environments)
const coreumRpcEndpoint = "https://full-node.testnet-1.coreum.dev:26657"; // rpc endpoint (different for different chains/environments)
const recipientAddress = "testcore1534s8rz2e36lwycr6gkm9vpfe5yf67wkuca7zs"
const senderMnemonic = ""; // put mnemonic here

const main = (async function() {
    console.log("preparing sender wallet");
    const senderWallet = await DirectSecp256k1HdWallet.fromMnemonic(senderMnemonic, {
        prefix: coreumAccountPrefix,
        hdPaths: [stringToPath(coreumHDPath)],
    });
    const [sender] = await senderWallet.getAccounts();
    console.log(`sender address: ${sender.address}`);

    const senderClient = await SigningStargateClient.connectWithSigner(
        coreumRpcEndpoint,
        senderWallet
    );

    const txResult = await sendCoins(sender.address, recipientAddress, senderClient, {
        denom: coreumDenom,
        amount: "9000000",
    })

    await queryBalance(recipientAddress, senderClient);

    await queryBlock(senderClient);

    const tx = await queryTx(txResult.transactionHash, senderClient);

    balanceUpdates(tx);
})();

async function sendCoins(sender: string, recipient: string, senderClient: SigningStargateClient, amount): Promise<DeliverTxResponse> {
    console.log(`sending ${amount.amount} to recipient:`);
    // Initial gas price is hardcoded for now here, because client doesn't support querying for current gas price.
    const gasPrice = GasPrice.fromString(`0.0625${coreumDenom}`);
    const singleBankSendGas = 111_000;
    const bankSendFee: StdFee = calculateFee(singleBankSendGas, gasPrice);
    const bankSendResult = await senderClient.sendTokens(
        sender,
        recipient,
        [amount],
        bankSendFee
    );
    isDeliverTxSuccess(bankSendResult);
    console.log(`successfully sent, tx hash: ${bankSendResult.transactionHash}`);

    return bankSendResult;
}

async function queryBalance(address: string, senderClient: SigningStargateClient) {
    const balance = await senderClient.getBalance(address, coreumDenom);
    console.log(`balance: ${balance.amount}`);
}

async function queryBlock(senderClient: SigningStargateClient) {
    // Query latest block
    const latestBlock = await senderClient.getBlock();
    console.log(`latest block: `, latestBlock.header);

    // Query block by height
    const block = await senderClient.getBlock(latestBlock.header.height);
    console.log(`block: `, block.header);
}

async function queryTx(txHash: string, senderClient: SigningStargateClient): Promise<IndexedTx> {
    // Query tx by hash
    const tx = await senderClient.getTx(txHash);
    console.log(`tx: `, tx);

    // Check if transaction succeeded
    console.log(tx.code === 0);

    return tx;
}

function balanceUpdates(tx: IndexedTx) {
    // Detecting balance changes
    tx.events.forEach((event: Event) => {
        switch (event.type) {
            case 'coin_received':
                var receiver: string;
                var amount: string;
                event.attributes.forEach((attr: Attribute) => {
                    switch (attr.key) {
                        case "receiver":
                            receiver = attr.value;
                            break;
                        case "amount":
                            amount = attr.value;
                            break;
                    }
                });

                parseCoins(amount).forEach((coin: Coin) => {
                    if (coin.denom != coreumDenom) {
                        return;
                    }

                    console.log(`${receiver} received ${coin.amount}${coin.denom}`);
                });
                break;
            case 'coin_spent':
                var spender: string;
                var amount: string;
                event.attributes.forEach((attr: Attribute) => {
                    switch (attr.key) {
                        case "spender":
                            spender = attr.value;
                            break;
                        case "amount":
                            amount = attr.value;
                            break;
                    }
                });

                parseCoins(amount).forEach((coin: Coin) => {
                    if (coin.denom != coreumDenom) {
                        return;
                    }

                    console.log(`${spender} spent ${coin.amount}${coin.denom}`);
                })
                break;
        }
    });
}

export default main;
```
{% endcode %}
