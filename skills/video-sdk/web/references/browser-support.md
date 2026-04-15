# Browser Support & SharedArrayBuffer

## Supported Browsers

SDK supports browsers within **2 versions** of current release.

### Desktop Browsers

| Browser | Video | Audio | Screen Share | Virtual BG | 1080p |
|---------|:-----:|:-----:|:------------:|:----------:|:-----:|
| Chrome | ✅ | ✅ | ✅ | ✅ | ✅ |
| Edge | ✅ | ✅ | ✅ | ✅ | ✅ |
| Firefox | ✅ | ✅ | ✅ | ✅* | ❌ |
| Safari | ✅ | ✅ | ✅ | ✅* | ❌ |

*Requires SharedArrayBuffer

### Mobile Browsers

| Platform | Video | Audio | Screen Share | 720p |
|----------|:-----:|:-----:|:------------:|:----:|
| iOS Safari | ✅ | ✅ | ❌ | ✅ (iOS 16.4+) |
| iOS Chrome | ✅ | ✅ | ❌ | ✅ (iOS 16.4+) |
| Android Chrome | ✅ | ✅ | ❌ | ✅ (Chrome 112+) |

**Note**: All iOS browsers use WebKit engine regardless of browser name.

### Feature Availability

| Feature | All Browsers | Limitations |
|---------|:------------:|-------------|
| Audio send/receive | ✅ | None |
| Video send/receive | ✅ | 720p max on mobile |
| Chat & file sharing | ✅ | None |
| Cloud recording | ✅ | None |
| Live transcription | ✅ | None |
| RTMP streaming | ✅ | None |
| Screen sharing | Desktop only | Not available on mobile |
| Virtual background | Desktop only | SAB required for Firefox/Safari |
| 1080p video | Chrome/Edge only | Desktop only |
| Noise suppression | SAB required | WebAssembly mode |
| End-to-end encryption | ❌ | Not supported on any browser |

## Checking Browser Support

```typescript
// Check basic compatibility BEFORE init
const { audio, video, screen } = ZoomVideo.checkSystemRequirements();

if (!audio) {
  console.error('Audio not supported');
}
if (!video) {
  console.error('Video not supported');
}
if (!screen) {
  console.log('Screen sharing not supported');
}

// Check detailed feature support
const features = ZoomVideo.checkFeatureRequirements();
console.log('Platform:', features.platform);
console.log('Supported:', features.supportFeatures);
console.log('Unsupported:', features.unSupportFeatures);
```

## SharedArrayBuffer (SAB)

### What is SharedArrayBuffer?

SharedArrayBuffer enables shared memory in JavaScript, improving performance for WebAssembly-based features:
- Multiple video rendering
- Virtual background
- Background noise suppression

### Is SAB Required?

**No** - The SDK works without SAB using WebRTC fallback, but with limitations:
- Max 4 concurrent videos (vs 25 with SAB)
- No virtual background on Firefox/Safari
- No noise suppression in WebAssembly mode

### Checking SAB Support

```typescript
// In browser console
console.log('SAB available:', typeof SharedArrayBuffer === 'function');
console.log('Cross-origin isolated:', crossOriginIsolated);
```

### Enabling SAB (COOP/COEP Headers)

Add these HTTP headers to your server:

```
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```

Or use credentialless mode:

```
Cross-Origin-Embedder-Policy: credentialless
Cross-Origin-Opener-Policy: same-origin
```

### Server Configuration Examples

**Nginx:**
```nginx
add_header Cross-Origin-Opener-Policy same-origin;
add_header Cross-Origin-Embedder-Policy require-corp;
```

**Apache:**
```apache
Header set Cross-Origin-Opener-Policy "same-origin"
Header set Cross-Origin-Embedder-Policy "require-corp"
```

**Express.js:**
```javascript
app.use((req, res, next) => {
  res.setHeader('Cross-Origin-Opener-Policy', 'same-origin');
  res.setHeader('Cross-Origin-Embedder-Policy', 'require-corp');
  next();
});
```

### Workarounds Without Server Headers

#### 1. Service Worker (e.g., GitHub Pages)

Use [coi-serviceworker](https://github.com/nickelow/coi-serviceworker):

```html
<script src="coi-serviceworker.js"></script>
```

#### 2. Document-Isolation-Policy (Chrome 137+)

```
Document-Isolation-Policy: isolate-and-require-corp
```

Or:
```
Document-Isolation-Policy: isolate-and-credentialless
```

This allows third-party iframes while maintaining isolation.

#### 3. Chrome Origin Trial (Temporary)

Register at [Chrome Origin Trials](https://developer.chrome.com/origintrials/) for temporary SAB access without headers. Requires renewal every 3 months.

### Without SAB: Init Options

```typescript
await client.init('en-US', 'Global', {
  // Enable multiple videos without SAB
  enforceMultipleVideos: true,

  // Enable > 4 videos (use carefully - high CPU)
  enforceMultipleVideos: { disableRenderLimits: true },

  // Enable virtual background without SAB
  enforceVirtualBackground: true,
});
```

## Content Security Policy (CSP)

Allow Zoom SDK resources in your CSP:

```
Content-Security-Policy:
  script-src 'self' https://source.zoom.us 'wasm-unsafe-eval';
  worker-src 'self' blob:;
  connect-src 'self' https://*.zoom.us wss://*.zoom.us;
```

**Note**: `wasm-unsafe-eval` or `unsafe-eval` required for WebAssembly compilation.

## Known Issues

### Edge Purple Background

Microsoft Edge may show purple backgrounds when receiving video due to enhanced security features.

**Solutions:**
- User can disable "Enhance your security on the web" in Edge settings
- Add your domain to Edge's exceptions list

### Safari Auto-Play

Safari blocks audio auto-play more strictly:

```typescript
await stream.startAudio({
  autoStartAudioInSafari: true,  // For macOS 15.2 - 16.0
});
```

### Firefox leaveOnPageUnload

The `leaveOnPageUnload` option doesn't work in Firefox due to browser limitations.

## Mobile Considerations

### iOS Landscape for 720p

720p video on iOS requires:
- iOS 16.4 or later
- Landscape orientation

### Android Camera Switching

Use `MobileVideoFacingMode` instead of device IDs:

```typescript
// Front camera
await stream.switchCamera(MobileVideoFacingMode.User);

// Back camera
await stream.switchCamera(MobileVideoFacingMode.Environment);
```

## Performance Recommendations

1. **Enable SAB** when possible for best performance
2. **Use appropriate video quality** based on network/device
3. **Limit concurrent videos** on low-powered devices
4. **Preload assets** with `preloadDependentAssets()`
5. **Monitor system resources** via `system-resource-usage-change` event

```typescript
client.on('system-resource-usage-change', (payload) => {
  // payload.level: SystemCPUPressureLevel
  if (payload.level >= SystemCPUPressureLevel.Serious) {
    // Reduce video quality or disable features
  }
});
```
