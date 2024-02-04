# Ledger Nano

This tutorial will show you how to utilize a Ledger device with the Cosmos Ledger app and `cored` CLI

The use of a hardware wallet for storing crypto keys significantly enhances the security of your digital assets. The Ledger device functions as a secure enclave that contains the seed and private keys, and the signing process for transactions occurs entirely within the device ensuring that no private data is ever transmitted externally.

DISCLAIMER

Since Ledger limits HD paths, CORE `--coin-type=990` is currently not supported, but we will use ATOM `--coin-type=118`(path m/44/118), which will also work for us.

### &#x20;<a href="#prerequisites" id="prerequisites"></a>

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

* Set up [Ledger Live](https://www.ledger.com/ledger-live/)
* on your machine.
* The Cosmos(ATOM) application must be installed on your Ledger device, for more information click [here](https://support.ledger.com/hc/en-us/articles/360013713840-Cosmos-ATOM-?support=true)

### &#x20;<a href="#instructions" id="instructions"></a>

### Instructions <a href="#instructions" id="instructions"></a>

* Connect your Ledger device and unlock it
* Open Cosmos(ATOM) app on you Ledger device
* Set up an account in `cored` using your Ledger key.

NOTE:

The instructions provided will generate private keys on your Ledger device using the `cored` tool in Coreum network and keep references to them saved locally. However, the actual keys will be exclusively stored within the Ledger device.

```sh
#cored keys add [name]  --chain-id=[chain ID] --keyring-backend=[os|file|test] --ledger --coin-type=[coin type]
cored keys add ledger-1 --chain-id=coreum-testnet-1  --ledger --coin-type=118
```

NOTE:

* You can use the `--keyring-backend` flag to increase the security of your keys.
* If you want to target other network than testnet, replace it with values at [network variables page](https://docs.coreum.dev/tutorials/network-variables.html)

```sh
#cored keys add [name]  --chain-id=[chain ID] --keyring-backend=[os|file|test] --ledger --coin-type=[coin type] 
cored keys add ledger-2 --chain-id=coreum-testnet-1 --keyring-backend=test  --ledger --coin-type=118
```

* Verify your address

Execute this command to display your address on your Ledger device. Use the `name` that you assigned to your Ledger key.

NOTE:

Before executing commands, make sure to unlock your device using the PIN and open the Cosmos app.

```sh
cored keys show ledger-1 -d --chain-id=coreum-testnet-1
# Confirm that the address shown on your device matches the one displayed when you added the key.
```

* Fund your account address
  * Navigate [faucet page](https://docs.coreum.dev/tools-ecosystem/faucet.html) designed specifically for this purpose.
  * On the faucet page, you will find a designated section for requesting testnet funds.
  * Enter the wallet address generated earlier into the `wallet address` field.
  * Click on the `Request Funds` button to initiate the request for testnet funds.

By following these steps, you can easily navigate to the faucet page, enter your wallet address, and request the desired testnet funds for your account. This process allows you to obtain the necessary funds to perform transactions and test the functionality of the network.

* Sign a transaction

You are all set to begin authorizing and transmitting transactions. To send a transaction with `cored`, use the `tx bank send` command.

```sh
cored tx bank send --help
# output:
# cored tx bank send [from_key_or_address] [to_address] [amount] [flags]
```

Full command should look like this:

```sh
#cored tx bank send <keyName> <destinationAddress> <amount><denom> <flags>
cored tx bank send ledger-1 tescore1snn05vrzvnwy7t0g00rr7hva63hmwxuuv7nrj0 1000000utestcore --chain-id=coreum-testnet-1 --node=https://full-node.testnet-1.coreum.dev:26657 --keyring-backend=os  --ledger
# ledger-1 is your local account name, which can be replaced by an address.
# 1000000utestcore is equal to 1testcore
```

Ensure that you respond with `Y` when prompted to confirm the transaction before signing.

Following that, you will receive a prompt on your Ledger device to review and authorize the transaction. It is essential to carefully examine the transaction JSON displayed on the screen. Take your time to scroll through each field and message. Scroll down for further information about the data fields of a standard transaction object.

Congratulations! You have sent your first transaction using Ledger and `cored` CLI!

## &#x20;<a href="#instructions-to-ledger-nano-keplr" id="instructions-to-ledger-nano-keplr"></a>

## Instructions to Ledger Nano + Keplr <a href="#instructions-to-ledger-nano-keplr" id="instructions-to-ledger-nano-keplr"></a>

### NOTE:

Before proceeding with this section, make sure to install the Cosmos app on your Ledger Nano device.

* Connect your Ledger device to your computer, enter the PIN to unlock it, and open the Cosmos app.
* Install the [Keplr browser extension](https://www.keplr.app/)
* Click the Keplr extension icon, then choose `Connect Hardware Wallet` from the options.
* Ensure that your Ledger device is unlocked and the Cosmos app is open. Then, follow the instructions provided in the Keplr pop-up.
* Connect to the [Coreum Chain](https://docs.coreum.dev/tools-ecosystem/wallet.html#keplr) to add the Coreum account.

<details>

<summary>OPTIONAL</summary>





Way to add `Coreum Mainnet` chain to the `Keplr` extension:

* Navigate to [Add Chains to Keplr](https://chains.keplr.app/)
* .
* In the search field, enter `Coreum`.
* From the search results, locate `Coreum` and click the `Add to Keplr` button.
* A transaction prompt will appear asking you to approve the `Add Coreum to Keplr` transaction.

</details>

*   Click the `Connect to coreum-testnet-1` button to establish a connection with the Coreum testnet chain.

    This action will also prompt you to approve the request to `Add Coreum Testnet 1 to Keplr`.

    TIP

    To establish a connection with various networks like Mainnet, Testnet, or Devnet, you can initiate the connection by clicking on the corresponding network buttons

    listed on [Coreum Docs page](https://docs.coreum.dev/tools-ecosystem/wallet.html#keplr).
* By default, the Coreum Chain may not be visible. To change this check the `DETAILS`:

<details>

<summary>DETAILS</summary>



* Open the Keplr extension.
* Select the `Manage Chain Visibility` option from the burger menu located at the top left corner.
* In the `Search networks` field, type `Coreum` to find the Coreum testnet.
* Choose the `Coreum Testnet 1` option.
* Finally, click the `Save` button to apply the changes and make the Coreum Chain visible in the Keplr extension.

By following these steps, you can easily connect to the Coreum testnet chain in Keplr and ensure that the Coreum Chain is visible within the extension for seamless interaction

</details>

* To fund the test account:
  * Start by clicking the `Deposit` button.
  * Next, click the `Copy` button located next to the address of the `Coreum Testnet 1`.
  * Open the [faucet](https://docs.coreum.dev/tools-ecosystem/faucet.html#testnet) and paste the copied address into the designated address field.
  * Finally, click the `Request Funds` button to initiate the request for funds.
* Now we can send the funds on `Coreum` testnet Chain:
  * In Keplr extension - click on the `TESTCORE` asset to select it.
  * On the `Send` page, enter the destination address (e.g., `testcore1hnr7882vkpg3rurqgva09pu329qq8f3c3phmwq`) and specify the amount you wish to send.
  * Once you have entered the necessary details, approve the transaction by using your Ledger device.
  * After approving the transaction, verify that the balance has been successfully updated to reflect the changes.

### NOTE:

Whenever you initiate a transaction, you will be required to confirm it on your Ledger device. The Keplr interface will provide the necessary prompts for confirmation.

Great! You have successfully set up Keplr with your Ledger Nano. For more comprehensive details, you can refer to the [Ledger support page](https://support.ledger.com/hc/en-us/articles/4411149814417?docs=true)
