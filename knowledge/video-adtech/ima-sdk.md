# IMA SDK — Technical reference

## Table of contents
1. SDK variants by platform
2. IMA SDK vs PAL SDK vs SDK-less (PAI) comparison
3. SDK lifecycle
4. Player / SDK / App responsibility table
5. Interface contracts
6. WTA (Where The Ads) on CTV
7. PAL SDK: nonce and programmatic signals
8. Error handling and fallback strategy
9. Key configurations

---

## 1. SDK variants by platform

| Platform | SDK | Language | CTV notes |
|---|---|---|---|
| Android TV | IMA Android SDK | Java/Kotlin | ExoPlayer recommended; uses Google Play Services for GAID |
| tvOS / Apple TV | IMA iOS SDK | Swift/ObjC | AVPlayer mandatory; requires ATT framework for IDFA on iOS 14.5+ |
| Tizen (Samsung) | IMA HTML5 SDK | JavaScript | Runs in WebView; TIFA access via `webapis.adinfo.getTIFA()` |
| webOS (LG) | IMA HTML5 SDK | JavaScript | WebView + webOS media API; LGUDID via proprietary LG API |
| Fire TV | IMA Android SDK | Java/Kotlin | Android fork; AFAI via `AdRegistration` API |
| Roku | IMA Roku SDK | BrightScript | Proprietary environment; RIDA via Roku API |
| Cast (Chromecast) | IMA CAF SDK | JavaScript | Receiver app; inherits device info from cast device |
| Web / HbbTV | IMA HTML5 SDK | JavaScript | HbbTV: profiles 1.4/2.0/3.0 with variable limitations |

---

## 2. IMA SDK vs PAL SDK vs SDK-less (PAI) comparison

| Aspect | IMA SDK | PAL SDK | SDK-less (PAI) |
|---|---|---|---|
| Ad request | ✅ Full | ❌ No | ❌ No (server-side) |
| VAST parsing | ✅ Automatic | ❌ No | ❌ Manual |
| Ad rendering | ✅ Automatic | ❌ No | ❌ Manual |
| Tracking pixels | ✅ Automatic | ❌ No | ❌ Manual |
| Programmatic signals | ✅ Automatic | ✅ Via nonce | ⚠️ Limited |
| Open Measurement | ✅ Built-in | ⚠️ Limited | ❌ No |
| Google Ads demand | ✅ Yes | ❌ AdX/AB only | ❌ AdX/AB only |
| Integration complexity | Low | Medium | High |
| Device support | Broad | Broad | Legacy without SDK |

**Decision tree:**
- Standard player (ExoPlayer, AVPlayer) → **IMA SDK**
- Custom player with proprietary ad logic → **PAL SDK** (captures signals without modifying the player)
- Device without SDK support (legacy STB, old consoles) → **SDK-less/PAI** (last resort)

---

## 3. SDK lifecycle

```
1. Create ImaSdkSettings (once per app)
2. Create AdDisplayContainer (once per player session)
3. Create AdsLoader (once per player session)
4. Build AdsRequest with ad tag URL
5. adsLoader.requestAds(adsRequest)
6. onAdsLoaded → get AdsManager
7. adsManager.init()
8. adsManager.start() → pre-roll starts
9. SDK emits events: CONTENT_PAUSE_REQUESTED, STARTED, quartiles, COMPLETED, CONTENT_RESUME_REQUESTED
10. ALL_ADS_COMPLETED → entire pod finished
11. On content change: adsManager.destroy()
12. Repeat from step 4 for new content
```

**Critical rule:** Always `destroy()` before creating a new AdsManager. Reusing without destroy → undefined behavior [Verified].

---

## 4. Responsibility table

| Function | Content Player | IMA SDK | App |
|---|---|---|---|
| Play content | ✅ | ❌ | ❌ |
| Report playhead position | ✅ (every ~250ms) | Consumes | ❌ |
| Request ads (ad request) | ❌ | ✅ | ❌ |
| Parse VAST | ❌ | ✅ | ❌ |
| Play creatives | ❌ | ✅ | ❌ |
| Fire tracking pixels | ❌ | ✅ (automatic) | ❌ |
| Detect cue points | ❌ | ✅ (monitors playhead) | ❌ |
| Pause/resume content | ✅ (executes) | Requests via event | Coordinates |
| Ad error handling | ❌ | ✅ (auto-resume) | Log + monitoring |
| Show "Ad X of Y" UI | ❌ | ❌ | ✅ |

**Anti-pattern:** the player attempting to manage VAST URLs, parse wrappers, or fire quartiles. That is the SDK's responsibility.

---

## 5. Interface contracts

**Player → SDK:** `ContentPlayhead` interface
- `getCurrentTime()` → position in seconds (SDK calls ~4x/s to detect cue points)
- `getDuration()` → total content duration

**SDK → App:** events
- `CONTENT_PAUSE_REQUESTED` → app pauses content player
- `CONTENT_RESUME_REQUESTED` → app resumes content player
- `STARTED`, `FIRST_QUARTILE`, `MIDPOINT`, `THIRD_QUARTILE`, `COMPLETED` → informational
- `AD_ERROR` → SDK already handled; app only logs
- `ALL_ADS_COMPLETED` → full pod finished

**Fundamental rule:** NEVER block content on ad error. If ad fails → log, analytics, automatic content resume.

---

## 6. WTA (Where The Ads) on CTV

WTA = "Why This Ad" / advertising transparency.

**Behavior:**
- If the GAM network has WTA compliance active, GAM requires ads to be able to show transparency information
- On CTV, many user agents do not support WTA overlay rendering
- IMA SDK may automatically set `wta=0` based on the user agent [Probable]
- If `wta=0` and there are no non-personalized creatives available → GAM may return an empty VAST (error 303) [Probable]

**Diagnosis:**
1. Intercept the ad request → verify if `wta=0` is present
2. Publisher Console → verify if there are eligible creatives
3. If the problem is WTA → verify WTA compliance configuration in GAM

**Mitigation:** ensure non-personalized creatives are available as fallback; or verify WTA configuration for the CTV inventory type.

---

## 7. PAL SDK: nonce and programmatic signals

**What PAL does:**
- Generates a nonce (cryptographic token) that encapsulates environment signals
- The nonce is passed in the ad tag as the `givn=` parameter
- GAM uses the nonce to validate the environment and improve programmatic signals

**Nonce granularity:**
- **New nonce per stream/content** [Verified in PAL docs]
- Reusable for multiple ad requests within the same stream
- Reusing across different streams → incorrect signals [Probable: may affect invalid traffic detection]

**Supported PAL platforms:** HTML5, tvOS, iOS, Android, Cast, Roku, Samsung Tizen

**PAL limitation:** does not have access to Google Ads demand (only AdX, Authorized Buyers). If Google Ads demand is needed → use IMA SDK.

---

## 8. Error handling and fallback strategy

**Recommended fallback waterfall:**

```
1. Main ad request (GAM with Open Bidding)
   └─ Fails (timeout, VAST error, no-fill)
   ↓
2. Fallback tag (direct GAM, without Open Bidding — lower latency)
   └─ Fails
   ↓
3. House ad from GAM (priority 16, network line item)
   └─ Fails
   ↓
4. Local house ad (asset bundled in app — instant, no network)
   └─ Fails
   ↓
5. Resume content (never blank screen)
```

**Recommended timeouts (heuristic, measure in each environment):**
- General CTV: 5s
- Live TV: 2-3s (viewer less tolerant)
- Legacy devices: 3s (limited CPU/network)

---

## 9. Key configurations

| Setting | Recommended value | Reason |
|---|---|---|
| `vastLoadTimeout` | 5000ms (CTV), 3000ms (live) | Balance between fill and UX |
| `maxRedirects` | 3-5 | Limit wrapper chain latency |
| `language` | ISO 639-1 of the content | Ad UI localization |
| `autoPlayAdBreaks` | true (default) | SDK manages ad breaks automatically |
