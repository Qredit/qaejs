# QAEJS

SLPJS is a JavaScript Library for building [Simple Ledger Protocol (SLP)](https://github.com/simpleledger/slp-specification/blob/master/slp-token-type-1.md) token transactions.  

GENESIS, MINT, and SEND transaction functions are currently supported. 

[![NPM](https://nodei.co/npm/slpjs.png)](https://nodei.co/npm/slpjs/)



# Installation

#### For node.js
`npm install slpjs`

#### For browser
```<script src='https://unpkg.com/slpjs'></script>```



# Example Usage

The following examples show how this library should be used.

NOTE: The [BigNumber.js library](https://github.com/MikeMcl/bignumber.js) is used to avoid precision issues with numbers having more than 15 significant digits.

NOTE: For fast validation performance all of the following examples show how to use SLPJS using default SLP validation via rest.bitcoin.com.  See [Local Validation](#local-validation) section for instructions on how to validate SLP locally.

## Get Balances

```js
// Install BITBOX-SDK v3.0.2+ instance for blockchain access
// For more information visit: https://www.npmjs.com/package/bitbox-sdk
const BITBOXSDK = require('bitbox-sdk/lib/bitbox-sdk').default
const slpjs = require('slpjs');

// FOR TESTNET UNCOMMENT
let addr = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
const BITBOX = new BITBOXSDK({ restURL: 'https://trest.bitcoin.com/v2/' });

// FOR MAINNET UNCOMMENT
// let addr = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";
// const BITBOX = new BITBOXSDK({ restURL: 'https://rest.bitcoin.com/v2/' });

const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX);

let balances;
(async function() {
  balances = await bitboxNetwork.getAllSlpBalancesAndUtxos(addr);
  console.log("balances: ", balances);
})();

// RETURNS ALL BALANCES & UTXOs: 
// { satoshis_available_bch: 190889,
//   satoshis_locked_in_slp_baton: 546,
//   satoshis_locked_in_slp_token: 1092,
//   satoshis_in_invalid_token_dag: 0,
//   satoshis_in_invalid_baton_dag: 0,
//   slpTokenBalances: {
//      '1cda254d0a995c713b7955298ed246822bee487458cd9747a91d9e81d9d28125': BigNumber { s: 1, e: 3, c: [ 1000 ] },
//      '047918c612e94cce03876f1ad2bd6c9da43b586026811d9b0d02c3c3e910f972': BigNumber { s: 1, e: 2, c: [ 100 ] } 
//   },
//   slpTokenUtxos: [ ... ],
//   slpBatonUtxos: [ ... ],
//   invalidTokenUtxos: [ ... ],
//   invalidBatonUtxos: [ ... ],
//   nonSlpUtxos: [ ... ]
// }
```



## GENESIS - New token creation (fungible)

GENESIS is the most simple type of SLP transaction since no special inputs are required.

```js
// Install BITBOX-SDK v3.0.2+ instance for blockchain access
// For more information visit: https://www.npmjs.com/package/bitbox-sdk
const BITBOXSDK = require('bitbox-sdk/lib/bitbox-sdk').default
const BigNumber = require('bignumber.js');
const slpjs = require('slpjs');

// FOR TESTNET UNCOMMENT
const BITBOX = new BITBOXSDK({ restURL: 'https://trest.bitcoin.com/v2/' });
const fundingAddress           = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
const fundingWif               = "cVjzvdHGfQDtBEq7oddDRcpzpYuvNtPbWdi8tKQLcZae65G4zGgy";
const tokenReceiverAddress     = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
const bchChangeReceiverAddress = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
// For unlimited issuance provide a "batonReceiverAddress"
const batonReceiverAddress     = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";

// FOR MAINNET UNCOMMENT
// const BITBOX = new BITBOXSDK({ restURL: 'https://rest.bitcoin.com/v2/' });
// const fundingAddress           = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";  // <-- must be simpleledger format
// const fundingWif               = "L3gngkDg1HW5P9v5GdWWiCi3DWwvw5XnzjSPwNwVPN5DSck3AaiF";     // <-- compressed WIF format
// const tokenReceiverAddress     = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";  // <-- must be simpleledger format
// const bchChangeReceiverAddress = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";  // <-- cashAddr or slpAddr format
// // For unlimited issuance provide a "batonReceiverAddress"
// const batonReceiverAddress     = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";

const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX);

// 1) Get all balances at the funding address.
let balances; 
(async function() {
  balances = await bitboxNetwork.getAllSlpBalancesAndUtxos(fundingAddress);
  console.log("'balances' variable is set.");
  console.log('BCH balance:', balances.satoshis_available_bch);
})();

// WAIT FOR NETWORK RESPONSE...

// 2) Select decimal precision for this new token
let decimals = 2;
let name = "Awesome SLPJS README Token";
let ticker = "SLPJS";
let documentUri = "info@simpleledger.io";
let documentHash = null
let initialTokenQty = 1000000

// 3) Calculate the token quantity with decimal precision included
initialTokenQty = (new BigNumber(initialTokenQty)).times(10**decimals);

// 4) Set private keys
balances.nonSlpUtxos.forEach(txo => txo.wif = fundingWif)

// 5) Use "simpleTokenGenesis()" helper method
let genesisTxid;
(async function(){
    genesisTxid = await bitboxNetwork.simpleTokenGenesis(
        name, 
        ticker, 
        initialTokenQty,
        documentUri,
        documentHash,
        decimals,
        tokenReceiverAddress,
        batonReceiverAddress,
        bchChangeReceiverAddress,
        balances.nonSlpUtxos
        )
    console.log("GENESIS txn complete:",genesisTxid)
})();

```

## GENESIS - NFT1 creation (non-fungible token per [spec](https://github.com/simpleledger/slp-specifications/blob/master/NFT.md))

Non-fungible tokens can be created with the `simpleNFT1Genesis` method with these parameters:

```js
// Install BITBOX-SDK v3.0.2+ instance for blockchain access
// For more information visit: https://www.npmjs.com/package/bitbox-sdk
const BITBOXSDK = require('bitbox-sdk/lib/bitbox-sdk').default
const BigNumber = require('bignumber.js');
const slpjs = require('slpjs');

const BITBOX = new BITBOXSDK({ restURL: 'https://rest.bitcoin.com/v2/' });
const fundingAddress           = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu"; // <-- must be simpleledger format
const fundingWif               = "L3gngkDg1HW5P9v5GdWWiCi3DWwvw5XnzjSPwNwVPN5DSck3AaiF";    // <-- compressed WIF format
const tokenReceiverAddress     = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu"; // <-- must be simpleledger format
const bchChangeReceiverAddress = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu"; // <-- cashAddr or slpAddr format

const parentTokenIdHex = "<32-byte token id of parent token>";

const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX);

// 1) Get all balances at the funding address.
let balances;
(async function() {
  balances = await bitboxNetwork.getAllSlpBalancesAndUtxos(fundingAddress);
  console.log("'balances' variable is set.");
  console.log('BCH balance:', balances.satoshis_available_bch);
})();

// WAIT FOR NETWORK RESPONSE...

// 2) Select decimal precision for this new NFT1 token
let name = "I'm a unique token";
let ticker = "NFT1";

// 3) Set private keys for BCH & Tokens
balances.nonSlpUtxos.forEach(txo => txo.wif = fundingWif)
balances.slpTokenUtxos[parentTokenId][0].wif = fundingWif;
let inputs = balances.nonSlpUtxos.concat(balances.slpTokenUtxos[parentTokenId][0])

// 4) Use "simpleTokenGenesis()" helper method
let genesisTxid;
(async function(){
  //tokenName: string, tokenTicker: string, parentTokenIdHex: string, tokenReceiverAddress: string, bchChangeReceiverAddress: string, inputUtxos: SlpAddressUtxoResult[]
    genesisTxid = await bitboxNetwork.simpleNFT1Genesis(
        name, 
        ticker, 
        parentTokenIdHex,
        tokenReceiverAddress,
        bchChangeReceiverAddress,
        inputs
        )
    console.log("NFT1 GENESIS txn complete:",genesisTxid)
})();

```


## MINT - Create more tokens

Adding additional tokens for a token that already exists is possible if you are in control of the minting "baton".  This minting baton is a special UTXO that gives authority to add to the token's circulating supply.  

```javascript
// Install BITBOX-SDK v3.0.2+ instance for blockchain access
// For more information visit: https://www.npmjs.com/package/bitbox-sdk
const BITBOXSDK = require('bitbox-sdk/lib/bitbox-sdk').default
const BigNumber = require('bignumber.js');
const slpjs = require('slpjs');

// FOR TESTNET UNCOMMENT
const BITBOX = new BITBOXSDK({ restURL: 'https://trest.bitcoin.com/v2/' });
const fundingAddress           = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
const fundingWif               = "cVjzvdHGfQDtBEq7oddDRcpzpYuvNtPbWdi8tKQLcZae65G4zGgy";
const tokenReceiverAddress     = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
const batonReceiverAddress     = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
const bchChangeReceiverAddress = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";
const tokenIdHexToMint = "a67e2abb2fcfaa605c6a3b0dfb642cc830b63138d85b5e95eee523fdbded4d74";
let additionalTokenQty = 1000

// FOR MAINNET UNCOMMENT
// const BITBOX = new BITBOXSDK({ restURL: 'https://rest.bitcoin.com/v2/' });
// const fundingAddress           = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";  // <-- must be simpleledger format
// const fundingWif               = "L3gngkDg1HW5P9v5GdWWiCi3DWwvw5XnzjSPwNwVPN5DSck3AaiF";     // <-- compressed WIF format
// const tokenReceiverAddress     = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";  // <-- must be simpleledger format
// const batonReceiverAddress     = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";
// const bchChangeReceiverAddress = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";  // <-- cashAddr or slpAddr format
// const tokenIdHexToMint = "495322b37d6b2eae81f045eda612b95870a0c2b6069c58f70cf8ef4e6a9fd43a";
// let additionalTokenQty = 1000

const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX);

// 1) Get all balances at the funding address.
let balances; 
(async function() {
  balances = await bitboxNetwork.getAllSlpBalancesAndUtxos(fundingAddress);
  console.log("'balances' variable is set.");
  if(balances.slpBatonUtxos[tokenIdHexToMint])
    console.log("You have the minting baton for this token");
  else
    throw Error("You don't have the minting baton for this token");
})();

// 2) Fetch critical token decimals information using bitdb
let tokenDecimals;
(async function() {
    const tokenInfo = await bitboxNetwork.getTokenInformation(tokenIdHexToMint);
    tokenDecimals = tokenInfo.decimals; 
    console.log("Token precision: " + tokenDecimals.toString());
})();

// WAIT FOR ASYNC METHOD TO COMPLETE

// 3) Multiply the specified token quantity by 10^(token decimal precision)
let mintQty = (new BigNumber(additionalTokenQty)).times(10**tokenDecimals)

// 4) Filter the list to choose ONLY the baton of interest 
// NOTE: (spending other batons for other tokens will result in losing ability to mint those tokens)
let inputUtxos = balances.slpBatonUtxos[tokenIdHexToMint]

// 5) Simply sweep our BCH (non-SLP) utxos to fuel the transaction
inputUtxos = inputUtxos.concat(balances.nonSlpUtxos);

// 6) Set the proper private key for each Utxo
inputUtxos.forEach(txo => txo.wif = fundingWif)

// 7) MINT token using simple function
let mintTxid;
(async function() {
    mintTxid = await bitboxNetwork.simpleTokenMint(
        tokenIdHexToMint, 
        mintQty, 
        inputUtxos, 
        tokenReceiverAddress, 
        batonReceiverAddress, 
        bchChangeReceiverAddress
        )
    console.log("MINT txn complete:",mintTxid);
})();

```



## SEND - Send tokens

This example shows the general workflow for sending an existing token.

```js
// Install BITBOX-SDK v3.0.2+ instance for blockchain access
// For more information visit: https://www.npmjs.com/package/bitbox-sdk
const BITBOXSDK = require('bitbox-sdk/lib/bitbox-sdk').default;
const BigNumber = require('bignumber.js');
const slpjs = require('slpjs');

// FOR MAINNET UNCOMMENT
// const BITBOX = new BITBOXSDK({ restURL: 'https://rest.bitcoin.com/v2/' });
// const fundingAddress           = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu";     // <-- must be slpAddr format
// const fundingWif               = "L3gngkDg1HW5P9v5GdWWiCi3DWwvw5XnzjSPwNwVPN5DSck3AaiF";        // <-- compressed WIF format
// const tokenReceiverAddress     = [ "simpleledger:qplrqmjgpug2qrfx4epuknvwaf7vxpnuevyswakrq9" ]; // <-- must be slpAddr format
// const bchChangeReceiverAddress = "simpleledger:qrxx766pq856sm7l0nny5wtygnqlga52dvvhy9smlh";     // <-- cashAddr or slpAddr format
// let tokenId = "347975ff25c8ca30b309f229cd6f1968d6c37bf81ef8f4d0b3b28cb59f48acf3";
// let sendAmounts = [ 1 ];

// FOR TESTNET UNCOMMENT
const BITBOX = new BITBOXSDK({ restURL: 'https://trest.bitcoin.com/v2/' });
const fundingAddress           = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";   // <-- must be slpAddr format
const fundingWif               = "cVjzvdHGfQDtBEq7oddDRcpzpYuvNtPbWdi8tKQLcZae65G4zGgy"; // <-- compressed WIF format
const tokenReceiverAddress     = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";   // <-- must be slpAddr format
const bchChangeReceiverAddress = "slptest:qpwyc9jnwckntlpuslg7ncmhe2n423304ueqcyw80l";   // <-- cashAddr or slpAddr format
let tokenId = "78d57a82a0dd9930cc17843d9d06677f267777dd6b25055bad0ae43f1b884091";
let sendAmounts = [ 10 ];

const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX);

// 1) Fetch critical token information
let tokenDecimals;
(async function() {
    const tokenInfo = await bitboxNetwork.getTokenInformation(tokenId);
    tokenDecimals = tokenInfo.decimals; 
    console.log("Token precision: " + tokenDecimals.toString());
})();

// Wait for network responses...

// 2) Check that token balance is greater than our desired sendAmount
let balances; 
(async function() {
  balances = await bitboxNetwork.getAllSlpBalancesAndUtxos(fundingAddress);
  console.log("'balances' variable is set.");
  console.log("Token balance:", balances.slpTokenBalances[tokenId].toNumber() / 10**tokenDecimals);
})();

// Wait for network responses...

// 3) Calculate send amount in "Token Satoshis".  In this example we want to just send 1 token unit to someone...
sendAmounts = sendAmounts.map(a => (new BigNumber(a)).times(10**tokenDecimals));  // Don't forget to account for token precision

// 4) Get all of our token's UTXOs
let inputUtxos = balances.slpTokenUtxos[tokenId];

// 5) Simply sweep our BCH utxos to fuel the transaction
inputUtxos = inputUtxos.concat(balances.nonSlpUtxos);

// 6) Set the proper private key for each Utxo
inputUtxos.forEach(txo => txo.wif = fundingWif);

// 7) Send token
let sendTxid;
(async function(){
    sendTxid = await bitboxNetwork.simpleTokenSend(
        tokenId, 
        sendAmounts, 
        inputUtxos, 
        tokenReceiverAddress, 
        bchChangeReceiverAddress
        )
    console.log("SEND txn complete:", sendTxid);
})();
```


## BURN Tokens for a certain Token Id

This example shows the general workflow for sending an existing token.

```javascript

// Install BITBOX-SDK v3.0.2+ instance for blockchain access
// For more information visit: https://www.npmjs.com/package/bitbox-sdk
const BITBOXSDK = require('bitbox-sdk/lib/bitbox-sdk').default
const BigNumber = require('bignumber.js');
const slpjs = require('slpjs');

const BITBOX = new BITBOXSDK({ restURL: 'https://rest.bitcoin.com/v2/' });
const fundingAddress           = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu"; // <-- must be slpAddr format
const fundingWif               = "L3gngkDg1HW5P9v5GdWWiCi3DWwvw5XnzjSPwNwVPN5DSck3AaiF";    // <-- compressed WIF format
const bchChangeReceiverAddress = "simpleledger:qrhvcy5xlegs858fjqf8ssl6a4f7wpstaqnt0wauwu"; // <-- cashAddr or slpAddr format
let tokenId = "495322b37d6b2eae81f045eda612b95870a0c2b6069c58f70cf8ef4e6a9fd43a";
let burnAmount = 102;

const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX);

// 1) Fetch critical token information
let tokenDecimals;
(async function() {
    const tokenInfo = await bitboxNetwork.getTokenInformation(tokenId);
    tokenDecimals = tokenInfo.decimals; 
    console.log('Token precision:', tokenDecimals.toString());
})();

// 2) Check that token balance is greater than our desired sendAmount
let balances; 
(async function() {
  balances = await bitboxNetwork.getAllSlpBalancesAndUtxos(fundingAddress);
  console.log("'balances' variable is set.");
  console.log('Token balance:', balances.slpTokenBalances[tokenId].toNumber() / 10**tokenDecimals)
})();

// Wait for network responses...

// 3) Calculate send amount in "Token Satoshis".  In this example we want to just send 1 token unit to someone...
let amount = (new BigNumber(burnAmount)).times(10**tokenDecimals);  // Don't forget to account for token precision

// 4) Get all of our token's UTXOs
let inputUtxos = balances.slpTokenUtxos[tokenId];

// 5) Simply sweep our BCH utxos to fuel the transaction
inputUtxos = inputUtxos.concat(balances.nonSlpUtxos);

// 6) Set the proper private key for each Utxo
inputUtxos.forEach(txo => txo.wif = fundingWif)

// 7) Send token
let sendTxid;
(async function(){
    sendTxid = await bitboxNetwork.simpleTokenBurn(
        tokenId, 
        amount, 
        inputUtxos, 
        bchChangeReceiverAddress
        )
    console.log("BURN txn complete:",sendTxid);
})();
```


## SLP Address Conversion

```javascript
let Utils = require('slpjs').Utils;

let slpAddr = Utils.toSlpAddress("bitcoincash:qzat5lfxt86mtph2fdmp96stxdmmw8hchyxrcmuhqf");
console.log(slpAddr);
// simpleledger:qzat5lfxt86mtph2fdmp96stxdmmw8hchy2cnqfh7h

let cashAddr = Utils.toCashAddress(slpAddr);
console.log(cashAddr);
// bitcoincash:qzat5lfxt86mtph2fdmp96stxdmmw8hchyxrcmuhqf
```

# Local Validation

The examples above use default remote validation powered by rest.bitcoin.com `/slp/validateTxid` endpoint, and is enabled with the following code:
`const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX);`

You can override this behavior to validate SLP transaction locally by replacing that code with the following lines:
```js
const getRawTransactions = async function(txids) { return await BITBOX.RawTransactions.getRawTransaction(txids) }
const slpValidator = new slpjs.LocalValidator(BITBOX, getRawTransactions);
const bitboxNetwork = new slpjs.BitboxNetwork(BITBOX, slpValidator);
```

## Example Local Validator

This example validates a SLP transaction locally by downloading all required raw transactions from remote node, in this case BITBOX REST API.

```js

const BITBOXSDK = require('bitbox-sdk/lib/bitbox-sdk').default
const BITBOX = new BITBOXSDK({ restURL: 'https://rest.bitcoin.com/v2/' });
const slpjs = require('slpjs');

const getRawTransactions = async function(txids) { return await BITBOX.RawTransactions.getRawTransaction(txids) }
const slpValidator = new slpjs.LocalValidator(BITBOX, getRawTransactions);

// Result = false
let txid = "c2efd10e40d08c9d3867ba2c8eb69f2a9e0db35d9e1219f59c839151446d1d38";

// Result = true
//let txid = "44b2567e6a1c9f8d6ac5256ea4be02c31904d63cbe0f7a299c0ee28521443764";

let isValid;
(async function() {
  console.log("Validating:", txid);
  console.log("This may take a several seconds...");
  isValid = await slpValidator.isValidSlpTxid(txid);
  console.log("Validation result: ", isValid);
  if(!isValid)
    console.log("This transaction is invalid because:", slpValidator.cachedValidations[txid].invalidReason);
})();

```

# Caveats

* All SLPJS methods require token quantities to be expressed in the smallest possible unit of account for the token (i.e., token satoshis).  This requires the token's precision to be used to calculate the quantity. For example, token having a decimal precision of 9 sending an amount of 1.01 tokens would need to first calculate the sending amount using `1.01 x 10^9 => 1010000000`.



# Building & Testing

Building this project creates lib/*.js files and then creates browserified versions in the dist folder.

## Requirements
Running the unit tests require node.js v8.15+. 

## Build
`npm run build`

## Test
`npm run test`
