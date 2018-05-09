# qed-docs

In order to use the QED REST API, you must obtain an API Key. Please get in contact: mail@qed.digital

## Usage

Each HTTP request has to be authenticated by an API key. The API key is submitted via the `Authorization` header, i.e.

```
curl -X POST \
    https://api.qed.digital/...
    -H 'Authorization: app-token l1MttwRc4SJcuQwDabcDFF23noLjgEyUFmWZZZIgCVekwlOmBmGtDtaEvxJHBuM3'
```

### Storing a hash to the blockchain

An array of 32 bytes can be stored to the blockchain. In order to make the proof of existence of your file collision-free we highly recommend to use SHA256 as the hashing algorithm to securely create a checksum for your file. The hash is expected to be encoded as a hex digest.

#### Endpoint

`https://api.qed.digital/v1/reports/`

#### Data

|Key|Type|Description|Example|
|---|-----------|-------|----|
| ``name`` | `string` | an arbitrary title | "My new appartement"
| `note` | `string` | an arbitrary note | "This is so amazing."
| `hash` | `string` | hex encoded string representing a byte array (maximum bytes 32) | "646412deadbeef8812"

#### cURL example
```
curl -X POST \
    https://api.qed.digital/v1/reports/ \
    -H 'Authorization: app-token l1MttwRc4SJcuQwDabcDFF23noLjgEyUFmWZZZIgCVekwlOmBmGtDtaEvxJHBuM3' \
    -H 'Content-Type: application/json' \
    -d '{
        "name": "My new appartement", // an arbitrary title as string,
        "note": "This is so amazing.", // an arbitrary note as string,
        "hash": "deadbeef" // hex encoded string representing a byte array (maximum bytes 32)
    }'
```

#### Output

Due to the asynchronous nature of the blockchain interaction a report can have different states which are directly related to the transaction state on the blockchain. At which time a blockchain transaction is available and confirmed depends on both low level implementation details of a concrete blockchain and the high level SDKs wrapping blockchain interaction functionalities. For a more detailed description of the report resource, please jump to the [report detail output section](#detail-output).

```
{
  "slug": "U1FKSif",
  "status": "pending", // "pending"|"confirmed"|"failed"
  "name": "My new appartement",
  "note": "This is so amazing",
  "hash": "deadbeef",
  "tx": {
     "hash": "af051c4d7a44a43459661b0943153d929f5370beacfeedc6cde6545fae981969" // deterministic hash of the transaction which can be used to access details of the transaction
     // More transaction details as soon as the transaction has been successfully processed by the blockchain network
  }
}
```


### Retrieving the report status

#### Endpoint

`https://api.qed.digital/v1/reports/<slug>`

#### cURL example 

```
curl -X GET \
    https://api.qed.digital/v1/reports/U1FKSif \
    -H 'Authorization: app-token l1MttwRc4SJcuQwDabcDFF23noLjgEyUFmWZZZIgCVekwlOmBmGtDtaEvxJHBuM3' \
    -H 'Content-Type: application/json'
```

#### Output<a name="detail-output"></a>

```
{
  "slug": "U1FKSif",
  "status": "confirmed",
  "name": "My new appartement",
  "note": "This is so amazing",
  "hash": "deadbeef",
  "tx": {
     "hash": "af051c4d7a44a43459661b0943153d929f5370beacfeedc6cde6545fae981969"
     "data": {...}, // generic JSON object (Blockchain implementation specific)
     "block_height": 8872466,
     "block_time": "2018-05-09T11:32:03Z",
     "explorer_url": "https://testnet.steexp.com/tx/642b0791738b1d202cb2e6d3c7fd811310cf0d990baba1a2b418cc4123a970b6",
     "network_name": "Stellar Testnet",
     "network_id": "cee0302d59844d32bdca915c8203dd44b33fbb7edc19051ea37abedf28ecd472"
  }
}
```


