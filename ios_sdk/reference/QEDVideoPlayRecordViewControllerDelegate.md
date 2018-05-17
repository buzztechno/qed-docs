# `QEDVideoPlayRecordViewControllerDelegate` Protocol Reference

`Protocol`

A class conforming to this protocol will receive the necessary method calls to handle the lifecycle for the video playback, recording and deletion from a [`QEDVideoPlayRecordViewController`](reference/QEDVideoPlayRecordViewController.md).

## Managing File Deletion

### `videoPlayRecordController(_:allowToDeleteFileAt:)`

`Instance Method`

Asks the delegate if a local video file can be deleted.

**Declaration**

```swift
func videoPlayRecordController(_ controller: QEDVideoPlayRecordViewController, allowToDeleteFileAt fileURL : URL) -> Bool
```

| Parameter    | Description |
| ---------    | ----------- |
| `controller` | The controller object informing the delegate of this event |
| `fileURL`    | The local URL for the file the request is made for |

**Return Value**

`true` if the file at `fileURL` could be deleted by the user, otherwise `false`.

**Discussion**

Returning `true` does not trigger file deletion, but a delete button will be made available to the user. File deletion can then be triggered by the user by tapping on the delete button.
___

### `videoPlayRecordController(_:fileDeletionRequestedFor:)`

`Instance Method`

Tells the delegate a file deletion has been requested for a local file.

**Declaration**

```swift
func videoPlayRecordController(_ controller: QEDVideoPlayRecordViewController, fileDeletionRequestedFor fileURL : URL)
```

| Parameter    | Description |
| ---------    | ----------- |
| `controller` | The controller object informing the delegate of this event |
| `fileURL`    | The local URL for the file that has been requested for deletion |

**Discussion**

It is up to your implementation to actually delete the file from the file system.

After calling this method the controller will clear its reference to the reported `fileURL`

## Managing Video Recording

### `videoPlayRecordController(_:userRecordedFileAt:)`

`Instance Method`

Tells the delegate the user has finished recording a new video.

**Declaration**

```swift
func videoPlayRecordController(_ controller: QEDVideoPlayRecordViewController, userRecordedFileAt fileURL : URL)
```

| Parameter    | Description |
| ---------    | ----------- |
| `controller` | The controller object informing the delegate of this event |
| `fileURL`    | The local URL for the new video file recorded by the user |

**Discussion**

When this method is called, the user has ended recording a video file which is now available at `fileURL`. You must not delete this file as the controller has now switched to playback mode and expects the existence of the video file at `fileURL`

You can choose to act on this request and trigger your logic for new videos, but be aware that the user can still trigger file deletion for the new video. It is recommended instead to handle new video logic once the user has requested to dismiss the controller. For more info on this, see [`videoPlayRecordController(_:dismissalRequestWithNewRecordedFileAt:keepingFile)`](#videoplayrecordcontrollerdismissalrequestwithnewrecordedfileatkeepingfile:))

## Managing Controller Dismissal Requests

### `videoPlayRecordController(_:dismissalRequestWithNewRecordedFileAt:keepingFile:)`

`Instance Method`

Tells the delegate the user has requested dismissal of the controller.

**Declaration**

```swift
func videoPlayRecordController(_ controller: QEDVideoPlayRecordViewController, dismissalRequestWithNewRecordedFileAt fileURL : URL?, keepingFile keep : Bool)
```

| Parameter    | Description |
| ---------    | ----------- |
| `controller` | The controller object informing the delegate of this event |
| `fileURL`    | The local URL for for a new recorded video. |
| `keep`       | A Boolean indicating if the file at `fileURL` should be kept |

**Discussion**

It is the `delegate` responsibility to actually dismiss the controller.

`fileURL` when not `nil` is the local storage URL for a new recorded video.

If `keep` is `true` the user has indicated intention to keep the file by tapping on the confirm button. If `false` user has indicated to just dismiss the controller with no intention of keeping the file.

It is up to the `delegate` to delete the file at `fileURL` from the file system when `fileURL` is not `nil` and `keep == false`

___

### `videoPlayRecordController(_:dismissalRequestDueTo:)`

`Instance Method`

Tells the delegate the controller needs to be dismissed due to a setup error.

**Declaration**

```swift
func videoPlayRecordController(_ controller: QEDVideoPlayRecordViewController, dismissalRequestDueTo error : QEDVideoPlayRecordViewController.SetupError)
```

| Parameter    | Description |
| ---------    | ----------- |
| `controller` | The controller object informing the delegate of this event |
| `error`      | An instance of `QEDVideoPlayRecordViewController.SetupError` indicating the reason for controller setup failure |

**Discussion**

It is the `delegate` responsibility to actually dismiss the controller.

`SetupError` enum possible values are:
- `userNotAllowedToUseCamera`: User has denied usage of the camera to the app. Need to be changed in Settings
- `errorSettingUpForRecording`: A setup error due to HW availability or recording session setup.
