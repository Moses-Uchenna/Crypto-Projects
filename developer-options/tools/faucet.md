# Faucet

Welcome to the Coreum Faucet Page – the gateway to funding your account within the Coreum Network. Whether you're a seasoned user or new to the world of decentralized finance, this page provides you with straightforward options to generate and fund your Coreum wallet.

## **Creating Your Account:**

You can fund your account here.

If you don't have an account yet, fear not! There are three convenient ways to get started:

1. **For All Users:**
   * Head to the [wallet page](wallets.md) and connect (install) your preferred wallet. From there, follow the intuitive steps to create your Coreum account.
2. **For Advanced Users:**
   * Click the `Generate Funded Wallet` button below. This option is designed for users comfortable with advanced functionalities.
3.  **For CLI Enthusiasts:**

    * If you're a fan of command-line interfaces, run the `cored` CLI command provided:



    {% code lineNumbers="true" %}
    ```sh
    bashCopy codeexport COREUM_CHAIN_ID="{Chain ID}"
    cored keys add <name-of-the-key> --chain-id=$COREUM_CHAIN_ID
    ```
    {% endcode %}


4. The`{Chain ID}` variable depends on the network, and you can find the appropriate value on the [network variables page](https://docs.coreum.dev/tutorials/network-variables.html).

## **Network-Specific Instructions:**

### **Mainnet:**

* Generate your address on the [wallet page](wallets.md) and explore the list of Coreum Markets [here](https://coinmarketcap.com/currencies/coreum/markets/). Make sure to check if your preferred centralized exchange (CEX) supports withdrawals into the Coreum Network before initiating any transactions.

### **Testnet:**

* Utilize the **Generate Funded Wallet** option to secure testnet funds for your experimentation and testing needs. Additionally, you can request funds to accelerate your testing process [here](https://docs.coreum.dev/tools-ecosystem/faucet.html#testnet).

### **Devnet:**

* Similar to the testnet, the **Generate Funded Wallet** feature is available for devnet. Request funds [here](https://docs.coreum.dev/tools-ecosystem/faucet.html#devnet) if needed to facilitate your development activities.
