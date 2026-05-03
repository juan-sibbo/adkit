# SSAI / SGAI / DAI — Technical reference

## Table of contents
1. Types of server-side ad insertion
2. Beaconing: server-side vs client-side
3. SCTE-35 and ad break signaling
4. Live vs VOD: operational differences
5. Gap management and black screens
6. Google DAI: SDK types
7. SSAI problem diagnosis

---

## 1. Types of server-side ad insertion

| Type | Description | Who stitches | Ad break control |
|---|---|---|---|
| **SSAI (Server-Side Ad Insertion)** | Ads stitched into the manifest/stream before reaching the device | SSAI provider (Yospace, AWS Elemental, etc.) | Publisher/ad server |
| **SGAI (Server-Guided Ad Insertion)** | Server prepares the ad decision but the player executes the insertion | Server + player combination | Server guides, player executes |
| **DAI (Dynamic Ad Insertion — Google)** | Google service that manages stitching + tracking integrated with GAM | Google servers | GAM Ad Rules |

**When to use each:**
- **CSAI (client-side)**: apps with IMA SDK, no server-side infrastructure
- **SSAI**: high-volume live/FAST, devices without SDK support, seamless UX
- **Google DAI**: full GAM ecosystem, willing to use Google's DAI SDK

---

## 2. Beaconing: server-side vs client-side

**Client-side beaconing:**
- Player/SDK fires tracking pixels directly from the device
- Requires the player/SDK to know when the ad is playing
- **Problem in SSAI:** if the ad is stitched into the manifest, the player plays a continuous stream without knowing where ads start/end → beacons do not fire [Verified: by architectural definition]

**Server-side beaconing:**
- The SSAI provider fires tracking pixels from its servers
- The provider knows when each ad segment was served
- Quartiles calculated by the ad's temporal position in the stream

**Beaconing with DAI SDK:**
- Google DAI SDK integrates in the player
- SDK receives stream metadata indicating ad positions
- SDK fires beacons client-side with server data
- Hybrid: client-side precision with server information

**Anti-pattern:** configuring client-side beaconing when using SSAI without SDK → beacons do not fire.

---

## 3. SCTE-35 and ad break signaling

**SCTE-35:** ANSI standard for ad insertion opportunity signaling in broadcast/cable streams [Verified].

**Relevant SCTE-35 message types:**
- `splice_insert`: marks start/end of ad break
- `time_signal` + segmentation descriptors: signals break type (content, ad, chapter)

**In HLS:** SCTE-35 is carried as `#EXT-X-DATERANGE` or `#EXT-X-CUE-OUT` / `#EXT-X-CUE-IN` tags.

**In DASH:** SCTE-35 is carried in `EventStream` elements within the MPD.

**Diagnosing missing SCTE-35:**
- Symptom: ad breaks not inserted in live stream
- Verify: stream manifest → look for SCTE-35 markers
- If missing: upstream problem (encoder/transcoder not inserting markers)

---

## 4. Live vs VOD: operational differences

| Aspect | Live/FAST | VOD |
|---|---|---|
| Ad breaks | Synchronous, marked by SCTE-35 or EPG | Dynamic, based on cue points or ad rules |
| Latency tolerance | Very low (≤2-3s) | Moderate (≤5s) |
| Fill priority | Critical (no gap/silence allowed) | Flexible (can resume without ad) |
| Fallback | House ads mandatory | Resuming content acceptable |
| Manifest | Continuous, real-time segments | Static/semi-static, pre-generated |
| Pod composition | Fixed (duration determined by SCTE-35) | Dynamic (GAM can adjust) |

---

## 5. Gap management and black screens

**Causes of black screen in SSAI:**
1. No-fill: no ad for the slot → gap in the stream
2. Timeout: ad decision too slow for live
3. Stitching failure: SSAI provider cannot stitch the ad in time
4. Manifest corruption: ad segments not correctly inserted

**Mitigations:**
- Slates/bumpers: filler content (channel logo, "we'll be right back")
- House ads: always have available as fallback
- Fixed-duration slate as backup (3-5s of branded content)
- Aggressive timeout in SSAI provider (response deadline before the ad break)

**In VOD:** less critical — can resume content without ad.
**In Live/FAST:** critical — gap = "dead air" = worst possible UX.

---

## 6. Google DAI: SDK types

| Type | Platform | Use |
|---|---|---|
| DAI HTML5 SDK | Web, HbbTV, Tizen, webOS | JavaScript environments |
| DAI Android SDK | Android TV, Fire TV | Native Android apps |
| DAI iOS SDK | tvOS, Apple TV | Native iOS apps |
| DAI Cast SDK | Chromecast | Receiver apps |

**DAI vs IMA SDK (CSAI):**
- IMA SDK = CSAI (client decides and renders ads, separate from content)
- DAI SDK = SDK-assisted SSAI (stitched stream, SDK tracks)

---

## 7. SSAI problem diagnosis

| Symptom | Check | Probable cause |
|---|---|---|
| Ads not inserted | Stream manifest → are there ad segments? | SSAI provider not receiving ad response in time |
| Beacons not firing | Is beaconing server-side or client-side? | Client-side beaconing in SSAI without SDK |
| Black screen during ad break | Manifest → gap between content and ad segments? | No-fill + no slate/house ad |
| Ad cuts off halfway | Ad duration vs slot duration in manifest | Slot shorter than ad; transcoding issue |
| Audio desync | Ad codec vs content stream codec | Sample rate or audio codec mismatch |
| High latency in ad break | Time between SCTE-35 and first ad frame | SSAI provider slow to resolve + stitch |
