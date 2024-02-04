# Using CoreumJS

In this tutorial we are going use `coreum-js` library for issuing, minting and sending FT token.

### Steps <a href="#steps" id="steps"></a>

1. Install `nodejs` and `npm`.
2. Run the following commands:

* `mkdir coreum-js-tutorial && cd coreum-js-tutorial` to create dir for the project and go into it;
* `npm init` to init the project;
* `npm install coreum-js` to install `coreum-js` library;
* `touch index.js` to create js file;

3. Put the code from the [following section](https://docs.coreum.dev/tutorials/coreum-js.html#full-code) into `index.js` file. Reffer to the comments for the explanation.
4. Generate mnemonic for the testnet network [here](https://docs.coreum.dev/tools-ecosystem/faucet.html#testnet).
5. Assign the mnemonic to the `issuerMnemonic` variable in the code.
6. Run the code: `node index.js`. Check output.

### Full code <a href="#full-code" id="full-code"></a>

{% code lineNumbers="true" %}
```sh
const { Client, Bank, FT } = require("coreum-js");

// Replace the issuerMnemonic with your own. You can generate it at https://docs.coreum.dev/tools-ecosystem/faucet.html
// Caution: do not hardcode your production mnemonic here, otherwise your funds might be stolen.
const issuerMnemonic =
    "spend build hurdle notable lemon involve summer pudding sadness fit excite canyon flock relief unfold upgrade sure film alley chest census quiz ivory birth";

// If you are using a mnemonic from this tutorial you should provide another subunit and symbol,
// since tokens within one account should be unique.
const subunit = "umyft"
const symbol =  "MYFT"

const network = "testnet";

// We need another address to send tokens to. You can replace it with your own:
const receiver = "testcore1xa6mlzdjt669600kwumk35ep0d8gvcyf9z8fww"


async function main() {
    try {

// INIT SECTION

        // Init the client and target the testnet network:
        const coreum = new Client({ network:  network}); // Other values are "devnet" and "mainnet"

        // Access the private key of the mnemonic.
        await coreum.connectWithMnemonic(issuerMnemonic);

        // Let's store the mnemonic bank address to the variable(the Client instance saves the address of the connected account for easy access)
        const issuer = coreum.address;

        // Let's define the modules we are going to use:
        const { bank, ft } = coreum.queryClients;

// ISSUING ASSET-FT SECTION

        const issueFtMsg = FT.Issue({
            issuer: issuer,
            symbol: symbol,
            subunit: subunit,
            precision: "6",
            initialAmount: "100000000",
            description: "My first FT token",
            // To get valid values for the features go inside "Issue" object and then click at "token" within "./asset/ft/v1/token" path.
            features: ["minting"],
        })
        console.log("issueFtMsg: ", issueFtMsg);

        // Let's broadcast our issueFtMsg message and check the response:
        const issueBroadcastResponse = await coreum.sendTx([issueFtMsg]);
        console.log("issueBroadcastResponse: ", issueBroadcastResponse);

        // FT denom is generated on token's creation and consist of subunit and issuer address joined with dash.
        // Let's export it for further using:
        const ftDenom = `${subunit}-${issuer}`;

        // Let's fetch the details of created tokens from the chain:
        const tokenDetails = await ft.token(ftDenom);
        console.log(`tokenDetails: `, tokenDetails);

// MINTING  section

        const mintFtMsg = FT.Mint({
            sender: issuer,
            coin: {
                denom: ftDenom,
                amount: "1",
            }
        })

        console.log("mintFtMsg: ", mintFtMsg);

        // Let's broadcast our issueFtMsg message and check the response:
        const mintBroadcastResponse = await coreum.sendTx([mintFtMsg]);
        console.log("mintBroadcastResponse: ", mintBroadcastResponse);

        // Let's check the balance of our tokens:
        const issuerBalances = await bank.allBalances(issuer);
        console.log(`issuerBalances: `, issuerBalances);
        // initial balance was 100_000_000 ftDenom, and now we minted extra 1 ftDenom, so the balance is 100_000_001 ftDenom

// BANK SEND SECTION

        // Let's initiate the bank send transaction:
        const bankSendMsg = Bank.Send({
            fromAddress: issuer,
            toAddress: receiver,
            amount: [
                {
                    denom: ftDenom,
                    // amount is defined in subunits, taking the precision into an account we are sending 1MYFT token
                    amount: "1000000",
                },
            ],
        });
        console.log("bankSendMsg: ", bankSendMsg);

        const bankSendBroadcastResponse = await coreum.sendTx([bankSendMsg]);
        console.log("bankSendBroadcastResponse: ", bankSendBroadcastResponse);

        // Let's check the receiver's balance. You should see your tokens there:
        const receiverBalances = await bank.allBalances(receiver);
        console.log(`receiverBalances: `, receiverBalances);
    } catch (e) {
        console.log(e);
    }
}

main();
```
{% endcode %}
