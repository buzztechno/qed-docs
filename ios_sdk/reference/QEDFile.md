# `QEDFile` Type Reference

`Struct`

A `QEDFile` represents a data file locally stored on the user device that needs to be time stamped and notarised on the blockchain

## Initialisation

### `init?(with:)`

`Initialiser`

Creates a `QEDFile` instance for a local file URL.

**Declaration**

```swift
init?(with fileURL : URL)
```

| Parameter | Description |
| --------- | ----------- |
| `fileURL` | The local URL for the file to reference in the `QEDFile` instance. |

**Discussion**

The [`name`](#name), [`ext`](#ext) and [`dateTaken`](#dateTaken) will be inferred from the file at `fileURL`. If the content file at `fileURL` does not exist, the initialisation fails and returns `nil`

___

### `init(withName:extension:dateTaken:)`

`Initialiser`

Creates a `QEDFile` instance with a given [`name`](#name), [`ext`](#ext) and [`dateTaken`](#dateTaken).

**Declaration**

```swift
init(withName name : String, extension ext : String, dateTaken: Date)
```

| Parameter   | Description |
| ---------   | ----------- |
| `name`      | A string representing the name of the local file without extension |
| `ext`       | A string representing the extension of the local file |
| `dateTaken` | A `Date` instance representing the date the file was created |

**Discussion**

The [`fileURL`](#fileurl) will be inferred from the default file storage [`qedDirectory`](#qeddirectory), the [`name`](#name) and the extension([`ext`](#ext)).

The [`dateTaken`](#dateTaken) is just used as metadata and will be different that the timestamp on the blockchain certificate.

___

### `init?(fromQEDAPIResponseDict:fileURL:)`

`Initialiser`

Creates a `QEDFile` instance from a Dictionary from API response.

**Declaration**

```swift
init?(fromQEDAPIResponseDict dict : [String:Any], fileURL : URL? = nil)
```

| Parameter   | Description |
| ---------   | ----------- |
| `dict`      | A dictionary representation of a `QEDAPI` qualified response with report data |
| `fileURL`   | An optional URL indicating the local storage location of the associated file.|

**Discussion**

`dict` must include `name`, `slug`, `hash`, `dateTaken` string and `status` values as a minimum otherwise the instantiation will fail and return `nil`.

If `fileURL` is `nil`, the instance [`fileURL`](#fileurl) attribute will be inferred from the default file storage [`qedDirectory`](#qeddirectory) and the calculated instance [`name`](#name) and extension([`ext`](#ext)).

If you manage local file storage on a separate location, you should pass the `fileURL` indicating the local URL for the `QEDFile` content file if existing. You can also update the [`fileURL`](#fileurl) attribute at a later time.

## Default Storage Directories

### `qedDirectory`

`static property `

Default storage directory for the `QEDFile` content file in the app's bundle.

**Declaration**

```swift
static var qedDirectory : URL
```

**Discussion**

You can use this directory to store the files you want to represent as `QEDFile`'s for notarisation.
If you use this default directory, then you can create `QEDFile` instances using [`init(withName:extension:dateTaken:)`](#initwithnameextensiondatetaken), and the local [`fileURL`](#fileurl) location will be assumed to be on the root level of this directory.

 ___

 ### `pdfDirectory`

 `static property `

Storage dictionary for the downloaded PDF files in the app bundle.

**Declaration**

 ```swift
 static var pdfDirectory : URL
 ```

 **Discussion**

 When downloading PDF certificates for a `QEDFile` using the [`QEDAPI`](QEDAPI.md) class, they will be automatically stored on the root level of this local directory.

 You can get the exact PDF url by accessing the [`pdfURL`](#pdfurl) property of the `QEDFile`.

## File Attributes and Metadata

### `name`

`Instance Property`

The name for the associated file without extension

**Declaration**

```swift
var name : String
```
___

### `ext`

`Instance Property`

A string representing the type of file (i.e the extension).

**Declaration**

```swift
var ext : String
```
___

### `dateTaken`

`Instance Property`

The client date when the file was created.

**Declaration**

```swift
var dateTaken : Date
```

**Discussion**

Used only as metadata.

Will differ from the timestamp on the blockchain certificate.
___

### `note`

`Instance Property`

An optional string with a note about the file.

**Declaration**

```swift
var note : String?
```

**Discussion**

Used only as metadata.

## Synch status with backend and blockchain

### `slug`

`Instance Property`

A unique string that identifies the reported file in the backend.

**Declaration**

```swift
var slug : String?
```

**Discussion**

When initially created, this value is `nil`.
When using the [`QEDAPI.sendReport(for:completion:)](QEDAPI.md#sendreportforcompletion) method to submit reports, this attribute gets updated based on response from server.
___

### `isSynchedWithServer`

`Instance Property`

Indicates if the file has already been reported to the server. (i.e. if it has a [`slug`](#slug))

**Declaration**

```swift
var isSynchedWithServer : Bool
```
___

### `proofStatus`

`Instance Property`

The current status of the notarisation on the blockchain as reported by the backend.

**Declaration**

```swift
var proofStatus : ProofStatus
```

**Discussion**

`ProofStatus` enum possible values are:
- `undefined`: The file has not been requested for certification yet
- `pending`: The report has been submitted to blockchain but is not yet confirmed
- `confirmed`: Report confirmed in blockchain
- `failed`: Report was submitted to blockchain but failed

## Managing the Associated File Hash

### `hash`

`Instance Property`

The hash to be stored on the blockchain as proof of existence of the associated file.

**Declaration**

```swift
var hash : String?
```

**Discussion**

This value must not be `nil` when issuing a report with the [`QEDAPI.sendReport(for:completion:)](QEDAPI.md#sendreportforcompletion) method.

It is recommended to use _SHA256_ to calculate the hash, and the convenience method [`withCalculatedHash()`](#withcalculatedhash) will take care of this for you.

___

### `withCalculatedHash()`

`Instance Method`

Returns a `QEDFile` instance with the [`hash`](#hash) calculated from the file at [`localFileLocation`](#localfilelocation).

**Declaration**

```swift
func withCalculatedHash() -> QEDFile
```

**Return Value**

Returns a copy of the current instance where the _SHA256_ [`hash`](#hash) has been calculated from the data in file at [`localFileLocation`](#localfilelocation).

If there is no local file (i.e. `localFileLocation == nil`) then the same unaffected instance is returned (i.e. `self`)


## Managing the Local File Storage

### `fileURL`

`Instance Property`

The URL where the file is meant to be locally stored.

**Declaration**

```swift
var fileURL : URL
```

**Discussion**

There is no guarantee that a file actually exists at this URL. See [`localFileLocation`](#localfilelocation).

___

### `localFileLocation`

`Instance Property`

The actual location url for the local file.

**Declaration**

```swift
var localFileLocation : URL?
```

**Discussion**

Will return `nil` if the file addressed by the [`fileURL`](#fileurl) does not exist.

___

### `hasLocalFile`

`Instance Property`

Indicates if the instance references and existing local file at [`fileURL`](#fileurl).

**Declaration**

```swift
var hasLocalFile : Bool
```

## Managing the Certificate PDF File Storage

### `pdfURL`

`Instance Property`

The URL for the PDF certificate file when locally stored.

**Declaration**

```swift
var pdfURL : URL
```

**Discussion**

This property always holds a URL value without checking if the file actually exist.
Use [`hasLocalPDF`](#haslocalpdf) to check for PDF file existence
___

### `hasLocalPDF`

`Instance Property`

Indicates if the instance references and existing local certificate file at [`pdfURL`](#pdfurl)

**Declaration**

```swift
var hasLocalPDF : Bool
```

## Interfacing with [`QEDAPI`](QEDAPI)

### `dictionaryRepresentation()`

`Instance Method`

Returns a dictionary representation of the instance.

**Declaration**

```swift
func dictionaryRepresentation() -> [String:String]
```

**Return Value**

Returns a dictionary compatible with the [`QEDAPI`](QEDAPI.md) reporting functionality.

The keys are `String` instances as described on [_QED API_](../../README.md), the values are `String` instances derived from the respective instance attributes.

## Interacting with other instances

### `merge(withFile:)`

`Instance Method`

Returns a new `QEDFile` instance by merging the current instance with the provided `file`.

**Declaration**

```swift
func merge(withFile file : QEDFile) -> QEDFile
```

**Return Value**

Returns a new `QEDFile` instance.
All the properties of the `file` instance that are non `nil` will prevail over the values for the attributes of the current instance, the [`proofStatus`](#proofstatus) is also merged

___

### `<(_:_:)`

`Operator`

Returns a Boolean value indicating whether the value of the first argument is less than that of the second argument.

**Declaration**

```swift
static func < (lhs: QEDFile, rhs: QEDFile) -> Bool
```

| Parameter   | Description |
| ---------   | ----------- |
| `lhs`       | An instance to compare |
| `rhs`       | An instance to compare |

**Discussion**

`QEDFile` conforms to the `Comparable` protocol.

`<` is purely based on the same comparison executed on the [`dateTaken`](#datetaken) attribute of the instances.

___

### `==(_:_:)`

`Operator`

Returns a Boolean value indicating whether two values are equal.

**Declaration**

```swift
static func == (lhs: QEDFile, rhs: QEDFile) -> Bool
```

| Parameter   | Description |
| ---------   | ----------- |
| `lhs`       | An instance to compare |
| `rhs`       | An instance to compare |

**Discussion**

`QEDFile` conforms to the `Equatable` protocol.

Two `QEDFile` instances are considered equal if they have the same [`slug`](#slug) (i.e. The represent the same report on the backend).

If one or both files are missing the [`slug`](#slug), they are considered equal if they have the same [`dateTaken`](#datetaken). (i.e. The instance before reporting and the instance created from report response are considered equal)
