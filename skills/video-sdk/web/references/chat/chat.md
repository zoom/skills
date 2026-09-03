<!-- Source: https://developers.zoom.us/docs/video-sdk/web/chat.md -->
<!-- Fetched: 2026-06-15 -->

# Chat


Chat is is designed to support in-session text communication. Zoom truncates messages exceeding a binary size of 10,000 bytes. As a best practice, we recommend a limit of 1000 characters per message.

Chats can be sent during a session or in a [subsession](/docs/video-sdk/web/subsessions/) to and from all users or select users. For subsessions, the SDK scopes the chat to the subsession the user is in. See the following core features for chat such as how to initialize, send, and receive chats and get chat history. Then see the send files section to learn how to send files in chat.

## Init chat

After joining a session, call `client.getChatClient()` to get the chat client.

```javascript
const chat = client.getChatClient();
```

## Send chats

To send a chat to a specific user, call the `chat.send()` function.

```javascript
chat.send("Hey user", userId);
```

To send a chat to all users, call the `chat.sendToAll()` function.

```javascript
chat.sendToAll("Hey everyone");
```

## Receive chats

To receive chats sent from other users, use the `chat-on-message` event.

```javascript
client.on("chat-on-message", (payload) => {
    console.log(payload);
    console.log(
        `Message: ${payload.message}, from ${payload.sender.name} to ${payload.receiver.name}`,
    );
});
```

## Get chat history

To get the chat history for chats sent to everyone in the session, call the `chat.getHistory()` function. This returns chats that were sent and received while the user was in the session.

```javascript
chat.getHistory();
```

## More chat features

For the full set of chat features, see [ChatClient in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.ChatClient.html).
