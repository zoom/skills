<!-- Source: https://developers.zoom.us/docs/video-sdk/web/video-camera-controls.md -->
<!-- Fetched: 2026-06-15 -->

# Camera controls


Add controls for [Pan, Tilt, Zoom (PTZ)](#pan-tilt-zoom-ptz) cameras and [second cameras](#share-a-second-camera).

## Pan, Tilt, Zoom (PTZ)

The Video SDK can control local and far end cameras that are compatible with the PTZ standard. You can use these functions to build a remote monitoring solution with camera control that allows the user to pan-tilt-zoom the device.

> This is currently only supported on Chromium-based browsers on Windows. To determine whether your browser supports PTZ for a camera, go to the internal `about://media-internals` page and see the "Pan-Tilt-Zoom" column in the "Video Capture" tab.

### Control local camera

To get the capability of the default camera on the device, call [`getCameraPTZCapability`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#getCameraPTZCapability).

```javascript
stream.getCameraPTZCapability();
```

If the device has multiple cameras attached, pass in a [device ID](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.default.html#getDevices) as the `cameraId` to [`getCameraPTZCapability`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#getCameraPTZCapability) for the specific camera's capabilities.

```javascript
let cameras = stream.getCameraList();

let cameraId = cameras[1].deviceId;

stream.getCameraPTZCapability(cameraId);
```

To control the local camera, call [`controlCamera`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#controlCamera), and pass in the `cmd` and `range` parameters.

```javascript
stream.controlCamera({
    cmd: CameraControlCmd.Left,
    range: 10,
});
```

### Control far end camera

To get control of a specific user's far end camera, the user will need to request control from the specific user and approve the request.

```javascript
//User A on request control of User B's camera
stream.requestFarEndCameraControl(userBId);

// User B on receiving request for camera control
client.on("far-end-camera-request-control", (payload) => {
    const { userId, displayName } = payload;

    // popup a confirm window, approve or decline the request
    stream.approveFarEndCameraControl(userId);
});

// User A on receiving control's request from User B
let isFarEndCameraApproved = false;
let capability = undefined;
client.on("far-end-camera-response-control", (payload) => {
    const { isApproved, userId, displayName } = payload;
    isFarEndCameraApproved = isApproved;
});
```

To get the capability of the far end camera, pass in the `userId` to [`getCameraPTZCapability`](https://marketplacefront.zoom.us/sdk/custom/web/modules/Stream.html#getCameraPTZCapability). The function will return the far end camera capabilities if the user has already approved the request for far end camera control.

```javascript
stream.getFarEndCameraPTZCapabilitiy(userId);
```

To control the far end camera, call [`controlFarEndCamera`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#controlFarEndCamera), and pass in the `userId`, `cmd`, and `range` parameters.

```javascript
stream.controlFarEndCamera({
    userId,
    cmd: CameraControlCmd.Left,
    range: 10,
});
```

To give up control of the camera, call [`giveUpFarEndCameraControl`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#giveUpFarEndCameraControl), and pass in the `userId` parameter.

```javascript
stream.giveUpFarEndCameraControl(userId);
```

## Share second camera content

You can share content from a second camera, such as an external USB camera (for example, a document camera or ultrasound camera) using the `stream.startShareScreen()` function. Once the user grants camera permission, get the list of cameras and pass in a `deviceId` share.

```javascript
let cameras = stream.getCameraList();

if (stream.isStartShareScreenWithVideoElement()) {
    // if MediaStreamTrackProcessor is supported
    stream.startShareScreen(document.querySelector("#my-second-camera-video"), {
        secondaryCameraId: cameras[1].deviceId,
    });
} else {
    // if MediaStreamTrackProcessor is not supported
    stream.startShareScreen(
        document.querySelector("#my-second-camera-canvas"),
        {
            secondaryCameraId: cameras[1].deviceId,
        },
    );
}
```
