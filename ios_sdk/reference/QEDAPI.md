# `QEDAPI` Class Reference

`Class`

The `QEDAPI` provides a wrapper class on the [_QED REST API_](../../README.md).

## Setup

### `setup(with:)`

`Class Method`

Will setup the `QEDAPI` shared instance with the required `apiKey` to make authenticated requests.

**Declaration**

```swift
class func setup(with apiKey : String)
```

| Parameter | Description |
| --------- | ----------- |
| `apiKey`  | The key to make authenticated requests to the [_QED REST API_](../../README.md) |

**Discussion**

If you use the methods on this class without calling this setup method first, the requests will sill be made but will fail on the server due to Authentication restrictions.

## Issuing API Requests

### `sendReport(for:completion:)`

`Class Method`

Will issue a request to the backend (for initial submission or updates if previously reported) for a notarizing report of the given `file`.

**Declaration**

```swift
class func sendReport(for file : QEDFile, completion : (QEDFile?, Error?)->Void)
```

| Parameter  | Description |
| ---------  | ----------- |
| `file`     | The [`QEDFile`](QEDFile.md) representing the file content to be notarized|
|`completion`| The completion handler to call when the request is complete. This handler is not called on the main thread |

The `completion` handler takes the following parameters:

| Parameter        | Description |
| ---------        | ----------- |
| `updatedFile`    | `nil` if request failed, otherwise a [`QEDFile`](QEDFile.md) instance created as a copy of `file` with an updated `proofStatus` based on API response data|
| `error`          | An error object that indicates why the request failed, or `nil` if the request was successful|

**Discussion**

After posting a report for a `file` for the first time, a blockchain proof is initiated.

The server will respond with the updated report including blockchain transaction details. In most cases the report status will be `pending` when posting for the first time.

___

### `getReport(for:completion:)`

`Class Method`

Will get the current report data for a given `file` from the backend.

**Declaration**

```swift
class func getReport(for file : QEDFile, completion : (QEDFile?, Error?)->Void)
```

| Parameter | Description |
| --------- | ----------- |
| `file`    | The [`QEDFile`](QEDFile.md) representing the file content report to be updated from backend|
|`completion`| The completion handler to call when the request is complete. This handler is not called on the main thread |

The `completion` handler takes the following parameters:

| Parameter        | Description |
| ---------        | ----------- |
| `updatedFile`    | `nil` if request failed, otherwise a [`QEDFile`](QEDFile.md) instance created as a copy of `file` updated based on API response data|
| `error`          | An error object that indicates why the request failed, or `nil` if the request was successful|

**Discussion**

After initial submission of a report, you use this method to update the [`QEDFile`](QEDFile.md) `file` properties from backend. Specially the `proofStatus`

## Constants representing API attributes
