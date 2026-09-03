<!-- Source: https://developers.zoom.us/docs/video-sdk/web/transcription-translation.md -->
<!-- Fetched: 2026-06-15 -->

# Live transcription and translation


You can receive speech as JSON objects, in real time, using the Zoom [live transcription and translation feature](https://support.zoom.us/hc/en-us/articles/6643133682957-Enabling-and-configuring-translated-captions) (LTT). This feature can also translate speech from one language in real time to text in another language. This can power use cases like auto closed captioning, sentiment analysis, and language translation for e-learning.

For example, one user can say "Hello world" in English, and the other users can receive this speech, as text, in the language of their choice, like Italian "Ciao mondo", Spanish "Hola Mundo", and French "Bonjour le monde".

**Zoom recommends that you provide an in-product notice to your end-users when a participant enables live transcription.**

> **View usage**
>
> To monitor transcription (caption) and translation activity for your Video SDK sessions, see [Build Platform → View usage reports: Captions and translation](/docs/build/reports/#view-usage-reports) for detailed usage data.

## Initialize LTT

After joining a session, call `client.getLiveTranscriptionClient()` to get the live transcription and translation client.

```javascript
const liveTranscriptionTranslation = client.getLiveTranscriptionClient();
```

## Supported languages

We are continuously adding to our [supported languages](https://marketplacefront.zoom.us/sdk/custom/ZoomVideo.LiveTranscriptionLanguage.html) for transcription and translation. To get the list of supported [language](https://marketplacefront.zoom.us/sdk/custom/web/enums/ZoomVideo.LiveTranscriptionLanguage.html), call `liveTranscriptionTranslation.getLiveTranscriptionStatus()`. The `transcriptionLanguage` property lists the supported languages.

```javascript
liveTranscriptionTranslation.getLiveTranscriptionStatus();
```

-   To specify the [language](https://marketplacefront.zoom.us/sdk/custom/web/enums/ZoomVideo.LiveTranscriptionLanguage.html) you are speaking in, call `liveTranscriptionTranslation.setSpeakingLanguage()`.
-   To specify the [language](https://marketplacefront.zoom.us/sdk/custom/web/enums/ZoomVideo.LiveTranscriptionLanguage.html) you want translated to, call `liveTranscriptionTranslation.setTranslationLanguage()`.

The calls have to be promise chained or awaited. See [start live transcription and translation](#start-live-transcription-and-translation) for an example and [samples scenarios](#sample-scenarios) for detailed use cases.

## Start transcription and translation

To start live transcription and translation, call `liveTranscriptionTranslation.startLiveTranscription()`.

```javascript
liveTranscriptionTranslation.startLiveTranscription().then(() => {
    liveTranscriptionTranslation.setTranslationLanguage("ja");
    liveTranscriptionTranslation.setSpeakingLanguage("ja");
});
```

```javascript
await liveTranscriptionTranslation.startLiveTranscription();

liveTranscriptionTranslation.setTranslationLanguage("ja");
liveTranscriptionTranslation.setSpeakingLanguage("ja");
```

## Receive transcription

To receive speech text, add the following event listener.

```javascript
client.on(`caption-message`, (payload) => {
    console.log(payload);
    console.log(`${payload.displayName} said: ${payload.text}`);
});
```

## Receive translation

To receive translated speech text, add the following event listener.

```javascript
client.on(`caption-message`, (payload) => {
    console.log(payload);
    console.log(
        `${payload.displayName} said: ${payload.text}, translated to ${payload.language}`,
    );
});
```

## Stop live transcription and translation

To stop live transcription and translation, call `liveTranscriptionTranslation.disableCaptions()`.

```javascript
liveTranscriptionTranslation.disableCaptions();
```

**Note: This disables the feature for all users in the session.**

## Sample scenarios

You don't need to use transcript and translation at the same time. Here are some sample scenarios that show the different use cases.

### Scenario 1: Set speaking language with no translation enabled

The `setSpeakingLanguage` method specifies the user's spoken language. By default, this setting only applies to the user and does not affect others in the session. When no translation is enabled, users typically communicate in the same language. Any user can update their spoken language in the session.

```javascript
// Set session-level speaking language
liveTranscriptionTranslation.setSpeakingLanguage(lang, {
    mode: TranscriptionMode.Session,
});
```

When the session-level speaking language changes, users receive the `caption-status` event.

```javascript
client.on("caption-status", (payload) => {
    const { sessionLanguage } = payload;
    if (sessionLanguage) {
        console.log(`Session language has been changed to ${sessionLanguage}`);
    }
});
```

#### Restrict control to hosts

To restrict language changes to session hosts only, lock the transcription language setting.

```javascript
liveTranscriptionTranslation.lockTranscriptionLanguage(true);
```

### Scenario 2: Set translation language and enable translation

The `setTranslationLanguage` method sets the target language for translation. You can also set the speaking language and translation language to be the same (this is a common configuration). You can only call this method after you've enabled transcription.

```javascript
// Enable transcription first, then set translation language
liveTranscriptionTranslation.startLiveTranscription().then(() => {
    liveTranscriptionTranslation.setTranslationLanguage("ja"); // Japanese
});
```

When you've enabled translation, users may speak different languages. In this case, `setSpeakingLanguage` only affects the individual user's spoken language, regardless of the `mode:TranscriptionMode.Session` option setting. Listen for the `caption-status` event when translation becomes available.

```javascript
client.on("caption-status", (payload) => {
    const { translationStarted } = payload;
    if (translationStarted) {
        // Prompt users who haven't set their speaking language
        // to do so for better translation accuracy
        showLanguageSelectionPrompt();
    }
});
```

## LTT best practices

When implementing Live Transcription and Translation (LTT) for your integration, consider the following best practices:

-   If the feature is enabled for the session, provide a button to allow people to start closed captioning and select the spoken and translated languages.
-   Display only the supported languages you wish to offer, rather than presenting all available options for transcription and translation.
-   If the session won't include LTT, programmatically disable when the host starts the session. When someone joins the session, check if the feature is enabled by the host. If not, inform users.
-   Use an event listener to detect when the host has enabled captions. You can use this to notify people that the feature is active, programmatically render a button for starting transcription or translation, or both.
-   Set `enableReceiveSpokenLanguageContent()` to false if you don't want to receive the spoken language data.
-   Use an event listener to detect when the host disables captions. You can use this to notify people who have enabled the feature that it has been disabled.
-   Offer closed captioning customization options, such as font sizes and colors, to differentiate between transcription and translation texts, to enhance readability and follow accessibility standards.
-   Inform people of these best practices when speaking:
    -   Minimize background noise, avoiding activities like shuffling papers, typing loudly, or engaging in side conversations.
    -   Speak clearly into the microphone.
    -   Position the microphone near active speakers.
    -   Opt for an external microphone over a built-in one to improve sound quality.

## More LTT features

You can also [control user audio volume](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#adjustUserAudioVolumeLocally) and [mute local audio](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#muteUserAudioLocally) to further refine live transcription and translation use cases.

For the full set of live transcription and translation features, see [LiveTranscriptionClient in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.LiveTranscriptionClient.html).
