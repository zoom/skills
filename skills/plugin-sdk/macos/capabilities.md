# macOS Plugin SDK Capabilities

Use [Plugin SDK Capabilities](../capabilities.md) for the grouped capability inventory.

Verify exact macOS names, signatures, callbacks, and availability against [Package and API Map](references/package-and-api-map.md) before writing code. The macOS package surface is exposed through `ZoomToolSuite.framework` Objective-C headers and Swift-imported namespaces.

## Platform Checks

- Verify calls against macOS headers first.
- Validate behavior with package samples.
- Confirm AppKit main-thread UI updates.
- Re-sign packaged frameworks before shipping.
- Check account settings and meeting roles.
- Prefer capability checks before privileged calls.
