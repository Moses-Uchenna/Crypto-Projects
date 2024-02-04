# Keplr Wallet

## IBC Transfer to Osmosis Using Keplr Wallet <a href="#ibc-transfer-to-osmosis-using-keplr-wallet" id="ibc-transfer-to-osmosis-using-keplr-wallet"></a>

This detailed guide provides step-by-step instructions on how to perform an IBC (Inter-Blockchain Communication) transfer to Osmosis using the Keplr wallet on the Coreum blockchain. Here's a guide of the key steps outlined below:

### 1. Open Keplr Wallet <a href="#id-1-open-keplr-wallet" id="id-1-open-keplr-wallet"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/keplr_open.da1c2684.png" alt="Open Keplr Wallet" width="50%">

</div>

Start by opening your Keplr wallet. If you don't have it installed, you can download it from the browser extension store.

### 2. Navigate to Advanced IBC Transaction <a href="#id-2-navigate-to-advanced-ibc-transaction" id="id-2-navigate-to-advanced-ibc-transaction"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/advanced_ibc_transfer.03f41ff7.png" alt="Navigate to Advanced IBC Transaction" width="50%">

</div>

In your Keplr wallet, navigate to the section where you can initiate an advanced IBC transaction.

`Note:` If you do not have "Developer Mode" enabled on your Keplr Wallet, you will not see this option. To enable developer mode, go to Settings -> Advanced, and enabled "Developer Mode".

### 3. Select Asset <a href="#id-3-select-asset" id="id-3-select-asset"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/select_asset.dbcdbc74.png" alt="Select Asset" width="50%">

</div>

Choose the asset/token you want to transfer. Ensure you have sufficient balance for the transfer.

### 4. Choose Destination Chain <a href="#id-4-choose-destination-chain" id="id-4-choose-destination-chain"></a>

#### Adding IBC Transfer Channel in Keplr <a href="#adding-ibc-transfer-channel-in-keplr" id="adding-ibc-transfer-channel-in-keplr"></a>

You must first add the IBC Transfer Channel to Keplr. When you click on destination chain, you will see "Add New IBC Tranfer Channel", with a "+" icon. Choose the relevant destination chain.

<div align="center">

<img src="https://docs.coreum.dev/assets/new_ibc_transfer_channel.9b3bd7e1.png" alt="New IBC Transfer Channel" width="50%">

</div>

Next, you can add an IBC Channel, and the corresponding `Source Channel ID`

<div align="center">

<img src="https://docs.coreum.dev/assets/add_ibc_channel.0f214c2e.png" alt="Add IBC Channel" width="50%">

</div>

You can find the relevant Configurations for Coreum here:

* [Coreum IBC Channels](./)

<div align="center">

<img src="https://docs.coreum.dev/assets/choose_destination_chain.ac4607d1.png" alt="Choose Destination Chain" width="50%">

</div>

Select the blockchain where you want to send the asset. This is the receiving chain of the IBC transfer.

`Note:`This will be blank if the channel-id has not been registered in the IBC Chain Registry, and correctly opened by the relayer.

### 5. Enter Wallet Address <a href="#id-5-enter-wallet-address" id="id-5-enter-wallet-address"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/wallet_address.bf7d428f.png" alt="Enter Wallet Address" width="50%">

</div>

Provide the address of the recipient's wallet on the destination chain.

### 6. Enter Amount <a href="#id-6-enter-amount" id="id-6-enter-amount"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/amount.7db008dd.png" alt="Enter Amount" width="50%">

</div>

Specify the amount of the asset you wish to transfer. Ensure you account for any transaction fees.

### 7. Review Transaction <a href="#id-7-review-transaction" id="id-7-review-transaction"></a>

![Confirm Transaction](https://docs.coreum.dev/assets/confirm\_transaction.d3538121.png)

Review all the details of the transfer. Ensure everything looks correct before proceeding.

#### 7a. Confirm Transaction Details <a href="#id-7a-confirm-transaction-details" id="id-7a-confirm-transaction-details"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/confirm_transaction2.009ced45.png" alt="Confirm Transaction Details" width="50%">

</div>

Keplr might ask you to confirm the transaction details once more. Double-check and proceed.

#### 7b. Approve Transaction <a href="#id-7b-approve-transaction" id="id-7b-approve-transaction"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/confirm_data.ed77fdd1.png" alt="Confirm Data" width="50%">

</div>

Ensure all transaction data is correct. This includes the amount, destination address, and asset type.

### 10. Final Transaction Confirmation <a href="#id-10-final-transaction-confirmation" id="id-10-final-transaction-confirmation"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/confirm_transaction3.9a200ac4.png" alt="Final Transaction Confirmation" width="50%">

</div>

Before the transaction is broadcasted, confirm it for the last time. This is the final step before the transaction is sent to the network.

### 11. Transaction Explorer <a href="#id-11-transaction-explorer" id="id-11-transaction-explorer"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/transaction_explorer.080a1b3b.png" alt="Transaction Explorer">

</div>

Once the transaction is confirmed, you can view it on Coreum's blockchain explorer.

`Note:` Tt's essential to understand that IBC transactions might not be instantaneous. There's a brief delay between sending the transaction from the source chain and receiving it on the destination chain. Additionally, if a transaction timeouts for any reason, this will be visible on the source chain, not the destination chain. Always monitor the source chain's explorer for any timeout or error messages related to your IBC transaction.

### 12. Transaction Hash <a href="#id-12-transaction-hash" id="id-12-transaction-hash"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/transaction_hash.c7a090cf.png" alt="Transaction Hash">

</div>

This is the unique identifier for your transaction. You can use it to track the status of your transfer.

### 13. Explorer Transaction Messages <a href="#id-13-explorer-transaction-messages" id="id-13-explorer-transaction-messages"></a>

<div align="center">

<img src="https://docs.coreum.dev/assets/explorer_tx_messages.75736892.png" alt="Explorer Transaction Messages">

</div>

In the blockchain explorer, you can see detailed messages associated with your transaction, including confirmations from validators and more.

## Transfer Message <a href="#transfer-message" id="transfer-message"></a>

{% code lineNumbers="true" fullWidth="true" %}
```shell
{
    "memo": "",
    "@type": "/ibc.applications.transfer.v1.MsgTransfer",
    "token": {
        "denom": "ucore",
        "amount": "52931"
    },   "sender": "core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74",
    "receiver": "osmo1pwvcapna75slt3uscvupfe52492yuzhflhakem",
    "source_port": "transfer",
    "source_channel": "channel-2",
    "timeout_height": {
        "revision_height": "10958485",
        "revision_number": "1"
    },
    "timeout_timestamp": "0"
}
```
{% endcode %}

### Transfer Logs <a href="#transfer-logs" id="transfer-logs"></a>

Here are the example logs from our IBC Transfer:

#### Logs <a href="#logs" id="logs"></a>

{% code lineNumbers="true" fullWidth="true" %}
```sh
[
    {
        "events": [
            {
                "type": "coin_received",
                "attributes": [
                    {
                        "key": "receiver",
                        "value": "core12k2pyuylm9t7ugdvz67h9pg4gmmvhn5vvgafk0"
                    },
                    {
                        "key": "amount",
                        "value": "52931ucore"
                    }
                ]
            },
            {
                "type": "coin_spent",
                "attributes": [
                    {
                        "key": "spender",
                        "value": "core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74"
                    },
                    {
                        "key": "amount",
                        "value": "52931ucore"
                    }
                ]
            },
            {
                "type": "ibc_transfer",
                "attributes": [
                    {
                        "key": "sender",
                        "value": "core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74"
                    },
                    {
                        "key": "receiver",
                        "value": "osmo1pwvcapna75slt3uscvupfe52492yuzhflhakem"
                    }
                ]
            },
            {
                "type": "message",
                "attributes": [
                    {
                        "key": "action",
                        "value": "/ibc.applications.transfer.v1.MsgTransfer"
                    },
                    {
                        "key": "sender",
                        "value": "core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74"
                    },
                    {
                        "key": "module",
                        "value": "ibc_channel"
                    },
                    {
                        "key": "module",
                        "value": "transfer"
                    }
                ]
            },
            {
                "type": "send_packet",
                "attributes": [
                    {
                        "key": "packet_data",
                        "value": "{\"amount\":\"52931\",\"denom\":\"ucore\",\"receiver\":\"osmo1pwvcapna75slt3uscvupfe52492yuzhflhakem\",\"sender\":\"core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74\"}"
                    },
                    {
                        "key": "packet_data_hex",
                        "value": "7b22616d6f756e74223a223532393331222c2264656e6f6d223a2275636f7265222c227265636569766572223a226f736d6f317077766361706e613735736c74337573637675706665353234393279757a68666c68616b656d222c2273656e646572223a22636f726531307a7432723570327a68396c7463796739387a743267746d6379706b716771337173666a3734227d"
                    },
                    {
                        "key": "packet_timeout_height",
                        "value": "1-10958485"
                    },
                    {
                        "key": "packet_timeout_timestamp",
                        "value": "0"
                    },
                    {
                        "key": "packet_sequence",
                        "value": "17"
                    },
                    {
                        "key": "packet_src_port",
                        "value": "transfer"
                    },
                    {
                        "key": "packet_src_channel",
                        "value": "channel-2"
                    },
                    {
                        "key": "packet_dst_port",
                        "value": "transfer"
                    },
                    {
                        "key": "packet_dst_channel",
                        "value": "channel-2188"
                    },
                    {
                        "key": "packet_channel_ordering",
                        "value": "ORDER_UNORDERED"
                    },
                    {
                        "key": "packet_connection",
                        "value": "connection-3"
                    }
                ]
            },
            {
                "type": "transfer",
                "attributes": [
                    {
                        "key": "recipient",
                        "value": "core12k2pyuylm9t7ugdvz67h9pg4gmmvhn5vvgafk0"
                    },
                    {
                        "key": "sender",
                        "value": "core10zt2r5p2zh9ltcyg98zt2gtmcypkqgq3qsfj74"
                    },
                    {
                        "key": "amount",
                        "value": "52931ucore"
                    }
                ]
            }
        ]
    }
]
```
{% endcode %}

### 14. Verify Successful Transfer to Osmosis <a href="#id-14-verify-successful-transfer-to-osmosis" id="id-14-verify-successful-transfer-to-osmosis"></a>

Now we can see our tokens have arrived, and are named "CORE on Osmosis", and have an IBC label.

<div align="center">

<img src="https://docs.coreum.dev/assets/success.e6712391.png" alt="Successful Transfer to Osmosis">

</div>

***

By following these steps, you should successfully send an IBC transfer to osmosis using the Keplr wallet on Coreum. Remember to always double-check details before confirming transactions to ensure the security and accuracy of your transfers.

\
