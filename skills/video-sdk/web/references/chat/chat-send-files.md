<!-- Source: https://developers.zoom.us/docs/video-sdk/web/chat-send-files.md -->
<!-- Fetched: 2026-06-15 -->

# Send files


Users can transfer files in chat during the session.

## Prerequisites

-   Enable [send files via in-session chat](/docs/build/account/#in-session-advanced) in your Video SDK account portal settings and configure the following settings.
    -   **Only allow specified file types** - Select to allow only specific file types and enter the file extensions in the text box, separated by commas.
    -   **Maximum file size** - Select to limit the file size to 2048 MB.
-   Use [`getFileTransferSetting`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.ChatClient.html#getFileTransferSetting) to determine whether this feature is available. If it is, use [`sendFile`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.ChatClient.html#sendFile) to send the file.

_**Note: Zoom deletes files sent via in-session chat when the session ends.**_

## Sample implementation

```html

```

```javascript
// handle file input change
const chatClient = client.getChatClient();
document
    .querySelector(".send-file-input")
    .addEventListener("change", async (event) => {
        const target = event.target;
        const { files } = target;
        if (files && files?.length > 0) {
            // send file
            window.cancelSendFile = await chatClient.sendFile(files[0], 0);
        }
        target.value = "";
    });
```

## Upload progress event

Zoom encrypts the file and uploads it to its file server. It may take a few moments to encrypt and upload the file. You can cancel the upload process by invoking the returned method at any time.

```javascript
client.on("chat-file-upload-progress", (payload) => {
    const { status, progress, fileName } = payload;
    if (status === ChatFileUploadStatus.InProgress) {
        // Todo: display file upload progress
        console.log(`File ${fileName} upload progress:${progress}`);
    } else if (status === ChatFileUploadStatus.Fail) {
        // Due to network issues, the file upload failed. You can retry the file upload using the `retryToken`.
        // When responding to the retry button on the UI, you can use the following code:
        const { retryToken } = payload;
        chatClient.sendFile(retryToken, userId);
    } else if (status === ChatFileUploadStatus.Cancel) {
        // The user has actively canceled the file upload.
        console.log(`File ${fileName} upload canceled`);
    } else if (status === ChatFileUploadStatus.Success) {
        console.log(`File ${fileName} upload successful.`);
    }
});

// If you no longer want to send the file, you can actively cancel the upload.
window.cancelSendFile();
```

## File information and download

When receiving a new message of type 'file,' you can obtain information about the file such as name, size, type, URL, etc., through the `file` field in the payload. However, direct access to the `fileURL` for downloading is not possible because the file is encrypted. To download the file, you must use the `chatClient.downloadFile` method.

```javascript
client.on("chat-on-message", async (payload) => {
    const {
        file,
        sender: { name, userId },
        id,
    } = payload;
    if (file !== undefined) {
        //Received a file message.
        const { name: fileName, fileUrl } = file;
        console.log(`Received a file from ${name},file name:${fileName}`);
        // The file cannot be downloaded directly from the file URL as it is encrypted. To download the file,use the `downloadFile` method.
        window.cancelDownloadFile = await chatClient.downloadFile(id, fileUrl);
    }
});
```

## Download progress event

Similarly, when downloading a large file, it may take a few moments. We also provide corresponding download progress events, and you can cancel the download at any time using the callback returned by the `chatClient.downloadFile` method.

```javascript
client.on("chat-file-download-progress", (payload) => {
    const { fileName, progress, status } = payload;
    if (status === ChatFileDownloadStatus.InProgress) {
        // Todo: display file download progress
        console.log(`File ${fileName} download progress:${progress}`);
    } else if (status === ChatFileDownloadStatus.Fail) {
        console.log(`File ${fileName} download failed`);
        // To re-download, call the `chatClient.downloadFile` again
    } else if (status === ChatFileDownloadStatus.Cancel) {
        console.log(`File ${fileName} download canceled`);
    } else if (status === ChatFileDownloadStatus.Success) {
        // The file will be downloaded directly through the browser's download page.
        console.log(`File ${fileName} download successful`);
        // If you want to obtain the file's blob data, for example, to display an image in the chat record list inline, you can specify the 'blob' parameter as true when calling the `downloadFile` method. This way, you can retrieve the file's blob through the 'fileBlob' property in the payload.
        const { fileBlob } = payload;
        const fileObjUrl = window.URL.createObjectURL(fileBlob);
        // assign the url to the image src attribute
    }
});

// If you no longer want to download the file, you can actively cancel the download.
window.cancelDownloadFile();
```
