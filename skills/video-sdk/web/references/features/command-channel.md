<!-- Source: https://developers.zoom.us/docs/video-sdk/web/command-channel.md -->
<!-- Fetched: 2026-06-15 -->

# Command channel


The command channel is a ready-to-use, flexible real-time service through which you can send data to other users in the current session. Commands can be sent directly to specific users or to the session as a whole. It can be used to:

-   send real-time reactions to live content
-   conduct live surveys and polls during video sessions
-   share access to client-side settings of one participant to another participants's client

At a high level, this functions similarly to chat messages, you provide a receiver and the command you would like to send as a string. Receive it with an event listener.

Using a distinct service for commands instead of sending them through chat allows for better separation of concerns in your implementation and reduces the likelihood of encountering bugs.

The maximum size is 512 characters and the rate limit is 2 commands per second per session. If you require a higher command frequency, contact [Developer Support](/support/) to request an increase.

Command channel is not designed for high frequency commands broadcasting, and high command frequencies may impact the session experience. To maintain optimal session performance, some commands may be dropped under heavy load. If your application requires high availability, high frequency, and N-to-N broadcast topology, we recommend deploying a dedicated signaling service for your app.

## Init command channel

After joining a session, call `client.getCommandClient()` to get the command channel client.

```javascript
const commandChannel = client.getCommandClient();
```

## Send commands

Pass in a string as the command value. You can pass in JSON, just make sure to `JSON.stringify(command)` before passing it in.

If you don't set a `userId`, the command is sent to all users.

```javascript
commandChannel.send(command, userId);
```

## Receive commands

To receive commands, add the following event listener.

```javascript
client.on(`command-channel-message`, (payload) => {
    console.log(payload);
    console.log(`Command from ${payload.senderName} is ${payload.text}`);
});
```

## More command channel features

For the full set of command channel features, see [CommandChannel in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.CommandChannel.html).
