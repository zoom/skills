<!-- Source: https://developers.zoom.us/docs/video-sdk/web/share-annotation.md -->
<!-- Fetched: 2026-06-15 -->

# Annotation


Annotation enables presenters and users to draw, highlight, and interact directly with shared screen content.

## Requirements

-   A relative-positioned HTML container (``) surrounding your shared content (`<canvas>`or`<video>`).
-   No padding or borders added to the `<canvas>` or `<video>` elements.
-   Screen sharing features enabled. See [screen sharing core features](/docs/video-sdk/web/share/) and the [Stream namespace](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html) for details.
-   Video SDK for web version 2.2.0 is required to use annotation features, such as draw on screen. Users with prior versions will only be able to see annotation actions.

## Set up annotation

Set up the HTML file, for example, using this sample code.

```html
<div id="share-container" style="position: relative;">
    <canvas id="receiving-share-content"></canvas>
</div>
```

Follow these guidelines to add dynamic `<canvas>` sizing. Use a similar implementation for `<video>` sizing.

-   The shared content should take the 100% space of the `<canvas>`.
-   The `<canvas>` aspect ratio is always equal to its displayed content's aspect ratio.

See the following code for an example.

```javascript
client.on("share-content-dimension-change", ({ type, height, width }) => {
    const canvas = document.getElementById("receiving-share-content");
    const container = document.getElementById("share-container");

    const containerWidth = container.clientWidth;
    const containerHeight = container.clientHeight;
    const contentAspectRatio = width / height;
    const containerAspectRatio = containerWidth / containerHeight;

    if (containerAspectRatio > contentAspectRatio) {
        // Container is wider than content — fit by height
        canvas.style.width = `${containerHeight * contentAspectRatio}px`;
        canvas.style.height = `${containerHeight}px`;
    } else {
        // Container is taller than content — fit by width
        canvas.style.width = `${containerWidth}px`;
        canvas.style.height = `${containerWidth / contentAspectRatio}px`;
    }
});
```

## Method details

-   `startAnnotation()` - Starts annotation mode on shared content. Note: Use this method again if the previous share stopped and you need to annotate on a new share session.
-   `stopAnnotation()` - Stops annotation. Also use for the following use cases.

    -   Current screen share is paused for presenter and viewer.

        ```javascript
        // For viewer, listen to the 'user-update' event
        const activeShareUserId = client.getMediaStream().getActiveShareUserId()；
        const activeShareUser = client.getAllUser().find((user) ⇒ user.userId === activeShareUserId));
        const isSharePaused = activeShareUser?.sharerOn && activeShareUser?.sharerPause;
        ```

    -   For viewer, Current screen share session annotation is disabled by presenter via `changeAnnotationPrivilege()`. See `annotation-privilege-change` for details.

-   `changeAnnotationPrivilege(enable)` - Enable or disable the viewers's annotation permissions.
-   `canDoAnnotation()` - Checks if the current user can annotate after sending or receiving share.
-   `getAnnotationController()` - Retrieves the annotation controller object.

## `AnnotationController` methods

-   `setToolType(toolType: AnnotationToolType)` - Sets the [annotation tool type](https://marketplacefront.zoom.us/sdk/custom/web/enums/ZoomVideo.AnnotationToolType.html), like pen, eraser, vanishing pen, and others (default: None = 0).
-   `setToolColor(color: number)` - Sets the annotation color in RGBA format (HEX number). Default is black `0xff000000`.
-   `setToolWidth(width: number)` - Sets the annotation width. Default is 8px.

    ```javascript
    // Step 1: Get the annotation controller
    const annotationController = client
        .getMediaStream()
        .getAnnotationController();

    // Step 2: Start annotation
    await client.getMediaStream().startAnnotation();

    // Step 3: Set up a pen tool
    await annotationController.setToolType(AnnotationToolType.Pen);
    await annotationController.setToolWidth(8);
    ```

-   `getToolType()` - Gets the current annotation tool.
-   `getToolColor()` - Gets the current annotation color.
-   `getToolWidth()` - Gets the current annotation width.
-   `redo()` - Redoes the last undone annotation.
-   `undo()` - Undoes the last annotation.
-   `clear(clearType - AnnotationClearType)` - Clears annotations based on specified type (mine, viewer, or all).
    -   Viewer: Can clear `Mine` = 0.
    -   Host: Can clear `Mine` = 0, `All` = 2.
    -   Presenter (Active Sharer) - Can clear `Mine` = 0, `Viewer` = 1, and `All` = 2.

## Handling active sharer events

Presenter responding to viewer annotation requests.

```javascript
client.on("annotation-viewer-draw-request", ({ userId }) => {
    // If the annotation is not started, the presenter should use the startAnnotation() to enable viewer to draw on the sharing content
});
```

Monitoring undo and redo availability.

```javascript
client.on("annotation-undo-status", ({ status, presenterId }) => {
    // Update UI elements based on undo status
});

client.on("annotation-redo-status", ({ status, presenterId }) => {
    // Update UI elements based on redo status
});
```

Check annotation privilege.

```javascript
client.on("annotation-privilege-change", ({ userId, isAnnotationEnabled }) => {
    // Disable the annotation feature on the viewer side
    // If the annotation is ongoing && isAnnotationEnabled === false, use the stopAnnotation() method to stop the ongoing annotation
});
```

## Sample workflow

1. Prepare a **relative positioned** container and content (`<canvas>` or `<video>`).
2. Listen to the related **screen share events** and **annotation events**.
3. Render the shared content. (`startScreenShare()` or `startShareView()`)
4. Start annotation mode on demand (`startAnnotation()`).
5. Use annotation tool methods (`getAnnotationController()`).
