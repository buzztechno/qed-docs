# `QEDVideoPlayRecordViewController` Class Reference

`Class`

A _ViewController_ providing record and playback video functionality for locally stored video files.

It will switch automatically between video playback and video recording modes depending on the internal value for the local stored video file.

If a local video file is assigned, the controller will be in playback mode, otherwise in recording mode.

If allowed by the [`delegate`](QEDVideoPlayRecordViewControllerDelegate.md), a user can trigger the deletion of the local stored video file. After deletion the controller will switch to recording mode automatically.

## Initialisation

### `init?(with:)`

`Initialiser`

Creates a `QEDVideoPlayRecordViewController` instance for presentation

**Declaration**

```swift
init(delegate : QEDVideoPlayRecordViewControllerDelegate, directory : URL, fileURL : URL? = nil, autoplay : Bool = false)
```

| Parameter  | Description |
| ---------  | ----------- |
| `delegate` | A `class` instance conforming to the [`QEDVideoPlayRecordViewControllerDelegate`](QEDVideoPlayRecordViewControllerDelegate.md) protocol. The delegate is not retained |
| `directory` | A local file directory where to store the newly recorded video files |
| `fileURL`   | An optional URL for a local video file. Defaults to `nil`|
| `autoplay`  | An indication if the video should autoplay when initialised with an existing local video URL. Defaults to `false`|

**Discussion**

The instance provide both recording and playback of video content. It will switch automatically between video playback and video recording modes depending on the internal value of `fileURL`.

When initialised with a valid `fileURL`, the controller will start on playback mode when presented and will play automatically if `autoplay == true`.

If `fileURL == nil` on initialisation, the controller will start on recording mode when presented.
