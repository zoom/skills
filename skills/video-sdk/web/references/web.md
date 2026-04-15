# Video SDK Web Compatibility Reference

This file remains as a compatibility index for repo links that previously targeted the older monolithic web reference.

## Primary Imported References

- [sessions.md](sessions.md)
- [audio.md](audio.md)
- [video.md](video.md)
- [screen-share.md](screen-share.md)
- [features.md](features.md)
- [advanced.md](advanced.md)
- [browser-support.md](browser-support.md)
- [error-codes.md](error-codes.md)
- [../troubleshooting/common-issues.md](../troubleshooting/common-issues.md)
- [react.md](react.md)
- [vue.md](vue.md)
- [angular.md](angular.md)
- [svelte.md](svelte.md)

## Video Rendering Best Practices

For rendering guidance, use [video.md](video.md#rendering-video).

Key points retained for callers that linked here:
- Use `attachVideo()`, not deprecated `renderVideo()`.
- Render into a `video-player-container` element with explicit CSS sizing.
- Listen for `peer-video-state-change` and reconcile attach/detach with participant state.
- When you join mid-session, manually render already-active participant video after join.
