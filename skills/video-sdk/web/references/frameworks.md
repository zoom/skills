<!-- Source: https://developers.zoom.us/docs/video-sdk/web/frameworks.md -->
<!-- Fetched: 2026-06-15 -->

# Using Frameworks


Some front-end web application frameworks, such as Angular, and module loaders, such as RequireJS require additional coding steps to integrate Video SDK.

## Using Angular

When using the SDK with Angular, you must run the SDK [outside the Angular zone](https://angular.io/guide/zone#ngzone-run-and-runoutsideofangular) and set up a `zone-flags.ts` file to disable unnecessary change detection and avoid performance issues.

In the `src` directory, create a `zone-flags.ts` file and add the following code:

```javascript
// disable patching requestAnimationFrame
(window as any).__Zone_disable_requestAnimationFrame = true;

// disable patching specified eventNames
(window as any).__zone_symbol__UNPATCHED_EVENTS = ['message'];
```

Then, in your `angular.json` file in the polyfills array, add `"src/zone-flags.ts"` before zone.js.

```json
"polyfills": [
  "src/zone-flags.ts",
  "zone.js"
]
```

Also add it to your tsconfig.app.json file in the include array:

```javascript
"include": [
  "src/**/*.d.ts",
  "src/zone-flags.ts"
]
```

Finally, in your file where you are calling the `init` and `join` functions, run them outside the Angular zone. See the example for details.

## Video SDK Angular example

```javascript
import { Component, NgZone } from '@angular/core';
import ZoomVideo from '@zoom/videosdk';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {

  client: any = ZoomVideo.createClient();
  stream: any = null;

  // ...

  constructor(private ngZone: NgZone) {

  }

  // ...

  join() {
    this.ngZone.runOutsideAngular(() => {
      this.client.init('en-US').then(() => {
        this.client.join(sessionName, videoSDKJWT, userName, session, sessionPassword).then(() => {
          this.stream = this.client.getMediaStream();
          // ...
        })
      })
    })
  }

  // ...

}
```

## Using AMD mode, RequireJS or named modules

Video SDK for web supports AMD mode (RequireJS, Dojo, Almond, curl.js). Only the SDK itself needs to be registered in your loader config. Internal dependencies are resolved automatically.

Replace `#.#.#` with [the latest version number](https://www.npmjs.com/package/@zoom/videosdk?activeTab=versions).

```javascript
requirejs.config({
    paths: {
        WebVideoSDK: "https://source.zoom.us/videosdk/zoom-video-#.#.#.min",
    },
});

function loadZoomVideo() {
    return new Promise((resolve, reject) => {
        requirejs(
            ["WebVideoSDK"],
            (WebVideoSDK) => {
                const { default: ZoomVideo } = WebVideoSDK;
                resolve(ZoomVideo);
            },
            reject,
        );
    });
}

async function main() {
    const ZoomVideo = await loadZoomVideo();
    const client = ZoomVideo.createClient();
    // your code here
}
```

## From the developer blog

-   [Build a video conferencing app with the Zoom Video SDK & Angular](/blog/angular-video-conferencing-app) by Rehema Armorer - 09-03-2024
