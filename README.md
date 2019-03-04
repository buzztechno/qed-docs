# qed-docs

The QED REST API provides an easy way to leverage blockchain technology attaching immutable timestamps to your files.

![QED Diagram](images/qed.png)

For IOS we also provide an SDK wrapper around the API. [IOS QEDSDK](IOSQEDSDK/index.html)

## Usage

### Get your API Key

Each HTTP request has to be authenticated by an **API key**. In order to use the `QED REST API`, you must obtain an **API Key**. Please get in contact: [mail@qed.digital](mailto:mail@qed.digital).

The API key is submitted via the `Authorization` header as `app-token <API_KEY>`, i.e.
```
curl -X POST \
    https://api.qed.digital/...
    -H 'Authorization: app-token <API_KEY>'
```

### Storing a hash to the blockchain

An array of 32 bytes can be stored to the blockchain. In order to make the hash of your file collision-free we highly recommend to use SHA256 as the hashing algorithm to securely create a checksum for your file. The hash is expected to be encoded as a hex digest.

#### Endpoint

`https://api.qed.digital/v1/reports/`

#### Data

|Key|Type|Required|Description|Example|
|---|----|--------|-------|----|
| `hash` | `string` | x | hex encoded string representing a byte array (maximum bytes 32) | "646412deadbeef8812" |
| `provable_files` | [`ProvableFile`](#provable-files) list | x | A list of files represented by their meta data. Can be empty. |
| `original_created_at` | `iso datetime` | x | Timestamp when the report is bundled by the client application |
| `name` | `string` | - | an arbitrary title | "My new appartement" |
| `note` | `string` | - | an arbitrary note | "This is so amazing." |
| `email` | `string` | - | email field (should be set, if the issuer wants to be notified |
| `meta_data` | `JSON string` | - | additional meta data (i.e. address, geo location, references) |

#### cURL example
```
curl -X POST \
    https://api.qed.digital/v1/reports/ \
    -H 'Authorization: app-token <API_KEY>' \
    -H 'Content-Type: application/json' \
    -d '{
        "name": "My new appartement", // an arbitrary title as string,
        "note": "This is so amazing.", // an arbitrary note as string,
        "hash": "deadbeef", // hex encoded string representing a byte array (maximum bytes 32)
        "email": "tenant@example.com",  // an email as string
        "original_created_at": "2018-10-24T13:58:57.344Z",
        "provable_files": [{
             "hash": "968b0ef...", // SHA256 hash representation of the file binary
             "file_name": "my-file.jpg"
        }],
        "meta_data": {   // optional meta data
            "property": {
                "location": {
                    "address": {
                        "number": "property",
                        "street": "Bahnhofstrasse 2",
                        "country": "Belgium",
                        "postalCode": "10245"
                    }
                }
            },
            "transaction": {
                "contractStartDate": "27 Feb 2020",
                "relationToProperty": "Tenant"
            },
            "also":{
               "arbitrary_data": {
                   "status": "possible",
                   "id": 27563
               } 
            }
        }
    }'
```

#### Output

Due to the asynchronous nature of the blockchain interaction, a report can have different states which are directly related to the transaction state on the blockchain. At which time a blockchain transaction is available and confirmed depends on both low level implementation details of the concrete chain and the high level SDKs wrapping blockchain interaction functionalities.

For a more detailed description of the transaction resource, please jump to the [transaction section](#transaction).

```
{
  "slug": "U1FKSif",
  "status": "pending", // "pending"|"confirmed"|"failed"
  "name": "My new appartement",
  "note": "This is so amazing",
  "hash": "deadbeef",
  "provable_files": [{
     "slug": "BZzRxnX",
     "hash": "968b0ef...", // SHA256 hash representation of the file binary
     "file_name": "my-file.jpg"
  }],
  "meta_data": {
     "address": // ...
  },
  "tx": {
     "hash": "af051c4d7a44a43459661b0943153d929f5370beacfeedc6cde6545fae981969" // deterministic hash of the transaction which can be used to access details of the transaction
     // More transaction details as soon as the transaction has been successfully processed by the blockchain network
  }
}
```

### `ProvableFile` objects<a name="provable-files"></a>

`ProvableFile` objects represent the files being contained in the timestamped QED proof. Files can be kept secretly, but should be added as references of their original content. 
Keeping the file hashes as records ensures, that a report's file timestamp can be proven as long as all other report file hashes can be taken into the equation, even if all other files have been lost.

The `file` field is optional to link to a web storage.


```
{
    "slug": "BZzRxnX", //
    "hash": "968b0ef546556914662ed0376c4db10a67a03ed137c3168662edb0bab7104ed0",
    "file_name": "my-file.jpg",
    "file": "https://proof-existence-stage.s3.amazonaws.com/report-file.jpg",
}
```

### Retrieving the report status

#### Endpoint

`https://api.qed.digital/v1/reports/<slug>`

#### cURL example

```
curl -X GET \
    https://api.qed.digital/v1/reports/U1FKSif \
    -H 'Authorization: app-token <API_KEY>' \
    -H 'Content-Type: application/json'
```

#### Output<a name="report-resource"></a>

For an explanation of the `tx` object please jump to the [transaction resource](#transaction) section.

```
{
  "slug": "U1FKSif",
  "status": "confirmed",
  "name": "My new appartement",
  "note": "This is so amazing",
  "hash": "deadbeef",
  "provable_files": [{
     "slug": "BZzRxnX",
     "hash": "968b0ef...", // SHA256 hash representation of the file binary
     "file_name": "my-file.jpg",
     "file": "https://proof-existence-production.s3.amazonaws.com/report-file.jpg",
  }],
  "tx": {
     "hash": "af051c4d7a44a43459661b0943153d929f5370beacfeedc6cde6545fae981969"
     "data": {...}, // generic JSON object (Blockchain implementation specific)
     "block_height": 8872466,
     "block_time": "2018-05-09T11:32:03Z",
     "explorer_url": "https://testnet.steexp.com/tx/642b0791738b1d202cb2e6d3c7fd811310cf0d990baba1a2b418cc4123a970b6",
     "network_name": "Stellar Testnet",
     "network_id": "cee0302d59844d32bdca915c8203dd44b33fbb7edc19051ea37abedf28ecd472"
  },
  "meta_data": {
     "address": {
        "street": "Teststr.",
        "number": "1234",
        ...
     },
     "also": {
        "arbitrary_data": {
            "status": "possible",
            "id": 27563
        } 
     }
  }
}
```

### Filtering for a hash

It is also possible to query for a list of app reports that represent a given file hash. The `hash` parameter is required in order to permit access only to clients that hold a given file or know its corresponding hash.

#### Endpoint

```
https://api.qed.digital/v1/reports/?hash=...
```

#### cURL Example

```
curl -X GET \
    https://api.qed.digital/v1/reports/?hash=af051c4d7a44a43459661b0943153d929f5370beacfeedc6cde6545fae981969 \
    -H 'Authorization: app-token <API_KEY>' \
    -H 'Content-Type: application/json'
```

#### Output

A list of [Report resources](#report-resource)


### Transaction ressource<a name="transaction"></a>

| Key | Type | Description | Example |
| --- | ---- | ----------- | ------- |
| `hash` | `string` | Deterministic hash of the transaction which can be used to access details of the transaction. | "af051c4d..." |
|`data`|`JSON`|Blockchain-dependent data representation|*{"id": "fdcbb4e...", "paging_token": "38112560532193280", "hash": "fdcbb4e...", "ledger": 8873772, "created_at": "2018-05-09T13:20:53Z", "source_account": "GAS5AL...", "source_account_sequence": "36978014856151081", "fee_paid": 100, "operation_count": 1, ...}*|
|`block_height`|`integer`|Block number|8872466|
|`block_time`|`ISO 8601`|Validation time of the block which represents the time when the proof has been ultimately persisted|"2018-05-09T11:32:03Z"|
|`explorer_url`|`URL`|Points to a blockchain explorer transaction detail page |[https://testnet.steexp.com/tx/642b07917...8cc4123a970b6](https://testnet.steexp.com/tx/642b0791738b1d202cb2e6d3c7fd811310cf0d990baba1a2b418cc4123a970b6)|
|`network_name`|`string`|High level name of the concrete blockchain network| "Stellar Testnet" |
|`network_id`|`string` or `integer`|Internal identifier for the network which lies within the scope of the concrete blockchain family. It can take multiple forms depending on the blockchain.|"cee0302d59844d32b...a37abedf28ecd472"|


### PDF Generation

Generate a certificate PDF.  

#### Endpoint

`https://api.qed.digital/v1/reports/<slug>/pdf`

#### cURL example

```
curl -X POST \
    https://api.qed.digital/v1/reports/<slug>/pdf/
    -H 'Authorization: app-token <API_KEY>'
```
