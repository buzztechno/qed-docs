# `QEDModel` Type Reference

`Struct`

`QEDModel` provides a set of helper methods to aid on persisting a data model ( represented as an array of [`QEDFile`](QEDFile.md) instances).

Usage of `QEDModel` is optional and is meant just as an aid to speed up your data source implementation.

## Persisting and Retrieving from Local Storage

### `persist(filesArray:)`

`static Method`

Will persist a given array of [`QEDFile`](QEDFile.md) instances to _User Defaults_.

**Declaration**

```swift
static func persist(filesArray files : [QEDFile])
```

| Parameter | Description |
| --------- | ----------- |
| `files`   | An array of [`QEDFile`](QEDFile.md) instances representing your data model to be persisted locally to _User Defaults_ |

**Discussion**

Normally you will call this method when the app goes to background, passing your in memory model array for persisting.
___

### `loadFromStore()`

`static Method`

Returns the locally persisted array of [`QEDFile`](QEDFile.md) instances.

**Declaration**

```swift
static func loadFromStore() -> [QEDFile]
```

**Return Value**

Returns the currently persisted array of [`QEDFile`](QEDFile.md) instances from _User Defaults_

If no array is persisted currently in  _User Defaults_, an empty array will be returned.
