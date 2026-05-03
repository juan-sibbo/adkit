# CTV Environments — Technical reference

## Table of contents
1. Tier 1/2/3 framework by device
2. Platforms and their specifics
3. HbbTV: profiles and limitations
4. FAST channels
5. CORS/CSP on CTV
6. App versioning and device tiers
7. localStorage and persistence

---

## 1. Tier 1/2/3 framework by device

| Tier | Characteristics | Devices | Ads strategy |
|---|---|---|---|
| **Tier 1** (Modern, 2020+) | Full SDK, OM, HEVC, 4K, good CPU/RAM | Samsung Tizen 5.5+, LG webOS 5.0+, Android TV 10+, Apple TV 4K, Fire TV Stick 4K | Full features: IMA SDK, Open Bidding 5 SSPs, OMID, 1080p/4K |
| **Tier 2** (Mid-range, 2017-2019) | SDK supported, H.264, 1080p, moderate CPU/RAM | Samsung Tizen 3.0-5.0, LG webOS 3.5-4.5, Android TV 8-9, Fire TV Gen 2 | Reduced: Open Bidding 3 SSPs, OMID, 1080p |
| **Tier 3** (Legacy, pre-2017) | Limited or no SDK support, H.264 720p, low CPU/RAM | Samsung Tizen 2.4, LG webOS 2.0, operator STBs | Minimum: direct or 1 SSP, Inferred Viewability, 720p, 3s timeout |

**Tier 3 support decision:** business trade-off. If revenue < maintenance cost → deprecate with notice.

---

## 2. Platforms and their specifics

### Samsung Tizen
- **Runtime:** WebView (JavaScript) for web apps; native C++ for native Tizen apps
- **IMA SDK:** HTML5 variant
- **Device ID:** TIFA (Tizen Identifier for Advertising) via `webapis.adinfo.getTIFA()`
- **idtype:** `tifa` [Verified]
- **LAT:** `webapis.adinfo.isLATEnabled()`
- **Specifics:** frequent WebView memory leaks; `localStorage` available but with limits; HEVC only on 2017+ models
- **European market share:** ~30-35% (leader in ES, FR)

### LG webOS
- **Runtime:** WebView (JavaScript) + webOS Media API
- **IMA SDK:** HTML5 variant
- **Device ID:** LGUDID (LG Advertising ID) via proprietary LG API
- **idtype:** `lgudid` [Probable]
- **Specifics:** proprietary LG ads API; `localStorage` available; Web Inspector for debugging
- **European market share:** ~15-20%

### Android TV
- **Runtime:** native Android (Java/Kotlin)
- **IMA SDK:** Android SDK
- **Device ID:** GAID via `AdvertisingIdClient.getAdvertisingIdInfo()`
- **idtype:** `adid` [Verified]
- **Specifics:** ExoPlayer recommended; requires `com.google.android.gms.permission.AD_ID` permission; Leanback UI
- **European market share:** ~15% (growing with Chromecast with Google TV)

### Fire TV (Amazon)
- **Runtime:** Android fork (Fire OS)
- **IMA SDK:** Android SDK (compatible)
- **Device ID:** AFAI via `AdRegistration.getAdvertisingId()`
- **idtype:** `afai` [Probable]
- **Specifics:** Fire OS is an Android fork but with the Amazon app store; debugging similar to Android (adb)
- **European market share:** ~10-12%

### Apple TV (tvOS)
- **Runtime:** native tvOS (Swift/ObjC)
- **IMA SDK:** iOS SDK
- **Device ID:** IDFA via `ASIdentifierManager`; requires ATT framework on tvOS 14.5+
- **idtype:** `idfa` [Verified]
- **Specifics:** AVPlayer mandatory; no HLS interstitials for ads; HEVC supported
- **European market share:** ~5-8%

### Roku
- **Runtime:** BrightScript (proprietary)
- **IMA SDK:** Roku variant
- **Device ID:** RIDA (Roku ID for Advertising) via Roku API
- **idtype:** `rida` [Verified]
- **Specifics:** closed environment; proprietary SDK; main market US/UK
- **European market share:** <5%

---

## 3. HbbTV: profiles and limitations

| Profile | Year | Ad capabilities |
|---|---|---|
| HbbTV 1.4 | 2014 | Basic VAST, no OMID, limited JavaScript |
| HbbTV 2.0 | 2016 | Better video support, DRM, WebSocket |
| HbbTV 3.0 | 2021 | Companion screen, better programmatic integration |

**Common HbbTV limitations:**
- Very limited CPU/RAM (runs on the TV, not on a STB)
- No resettable device ID in many implementations (depends on broadcaster)
- CORS can be problematic (variable security restrictions)
- `localStorage` limited or unavailable on older profiles
- Prebid.js IS used in some HbbTV cases (it is a JavaScript environment)
- Recommended timeout: 3s maximum

---

## 4. FAST channels (Free Ad-Supported Television)

- 24/7 scheduled content (similar to linear TV)
- Synchronous ad breaks, controlled by the broadcaster/platform
- Same rules as live TV: fill is critical, fallback mandatory
- Examples: Pluto TV, Samsung TV+, Rakuten TV, Amazon Freevee
- SSAI generally preferred over CSAI for FAST (seamless UX)

---

## 5. CORS/CSP on CTV

**CORS:** CTV apps make requests to ad serving domains (pubads.g.doubleclick.net, SSP tracking domains). If CORS is not configured:
- Tracking pixels may fail silently
- VAST fetch may be blocked

**CSP (Content Security Policy):** some CTV platforms impose restrictive CSPs.
- Verify that ad serving domains are allowed
- Include domains for: GAM, active SSPs, creative CDNs, tracking providers

**Diagnosis:** if beacons are not firing → verify CORS headers in the tracking endpoint response.

---

## 6. App versioning and device tiers

**Real problem:** not all devices have the same version of the app.
- Gradual rollout: v2.0 on 30% of devices, v1.9 on 70%
- Feature flags to enable/disable ad features by version
- `app_version` as cust_param → line item targeting by version

**Tier detection in code:**
```
if (device.year < 2017 || device.ram < 1GB) → Tier 3 config
else if (device.year < 2020) → Tier 2 config
else → Tier 1 config
```

---

## 7. localStorage and persistence

**Do not assume `localStorage` is available on CTV.**

| Platform | localStorage | Alternative |
|---|---|---|
| Samsung Tizen | ✅ Available (with limits) | — |
| LG webOS | ✅ Available | — |
| Android TV | N/A (native uses SharedPreferences) | SharedPreferences |
| Fire TV | N/A (native uses SharedPreferences) | SharedPreferences |
| Apple TV | N/A (native uses UserDefaults) | UserDefaults |
| HbbTV | ⚠️ Variable (depends on profile/implementation) | May not be available |

**Anti-pattern:** assuming `localStorage` is available for storing user IDs or configuration → fails silently in some CTV environments.
