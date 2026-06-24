# Windows Common Issues

## Linker Errors

Check:

- `<arch>/export_h` is in include directories.
- `<arch>/lib` is in library directories.
- `zToolSuiteIPCProxy.lib` is linked.
- The project architecture matches the selected package architecture.
- The active configuration is using the intended property set.

## Application Fails at Startup

- Copy the complete matching `<arch>/bin` dependency set beside the executable.
- Use a dependency inspection tool to identify missing DLLs.
- Do not copy only `zToolSuiteIPCProxy.dll`; its packaged dependencies are required.
- Confirm the Visual C++ runtime and target architecture.

## Listener Class Will Not Compile

`IZMToolSuiteProxyListener` is pure virtual. Implement every method in the selected package's `ToolSuiteProxyInterface.h`. Do not copy a callback list from another SDK version.

## Auth or IPC Never Becomes Ready

1. Confirm Zoom Workplace is installed and running.
2. Confirm the Workplace user matches the OAuth-authorized user.
3. Confirm `plugin_sdk:read:connection_meta`.
4. Confirm the token is current.
5. Keep the listener alive.
6. Log both `OnAuthResult` and `OnIPCConnectStatusChanged`, including `errorMessage`.
7. Handle `IPC_STATUS_CONNECT_TIMEOUT` and `IPC_STATUS_DISCONNECTED`.

## API Returns `false`

The request was not submitted. Check initialization, IPC state, meeting state, arguments, toolkit pointer, and meeting instance before expecting a completion.

## Completion Error

| Error | Likely check |
|---|---|
| `kZMToolSuiteProxyErrorsWrongUsage` | Lifecycle or state ordering |
| `kZMToolSuiteProxyErrorsInvalidArgument` | IDs, pointers, path, enum, or missing parameter |
| `kZMToolSuiteProxyErrorsNoPermission` | Role, host policy, account setting, or OS permission |
| `kZMToolSuiteProxyErrorsRateLimitReached` | Repeated actions; debounce and retry |
| `kZMToolSuiteProxyErrorsOperationTimeout` | IPC/client responsiveness |
| `kZMToolSuiteProxyErrorsVersionIncompatible` | Plugin SDK and Zoom Workplace compatibility |

## Meeting Control Fails

- Wait for `MEETING_STATUS_INMEETING`.
- Use the correct `ZMToolSuiteMeetingInstance`.
- Resolve the current role before host-only actions.
- Use `Can*`, `IsSupport*`, and state-query methods before changing state.
- Reconcile after reconnect, waiting-room movement, role changes, and breakout-room transitions.

## Window or Monitor Share Fails

- Call `CanStartShare`.
- Pass the platform-native identifier expected by `StartAppShare` or `StartMonitorShare`.
- Do not pass a window title as the `void*` source ID.
- Verify Windows privacy/capture permissions and host share policy.
- Observe `OnStartSendShare`, `OnStopSendShare`, `OnShareStatusChanged`, and content/error callbacks.

## UI Thread Problems

Post data to the owning window instead of updating controls directly from callbacks:

```cpp
PostMessage(hwnd, WM_APP + 1, statusCode, 0);
```

Copy any SDK-owned data required by the posted message before returning from the callback.
