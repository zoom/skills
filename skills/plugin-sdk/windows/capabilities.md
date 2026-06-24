# Windows Plugin SDK Capabilities

Use [Plugin SDK Capabilities](../capabilities.md) for the grouped capability inventory.

Verify exact Windows names, signatures, callbacks, and availability against [Package and API Map](references/package-and-api-map.md) before writing code. The Windows package surface is exposed through C++ headers under the selected `x86` or `x64` `export_h` folder.

## Platform Checks

- Verify calls against Windows headers first.
- Validate behavior with PSDKTest samples.
- Keep architecture selection consistent everywhere.
- Ship matching runtime DLL dependencies.
- Marshal callbacks onto UI threads.
- Prefer capability checks before privileged calls.
