<!-- Source: https://developers.zoom.us/docs/video-sdk/web/whiteboard.md -->
<!-- Fetched: 2026-06-15 -->

# Whiteboard


Zoom's whiteboard is a collaborative, cloud-based infinite canvas that developers can embed into their applications to enable real-time and asynchronous visual collaboration.

## Check platform support

Before using whiteboard features, verify that the interactive whiteboard is enabled for the current account and supported on the current platform.

```javascript
const isEnabled = whiteboardClient.isWhiteboardEnabled();
```

Interactive whiteboard **is not supported** on low-performance devices (CPU cores ≤ 2), mobile platforms, and older SDK versions (< 2.3.5). Whiteboard content is provided as a **compatible sharing stream** instead of an interactive canvas on unsupported platforms.

## Get the whiteboard instance

After joining a session, obtain the whiteboard client instance.

```javascript
const whiteboardClient = client.getWhiteboardClient();
```

## Start and stop whiteboard sessions

Currently, only the **session host or manager** can start a whiteboard as the presenter. Other users can join as viewers to see and interact with the whiteboard content.

### As a Presenter

Presenters must be a session host or manager.

#### Start a new whiteboard session

To start a new whiteboard session, provide an HTML element as the mounting point.

```javascript
await whiteboardClient.startWhiteboardScreen(whiteboardElement);
```

-   The element should be an `HTMLDivElement` (or similar block element).
-   The element should be empty; any existing child elements will be replaced.
-   A CSS class `whiteboard-sdk-css` will be added as a namespace to scope whiteboard styles. Avoid using the same class name in your application to prevent CSS conflicts.

#### Stop a whiteboard session

As a presenter, you can stop your own whiteboard session.

```javascript
await whiteboardClient.stopWhiteboardScreen();
```

**Note:** The `stopWhiteboardScreen()` method only supports presenters stopping their own whiteboard session.

### As a Viewer

Non-presenters can view the whiteboard by **listening to whiteboard state changes**.

#### Listen to whiteboard state changes

```javascript
client.on("peer-whiteboard-state-change", async (payload) => {
    const { action, userId } = payload;

    if (action === "Start") {
        // Another user started presenting a whiteboard
        await whiteboardClient.startWhiteboardView(whiteboardElement, userId);
    } else if (action === "Stop") {
        // Whiteboard presentation has ended
        await whiteboardClient.stopWhiteboardView();
    }
});
```

#### Handle late joins

If someone joins the session after a whiteboard has already been started, they may miss the `peer-whiteboard-state-change` event. In this case, check for an active whiteboard session and render it.

```javascript
const presenter = whiteboardClient.getWhiteboardPresenter();
if (presenter) {
    await whiteboardClient.startWhiteboardView(
        whiteboardElement,
        presenter.userId,
    );
}
```

#### Monitor whiteboard loading

It may take some time to initialize a whiteboard. Monitor the loading status using the `whiteboard-status-change` event.

```javascript
client.on("whiteboard-status-change", (status) => {
    if (status === WhiteboardStatus.Pending) {
        // Show whiteboard area with a loading indicator
        showLoadingIndicator();
    } else if (status === WhiteboardStatus.InProgress) {
        // Hide loading indicator - whiteboard is ready
        hideLoadingIndicator();
    } else if (status === WhiteboardStatus.Closed) {
        // Hide whiteboard area
        hideWhiteboardArea();
    }
});
```

## Manage data and export whiteboard content

In-session whiteboards **are not persisted by default**. Once you call `stopWhiteboardScreen()`, all whiteboard data is lost. Data can be lost when someone stops sharing the whiteboard, a user leaves a session, when a user joins a sub-session, or when the app crashes or has network connectivity issues.

### Export as PDF

If you need to preserve the whiteboard content, export it before stopping the session.

```javascript
// Export as PDF with a custom filename
await whiteboardClient.exportWhiteboard("pdf", "brainstorming-session");
```

The file will be downloaded through the browser's download mechanism.

## Complete whiteboard integration example

```javascript
const client = ZoomVideo.createClient();
let whiteboardClient;
client.init("en-us", "Global");

// Initialize after joining session
await client.join("session name", "jwt token", "name");
whiteboardClient = client.getWhiteboardClient();

// Check if whiteboard is available
if (!whiteboardClient.isWhiteboardEnabled()) {
    console.warn("Whiteboard not available on this platform");
    return;
}

// Set up event listeners
client.on("peer-whiteboard-state-change", async (payload) => {
    const { action, userId } = payload;

    if (action === "Start") {
        await whiteboardClient.startWhiteboardView(whiteboardElement, userId);
    } else if (action === "Stop") {
        await whiteboardClient.stopWhiteboardView();
    }
});

client.on("whiteboard-status-change", (status) => {
    updateUIBasedOnStatus(status);
});

// Start whiteboard as presenter (host/manager only)
if (whiteboardClient.canStartWhiteboard()) {
    try {
        await whiteboardClient.startWhiteboardScreen(whiteboardElement);
        console.log("Whiteboard started successfully");
    } catch (error) {
        console.error("Failed to start whiteboard:", error);
    }
}

// Export before stopping
async function stopWithExport() {
    await whiteboardClient.exportWhiteboard("pdf", "session-whiteboard");
    await whiteboardClient.stopWhiteboardScreen();
}
```
