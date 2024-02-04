# Wallet Integration

In order to integrate Coreum into a wallet, it should be noted that since Coreum is written using Cosmos SDK, the process is mostly the same to other Cosmos based chains with only 2 small differences relating to fees, namely gas and gas price. Worth considering is also handling of fungible tokens and how that differs from the standard Cosmos SDK functionality.

### How Gas Price is Different <a href="#how-gas-price-is-different" id="how-gas-price-is-different"></a>

To understand Coreum chain gas price read about the [coreum gas price](https://docs.coreum.dev/tutorials/gas-price.html).

### How Gas is Different <a href="#how-gas-is-different" id="how-gas-is-different"></a>

Gas estimation in Coreum is slightly different from default Cosmos SDK, since gas usage by each transaction is more deterministic.

For example, if you construct any 2 transactions with a single `bank.MsgSend` message and a single signature in each, and if the transaction size is less than a certain value, you will get identical gas requirement for both transactions.

You can get the gas requirement by calling the `simulate` endpoint just as you would with the default Cosmos SDK so if you already use the simulation endpoint no modifications are needed.

If you are not using the simulation endpoint (we strongly recommend you do) then you need to calculate the values, consulting the table in the `deterministicgas` module linked below.

However, these numbers are subject to change, and you need to keep up to date with them on every upgrade (this is why we recommend using the `simulate` endpoint).

You can find more information about our `deterministicgas` model [here](https://docs.coreum.dev/modules/deterministicgas.html).

### How Fungible Tokens Are Different <a href="#how-fungible-tokens-are-different" id="how-fungible-tokens-are-different"></a>

Although the functionality of fungible token creation and minting is present in the original `bank` module of Cosmos SDK, it is not exposed to end users, and it is only possible to create new fungible tokens via either the governance or IBC. To work around this problem we have wrapped the `bank` module into the `wbank` module. This structure allows reusing the code provided by Cosmos SDK, and also reuse the infrastructure that the community provides (e.g. explorers and wallets). But it also leads to the fact that some of the information regarding fungible tokens will exist in the `assetft` module and some in the `bank` module. For example, if you want to query for frozen balances of a fungible token, you need to query the `assetft` module but if you want to get the total supply, you must query the `bank` module.

In a nutshell, `assetft` module interacts with `wbank` which in turn wraps the original `bank` module.

The issue method implemented at Coreum, makes it possible for everyone to create a fungible token and manage its supply. When the issuer issues a token, they specify the initial total supply which will be delivered to the issuer's account address. While the token is being issued (and only then), a set of corresponding token features can be enabled: `minting`, `freezing`, `burning`, `burn rate` and/or `commission rate`.

#### Convention Around FT Token Issuance <a href="#convention-around-ft-token-issuance" id="convention-around-ft-token-issuance"></a>

A denom is the main identifier of the token. It is created by joining the subunit (provided by the issuer) and the issuer address, separated with a dash (`subunit-address`). The user also provides the token symbol and precision which will only be used for display purposes and will be stored in the bank module's metadata field.

For example, to represent Bitcoin on Coreum, one could choose `satoshi` as subunit, `BTC` as Symbol and `8` as precision. It means that if the issuer address is `core1tr3w86yesnj8f290l6ve02cqhae8x4ze0nk0a8` then the denom will be `satoshi-core1tr3w86yesnj8f290l6ve02cqhae8x4ze0nk0a8` and since we have chosen `BTC` as token symbol and `8` as precision, it will result in `1 BTC = 10^8 satoshi-core1tr3w86yesnj8f290l6ve02cqhae8x4ze0nk0a8`.

More information about FT module is covered [here](https://docs.coreum.dev/modules/assetft.html). Moreover, there are several guides describing different ways of interacting with fungible token:

* [CLI](https://docs.coreum.dev/tutorials/smart-tokens/asset-ft.html)
* programmatically with [go](https://docs.coreum.dev/tutorials/smart-tokens/asset-ft-go.html) or [TS](https://docs.coreum.dev/tutorials/cosmjs.html)
