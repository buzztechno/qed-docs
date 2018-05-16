# QEDSDK Documentation for IOS

The IOS _QEDSDK_ provides a set of convenience instance types aimed to ease and shorten the development time for the usage of the [_QED REST API_](../README.md).

The main type is the [_QEDAPI_ class](#qedapi-class-reference), which is a wrapper over the functionality of the [_QED REST API_](../README.md).
The [_QEDFile_](#qedfile-type-reference) defines the representation of a data file to be reported to and notarized by the [_QED REST API_](../README.md). The [_QEDAPI_ class](#qedapi-class-reference) class receives and returns "file report" representations using the [_QEDFile_ type](#qedfile-type-reference).

Two more convenience types are provided which are optional to use:
- The [_QEDModel_](#qedmodel-type-reference) provides a set of convenience methods to aid on persisting and retrieving a data model to/from local storage.
- The [_QEDVideoPlayRecordViewController_](#qedvideoplayrecordviewcontroller-class-reference) provides in a simple interface, the necessary functionality to play and record videos. You can use this view controller out of the box if your application is meant for video capturing for notarization.

# Contents

1. [Getting Started](#getting-started)
1. [Type Reference](#types)
    - [_QEDAPI_](#qedapi-class-reference)
    - [_QEDFile_](#qedfile-type-reference)
    - [_QEDModel_](#qedmodel-type-reference)
    - [_QEDVideoPlayRecordViewController_](#qedvideoplayrecordviewcontroller-class-reference)
1. [Protocols Reference](#protocols)
    - [_QEDVideoPlayRecordViewControllerDelegate_](#qedvideoplayrecordviewcontrollerdelegate-protocol-reference)

# Getting Started

## Add the SDK to your project

### Cocoa Pods
1. To integrate _QEDSDK_ for iOS into your Xcode project using CocoaPods, specify it in your `Podfile`:

    ```ruby
    source 'https://github.com/CocoaPods/Specs.git'
    platform :ios, '10.0'
    use_frameworks!

    target 'YOUR_APPLICATION_TARGET_NAME_HERE' do
        pod 'QEDSDK', '1.0.0'
    end
    ```
2. Then, run the following command:
    ```shell
    $ pod install
    ```

### Manually
1. Drag **QEDSDK.framework** to the **Embedded Binaries** section in the _General_ tap of your project's main target. Check _Copy items if needed_ and choose to _Create groups_.
    ![Drag to Embedd](images/IOS_Image01.png)

1. Add a new Run Script Phase in your target’s Build Phases.

    **IMPORTANT**: Make sure this Run Script Phase is below the Embed Frameworks build phase. You can drag and drop build phases to rearrange them.

    Paste the following line in this Run Script Phase's script text field:
    ```shell
    bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/QEDSDK.framework/ios-strip-frameworks.sh" QEDSDK
    ```

    ![Build Phases](images/IOS_Image02.png)

3. (Ignore if your project is a Swift only project) - Set the **Always Embed Swift Standard Libraries** setting in your targets _Build Settings_ to **YES**

    ![Allways Embed Swift](images/IOS_Image03.png)

## Get your API Key
In order to use the _QEDSDK_, you must obtain an API Key. Please get in contact: mail@qed.digital.

## Quick Launch
1. Open your _AppDelegate_ file and import the _QEDSDK_.
    ```swift
    import QEDSDK
    ```
1. Copy the lines below and paste them into your AppDelegate’s `application(_:didFinishLaunchingWithOptions:)` method
    ```swift
    QEDAPI.setup(with: "YOUR_API_KEY")
    ```
1. Prepare a [_QEDFile_](#qedfile-type-reference) representing a local data file to be reported and notarized

    - Create a [_QEDFile_](#qedfile-type-reference) instance. You can instantiate from the local URL for an existing file.
        ```swift
        var file = QEDFile(with: localFileURL)!
        ```
    - Calculate the _SHA256_ hash of the content file if not done yet.
        ```swift
        file = file.withCalculatedHash()
        ```
1. Use the [_QEDAPI_ class](#qedapi-class-reference) to interface with the [_QED REST API_](../README.md) server

    - Issue a report to the backend for notarization
        ```swift
        QEDAPI.sendReport(for: file) { (updatedFile, error) in
            // Make sure to perform UI update related actions on main thread
            DispatchQueue.main.async {
                guard error == nil else {
                    // Handle reporting errors. The `error` instance will
                    // include detailed information.
                    return
                }
                // Handle success. Update your data source and UI accordingly.
                // `updatedFile` contains a copy of `file` with an updated `proofStatus`
            }
        }
        ```
    - You can check periodically for status updates until report is `confirmed`
        ```swift
        QEDAPI.getReport(for: file) { (updatedFile, error) in
            // Make sure to perform UI update related actions on main thread
            DispatchQueue.main.async {
                guard error == nil else {
                    // Handle reporting errors. The `error` instance will
                    // include detailed information.
                    return
                }
                // Handle success. Update your data source and UI accordingly.
                // `updatedFile` contains a copy of `file` with an updated
                // `proofStatus`
            }
        }
        ```
    - Once the [_QEDFile_](#qedfile-type-reference) status is `confirmed`, you can request a PDF certificate from backend
        ```swift
        QEDAPI.generatePDF(for: file) { (success, error) in
            // Make sure to perform UI update related actions on main thread
            DispatchQueue.main.async {
                guard success, error == nil else {
                    // Handle pdf generation errors. The `error` instance will
                    // include detailed information.
                    return
                }
                // Handle success. Update your data source and UI accordingly.
                // The PDF proof document is now stored in the `pdfURL` of
                // the requested `file`
            }
        }
        ```

# Types

## _QEDAPI_ Class Reference

`Class`

The _QEDAPI_ provides a wrapper class on the [_QED REST API_](../README.md).

### Setup

```swift
class func setup(with apiKey : String)
```
Will setup the _QEDAPI_ shared instance with the required `apiKey` to make authenticated requests.

| Parameter | Description |
| --------- | ----------- |
| `apiKey`  | The key to make authenticated requests to the [_QED REST API_](../README.md) |

**Discussion**

If you use the methods on this class without calling this setup method first, the requests will sill be made but will fail on the server due to Authentication restrictions.

### Issuing API Requests

 ```swift
 class func sendReport(for file : QEDFile, completion : (QEDFile?, Error?)->Void)
 ```

 Will issue a request to the backend (for initial submission or updates if previously reported) for a notarizing report of the given `file`.

 | Parameter  | Description |
 | ---------  | ----------- |
 | `file`     | The [_QEDFile_](#qedfile-type-reference) representing the file content to be notarized|
 |`completion`| The completion handler to call when the request is complete. This handler is not called on the main thread |

 The `completion` handler takes the following parameters:

 | Parameter        | Description |
 | ---------        | ----------- |
 | `updatedFile`    | `nil` if request failed, otherwise a [_QEDFile_](#qedfile-type-reference) instance created as a copy of `file` with an updated `proofStatus` based on API response data|
 | `error`          | An error object that indicates why the request failed, or `nil` if the request was successful|

 **Discussion**

 After posting a report for a `file` for the first time, a blockchain proof is initiated.

 The server will respond with the updated report including blockchain transaction details. In most cases the report status will be `pending` when posting for the first time.

  ___
  ```swift
  class func getReport(for file : QEDFile, completion : (QEDFile?, Error?)->Void)
  ```

  Will get the current report data for a given `file` from the backend.

  | Parameter | Description |
  | --------- | ----------- |
  | `file`    | The [_QEDFile_](#qedfile-type-reference) representing the file content report to be updated from backend|
  |`completion`| The completion handler to call when the request is complete. This handler is not called on the main thread |

   The `completion` handler takes the following parameters:

   | Parameter        | Description |
   | ---------        | ----------- |
   | `updatedFile`    | `nil` if request failed, otherwise a [_QEDFile_](#qedfile-type-reference) instance created as a copy of `file` updated based on API response data|
   | `error`          | An error object that indicates why the request failed, or `nil` if the request was successful|
  **Discussion**

  After initial submission of a report, you use this method to update the [_QEDFile_](#qedfile-type-reference) `file` properties from backend. Specially the `proofStatus`

### Constants representing API attributes


## _QEDFile_ Type Reference
`Struct`

A _QEDFile_ represents a data file locally stored on the user device that needs to be time stamped and notarized on the blockchain

### Static properties

```swift
static var qedDirectory : URL
```
Default storage directory for the `QEDFile` content file in the app's bundle.

**Discussion**

You can use this directory to store the files you want to represent as `QEDFile`'s for notarization.
If you use this default directory, then you can create `QEDFile` instances directly with `name`, `ext` and `dateCreated` values for the file, and the local `fileURL` location will be assumed to be on the root level of this directory.

 ___

 ```swift
 static var pdfDirectory : URL
 ```
  Storage dictionary for the downloaded PDF files in the app bundle.

 **Discussion**

 When downloading PDF certificates for a `QEDFile` using the [_QEDAPI_ class](#qedapi-class-reference), they will be automatically stored on the root level of this local directory.
 You can get the exact PDF url by accessing the `pdfURL` property of the `QEDFile`.

### Initializers

```swift
init?(with fileURL : URL)
```
Creates a `QEDFile` instance for the given local `fileURL`.

| Parameter | Description |
| --------- | ----------- |
| `fileURL` | The local URL for the file to reference in the `QEDFile` instance. |

**Discussion**

The `name`, `ext` and `dateTaken` will be captured from the file at `fileURL`. If the content file at `fileURL` does not exist, the initialization fails and returns `nil`

___
```swift
init(withName name : String, extension ext : String, dateTaken: Date)
```
Creates a `QEDFile` instance with a given `name`, extension(`ext`) and `dateTaken`.

| Parameter   | Description |
| ---------   | ----------- |
| `name`      | A string representing the name of the local file without extension |
| `ext`       | A string representing the extension of the local file |
| `dateTaken` | A `Date` instance representing the date the file was created |

**Discussion**

The `fileURL` will be infered from the default file storage `qedDirectory`, the `name` and the extension(`ext`)

The `dateTaken` is just used as metadata and will be different that the timestamp on the blockchain certificate.

___
```swift
init?(fromQEDAPIResponseDict dict : [String:Any], fileURL : URL? = nil)
```
Creates a `QEDFile` instance from a Dictionary from API response.

| Parameter   | Description |
| ---------   | ----------- |
| `dict`      | A dictionary representation of a _QEDAPI_ qualified response with report data |
| `fileURL`   | An optional URL indicating the local storage location of the associated file.|

**Discussion**

For a list of valid _QEDAPI_ report response attribues see [here](#constants-representing-api-attributes).

`dict` must include `name`, `slug`, `hash`, `dateTaken` string and `status` as a minimum otherwise the instantiation will fail and return `nil`.

The `fileURL` will be infered from the default file storage `qedDirectory` and the calculated instance `name` and extension(`ext`). If you manage local file storage on a separate location, you must pass the `fileURL` indicating the local URL for the `QEDFile` content file if existing. You can also update the `fileURL` attribute at a later time.

## _QEDModel_ Type Reference

## _QEDVideoPlayRecordViewController_ Class Reference

# Protocols

## _QEDVideoPlayRecordViewControllerDelegate_ Protocol Reference
