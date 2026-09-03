<!-- Source: https://developers.zoom.us/docs/video-sdk/web/subsessions.md -->
<!-- Fetched: 2026-07-17 -->

# Subsessions


Subsessions (also known as breakout rooms) allow you to split users in a session into separate smaller sessions for group discussions or activities. The host can assign users to subsessions automatically (distributed evenly) or manually. A Video SDK session can have up to 50 subsessions.

## Key concepts

-   **Host/Manager privileges** - Only the host or manager can create, open, manage, and close subsessions
-   **Subsession states** - `NotStarted`, `InProgress`, `Closing`, `Closed`
-   **User states** - `Initial`, `Invited`, `InSubsession`
-   **Allocation patterns** - Evenly distributed or manually assigned
-   **Timer support** - Optional countdown timer with auto-close functionality
-   **User mobility** - Users can move between subsessions or return to the main session

## Init subsessions

After joining a session, call `client.getSubsessionClient()` to get the subsession client.

```javascript
const subsession = client.getSubsessionClient();
```

## Get subsession status

To check the current subsession status, call `subsession.getSubsessionStatus()`. This returns one of four possible states.

```javascript
const status = subsession.getSubsessionStatus();
// Returns: SubsessionStatus.NotStarted | SubsessionStatus.InProgress | SubsessionStatus.Closing | SubsessionStatus.Closed
```

## Get user status

To check the current user's subsession status, call `subsession.getUserStatus()`. This returns the user's current state in relation to subsessions.

```javascript
const userStatus = subsession.getUserStatus();
// Returns: SubsessionUserStatus.Initial | SubsessionUserStatus.Invited | SubsessionUserStatus.InSubsession
```

## Get current subsession

To get information about the subsession that the current user is in, call `subsession.getCurrentSubsession()`.

```javascript
const current = subsession.getCurrentSubsession();
// Returns: { subsessionId: string, subsessionName: string, userStatus: SubsessionUserStatus }
```

## Configure subsessions

The host can configure subsessions.

### Create subsessions

To create a subsession, call the `subsession.createSubsessions()` function. For the first parameter, pass in a number to specify how many subsessions to create, or an array of subsession names.

The second parameter specifies the allocation pattern.

-   `SubsessionAllocationPattern.Evenly` (value: `1`) - Automatically distributes users evenly to each subsession.
-   `SubsessionAllocationPattern.Manually` (value: `2`) - Allows you to assign users to subsessions manually (default).

```javascript
// Create 3 subsessions with auto-assignment
subsession.createSubsessions(3, 1);

// Or create with custom names and manual assignment
subsession.createSubsessions(["Room A", "Room B", "Room C"], 2);
```

### Manually assign users to a subsession

Get the subsession status using `subsession.getSubsessionStatus()`. Depending on the current status, there are two approaches to assigning users, subsessions not started, or subsessions already started.

-   **Subsessions not started** (`SubsessionStatus.NotStarted` or `SubsessionStatus.Closed`). Pass a `subsessionList` when calling `subsession.openSubsessions()` to pre-assign users to subsessions. Each item in the list can include not only the `subsessionId` and `subsessionName`, but also a `userList`. This assigns users to their respective subsessions when you open the subsessions.

    ```javascript
    const subsessionList = subsession.getSubsessionList();
    function assignUserToSubsessionBeforeStarted(
        userId,
        subsessionId,
        subsessionList,
    ) {
        const session = subsessionList.find(
            (item) => item.subsessionId === subsessionId,
        );
        if (session) {
            const { userList } = session;
            if (!userList.find((user) => user.userId == userId)) {
                userList.push({ userId });
            }
        }
    }
    ```

-   **Subsessions already started** (`SubsessionStatus.InProgress`). In this case, use the `subsession.assignUserToSubsession(userId,subsessionId)` method to dynamically assign users to the active subsessions.

## Open subsessions

After configuring your subsessions, call the `subsession.openSubsessions()` function to start the subsessions. The first parameter is the subsession list (obtained via `getSubsessionList()`), and the second parameter is an options object with the following properties:

-   `isTimerEnabled` - Enable a countdown timer for the subsessions
-   `timerDuration` - Duration in seconds (e.g., 1800 for 30 minutes)
-   `isTimerAutoStopSubsession` - Automatically close subsessions when timer expires
-   `waitSeconds` - Countdown in seconds before opening subsessions (gives users time to prepare)
-   `isBackToMainSessionEnabled` - Allow users to return to the main session
-   `isAutoMoveBackToMainSession` - Automatically move users back to the main session when subsessions close
-   `isSubsessionSelectionEnabled` - Allow users to choose which subsession to join

```javascript
subsession.openSubsessions(subsession.getSubsessionList(), {
    isTimerEnabled: true,
    timerDuration: 1800, // 30 minutes
    isTimerAutoStopSubsession: false,
    waitSeconds: 60, // 60 second countdown before opening
    isBackToMainSessionEnabled: true,
    isAutoMoveBackToMainSession: true,
    isSubsessionSelectionEnabled: false,
});
```

See the [corresponding event listeners](#subsession-state-change).

## Manage subsessions

The host can manage subsessions. Users can interact with sessions and the host.

### Broadcast to subsessions

The host can broadcast both text messages and voice to all subsessions.

#### Broadcast text message

```javascript
subsession.broadcast("Hello World");
```

See the [`subsession-broadcast-message`](#broadcast-message) event for how users receive broadcast messages.

#### Broadcast voice

The host can broadcast their audio to all subsessions simultaneously. Users must have joined audio to use this feature.

```javascript
// Start broadcasting voice
await subsession.startBroadcastVoice();

// Stop broadcasting voice
await subsession.stopBroadcastVoice();
```

See the [`subsession-broadcast-voice`](#broadcast-voice) event for how the SDK notifies users when voice broadcast starts or stops.

### Chat in subsession

Users can [chat](/docs/video-sdk/web/chat/) in a subsession. The SDK scopes the chat to the subsession the user is in.

### Ask for help

To have a user notify the host in a subsession that they need attention, call the `subsession.askForHelp()` function. This is typically used by attendees who need assistance.

```javascript
subsession.askForHelp();
```

See the [corresponding event listeners](#ask-for-help) for how the host receives and responds to help requests.

### Postpone response to help

When a user asks for help, the host can respond by either joining their subsession or postponing. To postpone, call the `subsession.postponeHelping()` function with the user's ID. This notifies the user that the host will assist them later.

```javascript
subsession.postponeHelping(userId);
```

The user will receive a response via the [`subsession-ask-for-help-response`](#ask-for-help-response) event.

### List users in a subsession

To list subsessions including users in the subsession, call the `subsession.getSubsessionList()` function. If the host calls this function they will get all the subsessions. If a user calls the function, they will get their current subsession.

```javascript
subsession.getSubsessionList();
```

### List users not in a subsession

To list users who are not in a subsession, call the `subsession.getUnassignedUserList()` function.

```javascript
subsession.getUnassignedUserList();
```

### Get subsession options

To get the current subsession configuration options, call the `subsession.getSubsessionOptions()` function. This returns the options set when opening subsessions.

```javascript
const options = subsession.getSubsessionOptions();
```

## Navigate subsessions

Users can navigate to subsessions, and the host can move users between them.

### Join subsession

To join a subsession, call the `subsession.joinSubsession()` function. This function is useful to place a user who joins the session after you open the subsessions.

```javascript
subsession.joinSubsession(subsessionId);
```

### Leave subsession

To leave a subsession, call the `subsession.leaveSubsession()` function.

```javascript
subsession.leaveSubsession();
```

### Move user to subsession

To have the host move a user to a subsession, call the `subsession.moveUserToSubsession()` function.

```javascript
subsession.moveUserToSubsession(userId, subsessionId);
```

### Move user back to main session

To have the host move a user back to the main session, call the `subsession.moveBackToMainSession()` function.

```javascript
subsession.moveBackToMainSession(userId);
```

## Close subsessions

To close all subsessions, call the `subsession.closeAllSubsessions()` function.

```javascript
subsession.closeAllSubsessions();
```

## Event listeners

Event listeners are essential for tracking subsession state changes and user interactions. The following are all available subsession events.

### Subsession state change

The `subsession-state-change` event fires whenever the subsession status changes. This is the primary event for tracking subsession lifecycle.

```javascript
client.on("subsession-state-change", (payload) => {
    console.log(payload.status);
    // status values:
    // - SubsessionStatus.NotStarted (0)
    // - SubsessionStatus.InProgress (1)
    // - SubsessionStatus.Closing (2)
    // - SubsessionStatus.Closed (3)
});
```

#### Subsession state change payload

```typescript
{
    status: SubsessionStatus;
}
```

### Subsession countdown

The `subsession-countdown` event fires when you open subsessions with a timer. Use this to display a countdown timer to users.

```javascript
client.on("subsession-countdown", (payload) => {
    console.log(`Time remaining: ${payload.countdown} seconds`);
});
```

#### Subsession countdown payload

```typescript
{
    countdown: number; // Remaining seconds
}
```

### Closing subsession countdown

The `closing-subsession-countdown` event fires when the host closes subsessions with a countdown. This gives users time to wrap up their discussions.

```javascript
client.on("closing-subsession-countdown", (payload) => {
    console.log(`Subsessions closing in ${payload.countdown} seconds`);
});
```

#### Closing subsession countdown payload

```typescript
{
    countdown: number; // Seconds until subsessions close
}
```

### Subsession time up

The `subsession-time-up` event fires when the subsession timer reaches zero. Hosts typically use this to decide whether to close subsessions or extend the time.

```javascript
client.on("subsession-time-up", () => {
    console.log("Subsession timer has expired");
});
```

### Main session user updated

The `main-session-user-updated` event fires when unassigned users in the main session change. This helps track changes to the unassigned user list.

```javascript
client.on("main-session-user-updated", () => {
    // Refresh unassigned user list
    const unassignedUsers = subsession.getUnassignedUserList();
    console.log("Main session users updated", unassignedUsers);
});
```

### Invite to join subsession

The `subsession-invite-to-join` event fires when the host invites a user to join a specific subsession. Users receive this event and can choose to accept or decline.

```javascript
client.on("subsession-invite-to-join", (payload) => {
    console.log(
        `You've been invited to join ${payload.subsessionName} (ID: ${payload.subsessionId})`,
    );
    // User can call subsession.joinSubsession(payload.subsessionId) to accept
});
```

#### Invite to join subsession payload

```typescript
{
    subsessionId: string;
    subsessionName: string;
}
```

### Invite back to main session

The `subsession-invite-back-to-main-session` event fires when the host invites a user to return to the main session from a subsession.

```javascript
client.on("subsession-invite-back-to-main-session", (payload) => {
    console.log(`${payload.inviterName} invited you back to the main session`);
    // User can call subsession.leaveSubsession() to return
});
```

#### Invite back to main session payload

```typescript
{
    inviterId: number;
    inviterGuid: string;
    inviterName: string;
}
```

### Broadcast message

The `subsession-broadcast-message` event fires when the host broadcasts a text message to all subsessions.

```javascript
client.on("subsession-broadcast-message", (payload) => {
    console.log(`Broadcast message: ${payload.message}`);
    console.log(`From: ${payload.senderName}`);
});
```

#### Broadcast message payload

```typescript
{
    message: string;
    senderName: string;
}
```

### Broadcast voice

The `subsession-broadcast-voice` event fires when the host starts or stops broadcasting their voice to all subsessions.

```javascript
client.on("subsession-broadcast-voice", (payload) => {
    if (payload.status) {
        console.log("Host is broadcasting voice to all subsessions");
    } else {
        console.log("Host stopped broadcasting voice");
    }
});
```

#### Broadcast voice payload

```typescript
{
    status: boolean; // true = started, false = stopped
}
```

### Ask for help

The `subsession-ask-for-help` event fires when a user in a subsession requests help from the host. Only the host receives this event.

```javascript
client.on("subsession-ask-for-help", (payload) => {
    console.log(
        `${payload.displayName} in ${payload.subsessionName} is asking for help`,
    );
    // Host can join the subsession or postpone
    // subsession.joinSubsession(payload.subsessionId)
    // subsession.postponeHelping(payload.userId)
});
```

#### Ask for help payload

```typescript
{
    userId: number;
    displayName: string;
    subsessionId: string;
    subsessionName: string;
}
```

### Ask for help response

The `subsession-ask-for-help-response` event fires when the host responds to a help request. The user who requested help receives this event.

```javascript
client.on("subsession-ask-for-help-response", (payload) => {
    switch (payload.result) {
        case 0: // AskHostHelpResponse.Received
            console.log("The host has received your request");
            break;
        case 1: // AskHostHelpResponse.Busy
            console.log("The host is currently busy in another subsession");
            break;
        case 2: // AskHostHelpResponse.Ignore
            console.log("The host will assist you later");
            break;
        case 3: // AskHostHelpResponse.AlreadyInRoom
            console.log("The host is already in your subsession");
            break;
    }
});
```

#### Ask for help response payload

```typescript
{
    result: AskHostHelpResponse; // 0=Received, 1=Busy, 2=Ignore, 3=AlreadyInRoom
}
```

## Event listener usage example

Here's a complete example showing how to handle subsession events in your application.

```javascript
const zmClient = ZoomVideo.createClient();
const subsessionClient = zmClient.getSubsessionClient();

// Track subsession state
client.on("subsession-state-change", (payload) => {
    updateSubsessionUI(payload.status);
});

// Handle invitations
client.on("subsession-invite-to-join", (payload) => {
    showInviteDialog(payload.subsessionName, () => {
        subsessionClient.joinSubsession(payload.subsessionId);
    });
});

// Display countdown
client.on("subsession-countdown", (payload) => {
    updateCountdownDisplay(payload.countdown);
});

// Host: handle help requests
if (zmClient.isHost()) {
    client.on("subsession-ask-for-help", (payload) => {
        showHelpRequest(payload.displayName, payload.subsessionName, () => {
            subsessionClient.joinSubsession(payload.subsessionId);
        });
    });
}

// User: handle help response
client.on("subsession-ask-for-help-response", (payload) => {
    showHelpResponseMessage(payload.result);
});

// Display broadcasts
client.on("subsession-broadcast-message", (payload) => {
    showBroadcastNotification(payload.senderName, payload.message);
});
```

## Best practices

Here are some best practices.

### Listen to state changes

Always listen to `subsession-state-change` to keep your UI synchronized with the actual subsession state. This event fires whenever subsessions open, close, or change status.

```javascript
client.on("subsession-state-change", (payload) => {
    // Update UI based on payload.status
    const subsessionList = subsessionClient.getSubsessionList();
    const currentSubsession = subsessionClient.getCurrentSubsession();
    updateUI(payload.status, subsessionList, currentSubsession);
});
```

### Handle user assignment changes

When users join or leave the session while subsessions are active, listen to `main-session-user-updated` to refresh the unassigned user list.

```javascript
client.on("main-session-user-updated", () => {
    const unassignedUsers = subsessionClient.getUnassignedUserList();
    updateUnassignedUsersList(unassignedUsers);
});
```

### Provide clear user feedback

When implementing invite-to-join functionality, give users clear options to accept or decline.

```javascript
client.on("subsession-invite-to-join", (payload) => {
    showModal({
        title: "Join Subsession",
        message: `You've been assigned to ${payload.subsessionName}`,
        onAccept: () => subsessionClient.joinSubsession(payload.subsessionId),
        onDecline: () => console.log("User declined invitation"),
    });
});
```

### Implement help request workflow

For hosts, implement a clear workflow for handling help requests.

```javascript
client.on("subsession-ask-for-help", (payload) => {
    showHelpDialog({
        message: `${payload.displayName} in ${payload.subsessionName} needs help`,
        onJoin: () => subsessionClient.joinSubsession(payload.subsessionId),
        onLater: () => subsessionClient.postponeHelping(payload.userId),
    });
});
```

### Handle timer expiration

When using timers, decide whether to automatically close subsessions or prompt the host.

```javascript
client.on("subsession-time-up", () => {
    if (zmClient.isHost()) {
        showConfirmDialog({
            message: "Subsession timer expired. Close all subsessions?",
            onConfirm: () => subsessionClient.closeAllSubsessions(),
            onCancel: () => console.log("Keep subsessions open"),
        });
    }
});
```

## Common scenarios

Here are some examples of common scenarios.

### Scenario 1: Create and open subsessions with even distribution

```javascript
const subsessionClient = client.getSubsessionClient();

// Create 5 subsessions with even distribution
await subsessionClient.createSubsessions(5, 1);

// Open with 30-minute timer
subsessionClient.openSubsessions(subsessionClient.getSubsessionList(), {
    isTimerEnabled: true,
    timerDuration: 1800,
    isAutoMoveBackToMainSession: true,
});
```

### Scenario 2: Manual assignment before opening

```javascript
// Create subsessions manually
await subsessionClient.createSubsessions(3, 2);

// Get subsessions and users
const subsessions = subsessionClient.getSubsessionList();
const unassignedUsers = subsessionClient.getUnassignedUserList();

// Assign users to subsessions before opening
function assignUserToSubsessionBeforeStarted(
    userId,
    subsessionId,
    subsessionList,
) {
    const subsession = subsessionList.find(
        (item) => item.subsessionId === subsessionId,
    );
    if (subsession) {
        const { userList } = subsession;
        if (!userList.find((user) => user.userId == userId)) {
            userList.push({ userId });
        }
    }
}

// Assign each user
unassignedUsers.forEach((user, index) => {
    const targetSubsessionId =
        subsessions[index % subsessions.length].subsessionId;
    assignUserToSubsessionBeforeStarted(
        user.userId,
        targetSubsessionId,
        subsessions,
    );
});

// Open subsessions
subsessionClient.openSubsessions(subsessions, {
    isTimerEnabled: false,
    isBackToMainSessionEnabled: true,
});
```

### Scenario 3: Move users between subsessions dynamically

```javascript
// This only works when subsessions are in progress
if (subsessionClient.getSubsessionStatus() === SubsessionStatus.InProgress) {
    subsessionClient.moveUserToSubsession(userId, targetSubsessionId);
}
```

### Scenario 4: Close subsessions with countdown

```javascript
// Close all subsessions (triggers closing countdown if configured)
subsessionClient.closeAllSubsessions();

// Listen for closing countdown
client.on("closing-subsession-countdown", (payload) => {
    showNotification(`Returning to main session in ${payload.countdown}s`);
});
```

## More subsession features

For the full set of subsession features, see [SubsessionClient in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.SubsessionClient.html).
