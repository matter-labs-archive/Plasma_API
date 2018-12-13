# Public Rest API for Plasma

## General information

* Endpoints:
    * 1-st type - Plasma UTXOs and TXs:
        * Dev: https://plasma-testnet.thematter.io/
        * Prod: https://plasma.thematter.io/
    * 2-nd type - Blocks storage:
        * Dev: https://plasma-testnet.ams3.digitaloceanspaces.com/plasma/
        * Prod: https://plasma.ams3.digitaloceanspaces.com/plasma/
* *Plasma UTXOs and TXs* endpoints return either a JSON object or array
* Data is returned in **ascending** order. Oldest first, newest last
* All time and timestamp related fields are in milliseconds
* *Blocks storage* endpoints return a binary file
* *HTTP 4XX* return codes are used for for malformed requests; the issue is on the sender's side
* *HTTP 5XX* return codes are used for internal errors; the issue is on Plasma's side. It is important to **NOT** treat this as a failure operation; the execution status is **UNKNOWN** and could have been a success
* Any endpoint can return an ERROR
* For endpoints, parameters must be sent as a *query string* or in the *request body* with content type *application/json*

## Signed payload

* Some endpoints require to send payload which must be sign with *Private key* - *signature*
* *Private key* is an [*Ethereum account private key*](https://theethereum.wiki/w/index.php/Accounts,_Addresses,_Public_And_Private_Keys,_And_Tokens)
* *Private keys* **are case sensitive**
* *Signature* is sent in the *request body*.
* To create *signature* use [*ECDSA*](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) algorithm on [ *secp256k1*](https://en.bitcoin.it/wiki/Secp256k1). Use your *Ethereum private key* as the key and *totalParams* as the value for the sign operation
* The *signature* is not **case sensitive**
* *totalParams* must be in the *request body*
* Example:
    * You have some *params* you need to sign and send:
        * a1
        * a2
        * a3
    * 1st - place them into *Array* [a1, a2, a3]
    * 2nd - encode them into [**RLP**](https://github.com/ethereum/wiki/wiki/RLP) -> You'll get **Data RLP object**
    * Sign this **object** with your *Private key* and get signature params: **v, r, s**
    * Again encode into **RLP**:
        * a1
        * a2
        * a3
        * v
        * r
        * s
    * Final **RLP object** can be used as *Signed* data to send

## Main part

### Get UTXOs list

* **POST** 1st type /api/v1/listUTXOs

* **Description:**
Get [UTXOs](https://en.wikipedia.org/wiki/Unspent_transaction_output) list for account

* **Parameters:** NONE

* **Request body:** application/json

```
{
    blockNumber: 1
    for: "0x0A8dF54352eB4Eb6b18d0057B15009732EfB351c"
    limit: 50
    outputNumber: 0
    transactionNumber: 0
}

```
* **Response body:** application/json
```
{
    "error":false,
    "utxos":[
        {
            "blockNumber":212,
            "transactionNumber":0,
            "outputNumber":0,
            "value":"1000000000000000" //In Wei(https://en.wikipedia.org/wiki/Ethereum#Ether)
        },
        ...
    ]
}
```

### Send transaction

* **POST** 1st type /api/v1/sendRawTX

* **Description:**
Send transaction to Plasma.
Transaction structure described [here](https://github.com/matterinc/plasma.js#transaction-structure).
You will need to:
    - Build [**RLP Transaction object**](https://github.com/matterinc/plasma.js#transaction)
    - **Sign it**
    - Build [**RLP SignedTransaction object**](https://github.com/matterinc/plasma.js#signed-transaction)
    - Make **hex string** from this **RLP SignedTransaction**
    - Place **hex string** into **"tx"** field of the *request body*

* **Parameters:** NONE

* **Request body:** application/json

```
{
    "tx": "0xf8e6f8a101edec84000000da840000000000a000000000000000000000000000000000000000000000000022b1c8c1227a0000f870f70094832a630b949575b87c0e3c00f624f773d9b160f4a000000000000000000000000000000000000000000000000000038d7ea4c68000f701940a8df54352eb4eb6b18d0057b15009732efb351ca000000000000000000000000000000000000000000000000022ae3b427db380001ca0f7fc909af6f325bb4a493f48a12e253a8310c5afd2813769e75355b6d8f7f118a0526cc97b93849e40830d4bbd6acf75d07849267936aa1da18b1391c4a3df067b"
}

```
* **Response body:** application/json
```
{
    "error":false,
    "accepted":true
}
```

### Get Block by number

* **GET** 2st type /{blockNumber}

* **Description:**
Get block by block number

* **Parameters:**
    * **blockNumber**:
        * *Type*: Integer
        * *In*: Path
        * *Description*: Block number
        * *Example*: 123

* **Response body:** application/octet-stream
*Binary file with [block structure](https://github.com/matterinc/plasma.js#block-structure)*
