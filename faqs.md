# ❓ FAQs

## Creating and Managing FT with CLI <a href="#what-if-the-account-s-non-zero-balance-will-be-removed-from-the-whitelist" id="what-if-the-account-s-non-zero-balance-will-be-removed-from-the-whitelist"></a>

### What if the account's non-zero balance will be removed from the whitelist? <a href="#what-if-the-account-s-non-zero-balance-will-be-removed-from-the-whitelist" id="what-if-the-account-s-non-zero-balance-will-be-removed-from-the-whitelist"></a>

If an account's non-zero balance is removed from the whitelist, the account will still be able to spend any tokens it has received. However, if you try to send new tokens to this account, the transaction will fail.

## Creating and Managing NFT with CLI <a href="#what-if-the-account-s-non-zero-balance-will-be-removed-from-the-whitelist" id="what-if-the-account-s-non-zero-balance-will-be-removed-from-the-whitelist"></a>

### Can a token be always burnt by an owner? <a href="#can-a-token-be-always-burnt-by-an-owner" id="can-a-token-be-always-burnt-by-an-owner"></a>

No, when a token is frozen, it cannot be burnt. Also, `burning` feature needs to be enabled on NFT class id level.

### In order to burn a token, does `burning` feature always need to be enabled? <a href="#in-order-to-burn-a-token-does-burning-feature-always-need-to-be-enabled" id="in-order-to-burn-a-token-does-burning-feature-always-need-to-be-enabled"></a>

In general yes. However, an issuer of a token, can always burn it (while they own it), regardless of the feature setting.

### Can all tokens within an NFT class be frozen/unfrozen at once? <a href="#can-all-tokens-within-an-nft-class-be-frozen-unfrozen-at-once" id="can-all-tokens-within-an-nft-class-be-frozen-unfrozen-at-once"></a>

No. There's no single command allowing to achieve that. It can, though, be done programmatically (for example, by utilizing the output returned by `cored q nft nfts --class-id=$NFT_CLASS_ID --node=$RPC_URL --chain-id=$CHAIN_ID` command).

## Others <a href="#what-is-next" id="what-is-next"></a>

### How to point `cored` to a specific network? <a href="#how-to-point-cored-to-a-specific-network" id="how-to-point-cored-to-a-specific-network"></a>

By default, cored binary points to the local node(localhost:26657).

To point it to a specific node you should use two flags - `--node` and `--chain-id`.

For example status for the specific node can be retrieved this way:

```sh
cored status --node={NODE_URL_WITH_PORT} --chain-id={CHAIN_ID}
```

NODE\_URL\_WITH\_PORT and CHAIN\_ID can be found at [network variables page](https://docs.coreum.dev/tutorials/network-variables.html)

### How to fix `account not found` error? <a href="#how-to-fix-account-not-found-error" id="how-to-fix-account-not-found-error"></a>

If you see the next message:

```sh
Error: rpc error:
  code = NotFound desc = rpc error:
    code = NotFound desc = account testcore1q07ldrjnr8xtsy3rz82yxqcdrffu3uw3daslrw not found:
      key not found
```

Two reasons might cause this issue:

1. Your account probably has zero balance, and it is not visible on the network. You should [fund](https://docs.coreum.dev/tools-ecosystem/faucet.html) it before using it.
2. RPC node you used to send the request is not fully synced, use another RPC node or wait for sync completion.

### &#x20;<a href="#how-to-fix-key-not-found-error" id="how-to-fix-key-not-found-error"></a>

### How to fix `key not found` error? <a href="#how-to-fix-key-not-found-error" id="how-to-fix-key-not-found-error"></a>

If you see the next message:

```sh
Error: testcore1f2dyj8dhdv62ytrkuvn832ezzjdcpg2jhrtzvy.info: key not found
```

Probably, your `RPC_URL` and/or `chain-id` belong to different networks. Also, this error often occurs when you have added your account into a specific keyring (i.e. --keyring-backend=test) and you forget to specify that keyring in CLI command.



## Useful Links

### Websites <a href="#websites" id="websites"></a>

[Coreum Front Page](https://coreum.com)

[Github](https://github.com/CoreumFoundation)

### Communication <a href="#communication" id="communication"></a>

[Discord Validators channel](https://discord.gg/VgkhYeWmTd)
