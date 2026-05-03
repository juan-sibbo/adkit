# Video Players — Technical reference

## Table of contents
1. Player matrix by CTV environment
2. ContentPlayhead: correct implementation
3. Video.js + IMA SDK
4. JW Player: ad schedule and callbacks
5. Shaka Player: ad manager and integration
6. ExoPlayer (Android TV) + IMA extension
7. AVPlayer (tvOS) + IMA iOS SDK
8. hls.js / dash.js in web-based CTV environments
9. Player + SDK integration anti-patterns

---

## 1. Player matrix by CTV environment

| Environment | Recommended player | IMA SDK variant | Notes |
|---|---|---|---|
| Android TV | ExoPlayer | IMA Android SDK (ExoPlayer extension) | Official integration via `ImaAdsLoader` |
| Fire TV | ExoPlayer | IMA Android SDK | Identical to Android TV |
| tvOS / Apple TV | AVPlayer | IMA iOS SDK | AVPlayer mandatory per Apple policy |
| Tizen (Samsung) | Video.js / Shaka / custom | IMA HTML5 SDK | WebView; watch for memory leaks |
| webOS (LG) | Video.js / Shaka / custom | IMA HTML5 SDK | WebView + webOS Media API |
| HbbTV | Custom (broadcaster) / Video.js | IMA HTML5 SDK | Very variable by broadcaster; limited resources |
| Web (OTT) | Video.js / JW Player / Shaka | IMA HTML5 SDK | Most flexible environment |
| Chromecast | Custom CAF receiver | IMA CAF SDK | Receiver app on Cast device |

---

## 2. ContentPlayhead: correct implementation

The IMA HTML5 SDK queries `ContentPlayhead.getCurrentTime()` approximately 4 times per second to detect mid-roll cue points. An incorrect implementation is the most frequent cause of mid-rolls that never fire.

**Correct implementation:**
```javascript
// ContentPlayhead interface must return the player's REAL position in real time
const contentPlayhead = {
  getCurrentTime: () => videoElement.currentTime  // read directly from the DOM
};

adsManager.init(width, height, google.ima.ViewMode.NORMAL);
// Pass the playhead to the SDK
adsLoader.addEventListener(
  google.ima.AdsManagerLoadedEvent.Type.ADS_MANAGER_LOADED,
  (event) => {
    adsManager = event.getAdsManager(contentPlayhead);
  }
);
```

**Anti-pattern — cached value:**
```javascript
// BAD: SDK always reads the same value → cue points never detected
let cachedTime = 0;
videoElement.addEventListener('timeupdate', () => { cachedTime = videoElement.currentTime; });
const contentPlayhead = { getCurrentTime: () => cachedTime }; // BAD
```

**Why the second fails:** `timeupdate` can fire up to ~250ms after the current frame. If the SDK queries `getCurrentTime()` right between two `timeupdate` events, it receives a value up to 250ms behind. With cue points close to the detection threshold, this can cause the SDK to skip them. Reading `videoElement.currentTime` directly eliminates this delay.

---

## 3. Video.js + IMA SDK

**Official plugin:** `videojs-ima` (Google, open source)

**Basic integration:**
```javascript
const player = videojs('video-element', { ... });

player.ima({
  adTagUrl: 'https://pubads.g.doubleclick.net/gampad/ads?...',
  debug: false,
  timeout: 5000,
  // For CTV: disable ad autoplay if policy requires
  autoPlayAdBreaks: true
});
```

**Relevant events for monitoring:**
```javascript
player.ima.addEventListener(google.ima.AdEvent.Type.AD_ERROR, (event) => {
  console.error('IMA error:', event.getError().getErrorCode());
  // Do not block playback — the plugin handles resume automatically
});

player.ima.addEventListener(google.ima.AdEvent.Type.ALL_ADS_COMPLETED, () => {
  // Optional: analytics
});
```

**Quirks on Tizen / webOS (WebView):**
- The plugin creates a `<div>` for the `AdDisplayContainer`. On some Tizen firmwares, `z-index` is not respected correctly → the ad renders behind the content. Solution: explicitly set `position: absolute` and `z-index: 9999` on the ads container.
- Memory leaks: if the user navigates between content without destroying the player, `videojs-ima` may retain references. Call `player.ima.destroy()` before `player.dispose()`.

---

## 4. JW Player: ad schedule and callbacks

JW Player has its own ad abstraction layer over the IMA SDK.

**Configuration with ad schedule (VMAP):**
```javascript
jwplayer('player').setup({
  file: 'https://cdn.example.com/content.m3u8',
  advertising: {
    client: 'googima',
    adscheduleid: 'pre_mid_post',
    schedule: {
      adbreak1: {
        offset: 'pre',
        tag: 'https://pubads.g.doubleclick.net/gampad/ads?...'
      },
      adbreak2: {
        offset: '00:10:00',
        tag: 'https://pubads.g.doubleclick.net/gampad/ads?...'
      }
    }
  }
});
```

**Alternative with Master Video Tag (GAM Ad Rules):**
```javascript
advertising: {
  client: 'googima',
  tag: 'https://pubads.g.doubleclick.net/gampad/ads?ad_rule=1&...'
  // GAM returns VMAP; JW Player lets IMA SDK manage all breaks
}
```

**Diagnostic callbacks:**
```javascript
jwplayer().on('adError', (event) => {
  console.log('Ad error:', event.message, event.tag);
});
jwplayer().on('adRequest', (event) => {
  console.log('Ad request fired:', event.tag);
});
jwplayer().on('adImpression', (event) => {
  console.log('Ad impression:', event.adtitle);
});
```

**JW Player limitation on CTV:** the web version of JW Player (HTML5) on Tizen/webOS has the same WebView behavior as Video.js. For Android TV/Fire TV, JW Player offers a separate native SDK that does not use IMA SDK directly — verify JW Player SDK for Android documentation if applicable.

---

## 5. Shaka Player: ad manager and integration

Shaka Player has a `ui.Overlay` and an experimental `ads.Client` for IMA SDK. It is the most used player on Android TV via ExoPlayer (in its Android variant), but also available in HTML5.

**HTML5 integration with IMA:**
```javascript
const video = document.getElementById('video');
const ui = new shaka.ui.Overlay(player, video.parentElement, video);
const adManager = player.getAdManager();

adManager.initClientSide(
  ui.getControls().getClientSideAdContainer(),
  video
);

const adRequest = new google.ima.AdsRequest();
adRequest.adTagUrl = 'https://pubads.g.doubleclick.net/gampad/ads?...';

adManager.requestClientSideAds(adRequest);
```

**Note:** Shaka's IMA integration is marked as experimental in some versions. Verify the exact Shaka version before assuming `getAdManager()` is available.

**Shaka + DAI (SSAI):**
```javascript
adManager.initServerSide(
  ui.getControls().getServerSideAdContainer(),
  video
);
const streamRequest = new google.ima.dai.api.VodStreamRequest();
streamRequest.contentSourceId = 'CONTENT_SOURCE_ID';
streamRequest.videoId = 'VIDEO_ID';
adManager.requestServerSideStream(streamRequest);
```

---

## 6. ExoPlayer (Android TV) + IMA extension

**Dependency:**
```gradle
implementation 'com.google.android.exoplayer:extension-ima:2.x.x'
```

**Integration:**
```kotlin
val imaAdsLoader = ImaAdsLoader.Builder(context)
    .setAdEventListener(adEventListener)
    .build()

val player = ExoPlayer.Builder(context).build()
val mediaSource = buildMediaSourceWithAds(
    contentUri,
    Uri.parse("https://pubads.g.doubleclick.net/gampad/ads?..."),
    imaAdsLoader,
    playerView
)

player.setMediaSource(mediaSource)
player.prepare()
```

**Correct destroy:**
```kotlin
override fun onDestroy() {
    imaAdsLoader.release()  // release IMA before the player
    player.release()
    super.onDestroy()
}
```

**Android TV quirks:**
- `ImaAdsLoader` manages `ContentPlayhead` internally when integrated with ExoPlayer — no need to implement it manually.
- On Fire TV (Fire OS), the integration is identical. Only the device ID retrieval mechanism differs (AFAI vs GAID).
- `AD_ID` permission in `AndroidManifest.xml` is mandatory to receive the GAID. Without it, the field arrives empty → low programmatic fill.

---

## 7. AVPlayer (tvOS) + IMA iOS SDK

**Apple constraint:** on tvOS you cannot use a custom player for DRM content or in App Store apps. AVPlayer is mandatory.

**Basic integration:**
```swift
let adsLoader = IMAAdsLoader(settings: nil)
adsLoader.delegate = self

let adDisplayContainer = IMAAdDisplayContainer(
    adContainer: self.view,
    viewController: self
)

let request = IMAAVPlayerContentPlayhead(avPlayer: player)
// IMAAVPlayerContentPlayhead implements ContentPlayhead automatically
// reading avPlayer.currentTime() directly — no need to implement it manually

let adsRequest = IMAAdsRequest(
    adTagUrl: "https://pubads.g.doubleclick.net/gampad/ads?...",
    adDisplayContainer: adDisplayContainer,
    contentPlayhead: request,
    userContext: nil
)
adsLoader.requestAds(with: adsRequest)
```

**ATT (App Tracking Transparency) on tvOS 14.5+:**
```swift
ATTrackingManager.requestTrackingAuthorization { status in
    let idfa = status == .authorized
        ? ASIdentifierManager.shared().advertisingIdentifier.uuidString
        : "00000000-0000-0000-0000-000000000000"
    // Pass idfa as rdid in the ad tag URL
}
```
If ATT is not requested or the user denies → IDFA is all zeros → programmatic without targeting → low CPM. This is a monetization prerequisite, not optional.

---

## 8. hls.js / dash.js in web-based CTV environments

hls.js and dash.js are low-level parsers/players frequently used on Tizen and webOS when the publisher builds their own player instead of using Video.js/Shaka.

**hls.js + IMA SDK (manual pattern):**
```javascript
const hls = new Hls();
hls.loadSource('https://cdn.example.com/content.m3u8');
hls.attachMedia(videoElement);

// IMA SDK initializes in parallel, on the same videoElement
const adDisplayContainer = new google.ima.AdDisplayContainer(adContainer, videoElement);
const adsLoader = new google.ima.AdsLoader(adDisplayContainer);

// ContentPlayhead: read currentTime directly (see §2)
const contentPlayhead = { getCurrentTime: () => videoElement.currentTime };
```

**Important:** hls.js does not manage ad cue points — that is the IMA SDK's responsibility. hls.js manages content streaming; IMA SDK detects when to pause it for an ad break.

**dash.js is less common on CTV** because DASH is not the dominant format on Tizen/webOS (they prefer HLS). Relevant mainly on Android TV when building a web-based player (rare case).

---

## 9. Player + SDK integration anti-patterns

| Anti-pattern | Symptom | Cause | Solution |
|---|---|---|---|
| `ContentPlayhead` returns cached value | Mid-rolls never fire or fire late | `getCurrentTime()` does not read from the player in real time | Read `videoElement.currentTime` directly [Probable] |
| Creating `AdDisplayContainer` after `requestAds` | SDK initialization error; ads do not load | SDK needs the container before any request | Create container in `init()`, always before any request [Verified: IMA SDK docs] |
| Not calling `adsManager.destroy()` when changing content | Ads from previous content interfere; undefined behavior | AdsManager retains state and listeners | `destroy()` mandatory before new `AdsLoader.requestAds()` [Verified] |
| Player tries to parse VAST or build tracking URLs | Duplicated logic, inconsistent tracking | Confusion of responsibilities | Player only manages playback and `ContentPlayhead`; everything else is the SDK's |
| `muted` not declared on autoplay in WebView (Tizen/webOS) | WebView blocks ad playback | Chromium engine autoplay policy in WebView | `vpa=auto&vpmute=1` if muted; or manage autoplay permission explicitly |
| Not implementing `ContentPlayhead` in manual integration | Mid-rolls never fire | SDK cannot detect player position | Always implement when using IMA SDK without ExoPlayer/AVPlayer |
| Blocking content on `AD_ERROR` waiting for recovery | Blank screen; user blocked | App waits for an ad that will not arrive | SDK auto-resumes; app only logs and never blocks playback [Verified] |
| `AdsLoader` instantiated multiple times per session | Memory leaks, unpredictable behavior | Reuse the same `AdsLoader` for multiple requests | One `AdsLoader` per player session; multiple `requestAds()` on the same loader |
