# Create and Manage NFTs

This tutorial will guide you through the process of using the [assetnft](https://docs.coreum.dev/modules/assetnft.html) module to create and manage Non-Fungible Tokens.

Please note that each subsequent section depends on the previous one.

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

* [cored binary installed](https://docs.coreum.dev/tutorials/cored.html)
*   We are going to interact with testnet, so we need to set `--node` and `--chain-id` flag in each request.

    ```
    export CHAIN_ID="coreum-testnet-1"
    export RPC_URL="https://full-node.testnet-1.coreum.dev:26657"
    ```

    If you want to use other network, find relevant values at [network variables](https://docs.coreum.dev/tutorials/network-variables.html) page.
*   Two accounts at your local keychain. If you don't have them, follow these steps - go to [faucet page](https://docs.coreum.dev/tools-ecosystem/faucet.html) and generate two accounts there. Then, having the mnemonics, import them with the following commands

    ```
    cored keys add nft-issuer-wallet --recover --chain-id=$CHAIN_ID
    # put first mnemonic here
    cored keys add nft-receiver-wallet --recover --chain-id=$CHAIN_ID
    # put second mnemonic here
    ```
*   Also, since network operates with raw addresses, not your local names, we are going to bind raw addresses to env vars. We will export the following values:

    ```
    export NFT_ISSUER_ADDRESS=$(cored keys show nft-issuer-wallet --address --chain-id=$CHAIN_ID)
    export NFT_RECEIVER_ADDRESS=$(cored keys show nft-receiver-wallet --address --chain-id=$CHAIN_ID)
    ```

### Create your first NFT <a href="#create-your-first-nft" id="create-your-first-nft"></a>

*   We will export one more environment variable to store NFT class id, which is a uniquely named group of NFT objects. It can be defined with different characteristics (features) that are common for all NFTs belonging to the class.

    ```
    export NFT_CLASS_ID=$(echo "puppysmartnft1-"$NFT_ISSUER_ADDRESS)
    ```

#### Create NFT class <a href="#create-nft-class" id="create-nft-class"></a>

*   Let's create new NFT class:

    ```
    # cored tx assetnft issue-class [symbol] [name] [description] [uri] [uri_hash] --from [issuer] --features=burning,freezing,whitelisting,disable_sending [flags]
    cored tx assetnft issue-class puppysmartnft1 "Puppy NFTs" "A collection of awesome puppy NFTs" "http://puppy-nfts.com" "somehash" --from $NFT_ISSUER_ADDRESS --features=burning,freezing,whitelisting --node=$RPC_URL --chain-id=$CHAIN_ID
    # confirm transaction before signing and broadcasting [y/N]: y
    ```

    As an output, you should receive tx hash, copy it and go to [Block explorer](https://docs.coreum.dev/tools-ecosystem/block-explorer.html) to see the tx status. Features: `burning`, `freezing` and `whitelisting` are explained later in this guide.
*   Now, let's check what NFT classes reside on the Coreum blockchain:

    ```
    cored q nft classes --node=$RPC_URL --chain-id=$CHAIN_ID
    #classes:
    #- data: null
    #  description: A collection of awesome puppy NFTs
    #  id: puppysmartnft1-testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7
    #  name: Puppy NFTs
    #  symbol: puppysmartnft1
    #  uri: http://puppy-nfts.com
    #  uri_hash: somehash
    ```

    As you see from the output, your new token has got unique class `id` which consists of the `symbol` provided and issuer account address.

#### Mint NFTs <a href="#mint-nfts" id="mint-nfts"></a>

*   We will now mint 2 NFTs within created class

    ```
    #cored tx assetnft mint [class-id] [id] [uri] [uri_hash] --from [sender] [flags]
    cored tx assetnft mint $NFT_CLASS_ID puppysmartnft-1 "http://puppy-nfts.com/puppynft-1" "somehash" --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID

    cored tx assetnft mint $NFT_CLASS_ID puppysmartnft-2 "http://puppy-nfts.com/puppynft-2" "somehash" --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID
    ```

    Again, we can check transaction status by providing transaction hashes on the [Block explorer](https://docs.coreum.dev/tools-ecosystem/block-explorer.html) page
* We can query all NFTs of a given class with the following command

```
cored q nft nfts --class-id=$NFT_CLASS_ID --node=$RPC_URL --chain-id=$CHAIN_ID
#nfts:
#- class_id: puppysmartnft1-testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7
#  data: null
#  id: puppysmartnft-1
#  uri: http://puppy-nfts.com/puppynft-1
#  uri_hash: somehash
#- class_id: puppysmartnft1-testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7
#  data: null
#  id: puppysmartnft-2
#  uri: http://puppy-nfts.com/puppynft-2
#  uri_hash: somehash
```

As you can see, 2 NFTs were properly minted.

### Whitelisting <a href="#whitelisting" id="whitelisting"></a>

Whitelisting functionality allows transferring NFTs of a certain class only to specific addresses. Since we enabled that feature for our NFT class, in order to send one of our minted NFTs to a receiver, we need to first whitelist them.

```
cored tx assetnft whitelist $NFT_CLASS_ID puppysmartnft-1 $NFT_RECEIVER_ADDRESS --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID
```

### Sending and querying balance <a href="#sending-and-querying-balance" id="sending-and-querying-balance"></a>

Our NFT is now ready to be transferred to the receiving account.

#### Send <a href="#send" id="send"></a>

```
cored tx nft send $NFT_CLASS_ID puppysmartnft-1 $NFT_RECEIVER_ADDRESS --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID
```

#### Query the balance <a href="#query-the-balance" id="query-the-balance"></a>

We can query NFT balances in multiple ways. These could be as follows

```
cored q nft balance $NFT_RECEIVER_ADDRESS $NFT_CLASS_ID --node=$RPC_URL --chain-id=$CHAIN_ID
# amount: "1"
```

That way, we can query the number of NFTs of a given class owned by the owner. Our receiving account owns 1 NFT of this class.

We can also query the owner of the NFT based on its class and id.

```
cored q nft owner  $NFT_CLASS_ID puppysmartnft-1  --node=$RPC_URL --chain-id=$CHAIN_ID
# owner: testcore1k9575m9egrlmymnyd29p5g0p5e94d930tg67sv
```

### Freezing <a href="#freezing" id="freezing"></a>

Freezing, if enabled for NFT class, allows issuer to freeze specific NFT of a given class. A frozen token cannot be transferred until it is unfrozen by the issuer.

#### Freeze <a href="#freeze" id="freeze"></a>

```
cored tx assetnft freeze $NFT_CLASS_ID puppysmartnft-1 --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID
```

Our NFT of this class is now frozen. Let's verify if this is actually the case

* Query frozen

Using the following command, we can check `frozen` flag set for an NFT in a given class

```
cored q assetnft frozen $NFT_CLASS_ID puppysmartnft-1 --node=$RPC_URL --chain-id=$CHAIN_ID
#frozen: true
```

* Let's now try to send the NFT from the receiver back to the issuer. Such transaction should not succeed.

```
cored tx nft send $NFT_CLASS_ID puppysmartnft-1 $NFT_ISSUER_ADDRESS --from $NFT_RECEIVER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID -b=block
#at the end of the raw_log section, notice an error: "raw_log: 'failed to execute message; message index: 0: nft with classID:puppysmartnft1-testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7 and ID:puppysmartnft-1 is frozen: unauthorized'
```

Notice we used `-b=block` argument which waits for the tx to pass/fail checks and be committed in a block.

#### Unfreeze <a href="#unfreeze" id="unfreeze"></a>

* We can unfreeze the token and make it transferable again

```
cored tx assetnft unfreeze $NFT_CLASS_ID puppysmartnft-1 --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID
```

* Sending the token back to the issuer succeeds this time

```
cored tx nft send $NFT_CLASS_ID puppysmartnft-1 $NFT_ISSUER_ADDRESS --from $NFT_RECEIVER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID -b=block
```

* Let's query the current owner of the token

```
cored q nft owner $NFT_CLASS_ID puppysmartnft-1  --node=$RPC_URL --chain-id=$CHAIN_ID
#owner: testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7
```

The address shown belongs to the issuer.

#### Class Freeze <a href="#class-freeze" id="class-freeze"></a>

* Let's send the NFT back to receiver first.

```
cored tx nft send $NFT_CLASS_ID puppysmartnft-1 $NFT_RECEIVER_ADDRESS --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID
```

* Now we can freeze all the NFTs of a class held by an account

```
cored tx assetnft class-freeze $NFT_CLASS_ID $NFT_RECEIVER_ADDRESS --from $NFT_ISSUER_ADDRESS --chain-id=$CHAIN_ID -b=block
```

* Let's now try to send the NFT from the issuer to the receiver. Such transaction should not succeed.

```
cored tx nft send $NFT_CLASS_ID puppysmartnft-1 $NFT_ISSUER_ADDRESS --from $NFT_RECEIVER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID -b=block
#at the end of the raw_log section, notice an error: "raw_log: 'failed to execute message; message index: 0: nft with classID:puppysmartnft1-testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7 and ID:puppysmartnft-1 is frozen: unauthorized'
```

#### Class Unfreeze <a href="#class-unfreeze" id="class-unfreeze"></a>

* We can remove the class freeze of an account

```
cored tx assetnft class-unfreeze $NFT_CLASS_ID $NFT_RECEIVER_ADDRESS --from $NFT_ISSUER_ADDRESS --chain-id=$CHAIN_ID -b=block
```

* Sending the token back to the issuer succeeds this time

```
cored tx nft send $NFT_CLASS_ID puppysmartnft-1 $NFT_ISSUER_ADDRESS --from $NFT_RECEIVER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID -b=block
```

### Burning <a href="#burning" id="burning"></a>

If `burning` feature is enabled on NFT class level, it allows an owner of a token to burn it.

* First, let's query an nft we want to burn

```
cored q nft nft $NFT_CLASS_ID puppysmartnft-2 --node=$RPC_URL --chain-id=$CHAIN_ID
#nft:
#  class_id: puppysmartnft1-testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7
#  data: null
#  id: puppysmartnft-2
#  uri: http://puppy-nfts.com/puppynft-2
#  uri_hash: somehash
```

* Now, then token can be burnt

```
cored tx assetnft burn $NFT_CLASS_ID puppysmartnft-2 --from $NFT_ISSUER_ADDRESS --node=$RPC_URL --chain-id=$CHAIN_ID
```

* As a verification step, let's check if the token still exists

```
cored q nft nft $NFT_CLASS_ID puppysmartnft-2 --node=$RPC_URL --chain-id=$CHAIN_ID
#notice an error at the end of the output
#not found nft: class: puppysmartnft1-testcore105hmczwh0tkha2h5lu9rr07xtegzsm49d3hxq7, id: puppysmartnft-2: nft does not exist: invalid request
```

### Further Reading <a href="#what-is-next" id="what-is-next"></a>

You can read more about Non-Fungible Tokens at [Smart Token Overview](../../overview/smart-token.md) and [Coreum NFTs](../../developer-options/coreum/non-fungible-tokens.md) pages.
