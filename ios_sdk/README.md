# QEDSDK Documentation for IOS

The IOS _QEDSDK_ provides a set of convenience instance types aimed to ease and shorten the development time for the usage of the [_QED REST API_](../README.md).

The main type is the [`QEDAPI`](reference/QEDAPI.md) class, which is a wrapper over the functionality of the [_QED REST API_](../README.md).
The [`QEDFile`](reference/QEDFile.md) defines the representation of a data file to be reported to and notarized by the [_QED REST API_](../README.md). The [`QEDAPI`](reference/QEDAPI.md) class receives and returns "file report" representations using the [`QEDFile`](reference/QEDFile.md) struct.

Two more convenience types are provided which are optional to use:
- The [`QEDModel`](reference/QEDModel.md) provides a set of convenience methods to aid on persisting and retrieving a data model to/from local storage.
- The [`QEDVideoPlayRecordViewController`](reference/QEDVideoPlayRecordViewController.md) provides in a simple interface, the necessary functionality to play and record videos. You can use this view controller out of the box if your application is meant for video capturing for notarization.

# Contents

- [Getting Started](#getting-started)
- Type Reference
    - [`QEDAPI`](reference/QEDAPI.md)
    - [`QEDFile`](reference/QEDFile.md)
    - [`QEDModel`](reference/QEDModel.md)
    - [`QEDVideoPlayRecordViewController`](reference/QEDVideoPlayRecordViewController.md)
- Protocols Reference
    - [`QEDVideoPlayRecordViewControllerDelegate`](reference/QEDVideoPlayRecordViewControllerDelegate.md)

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
1. Prepare a [`QEDFile`](reference/QEDFile.md) representing a local data file to be reported and notarized

    - Create a [`QEDFile`](reference/QEDFile.md) instance. You can instantiate from the local URL for an existing file.
        ```swift
        var file = QEDFile(with: localFileURL)!
        ```
    - Calculate the _SHA256_ hash of the content file if not done yet.
        ```swift
        file = file.withCalculatedHash()
        ```
1. Use the [`QEDAPI`](reference/QEDAPI.md) class methods to interact with the [_QED REST API_](../README.md) server

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
    - Once the [`QEDFile`](reference/QEDFile.md) status is `confirmed`, you can request a PDF certificate from backend
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
