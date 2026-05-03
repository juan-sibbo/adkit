# Anti-patterns — Extended reference

Complements the top-7 in the core SKILL.md with additional anti-patterns by layer and platform.

---

## IMA SDK anti-patterns

| Anti-pattern | Symptom | Cause | Solution |
|---|---|---|---|
| AdsManager reused without destroy() | Ads do not play, events do not fire | State leak from previous AdsManager | `destroy()` before creating new AdsManager [Verified] |
| AdDisplayContainer created after requestAds | SDK initialization error | SDK needs container before request | Create container on init, before any request [Probable] |
| muted not declared on autoplay (Tizen/webOS) | Browser/WebView blocks playback | Autoplay policy of Chrome/Safari engine | Declare `vpa=auto&vpmute=0` or `vpmute=1` if muted [Verified for web-based CTV] |
| PAL nonce reused across streams | Incorrect programmatic signals | Nonce encapsulates stream-specific info | New nonce per stream; reusable within the same stream [Probable] |
| Player tries to parse VAST/build tracking URLs | Duplicated logic, inconsistent tracking | Confusion of responsibilities between Player and SDK | Let the SDK manage all ad logic |
| Not implementing ContentPlayhead interface / implementing it with a cached value | Mid-rolls never fire or fire late | SDK queries `getCurrentTime()` ~4x/s; if it returns a static or cached value, cue points are not detected | Implement `getCurrentTime()` reading `videoElement.currentTime` directly, not a value updated by `timeupdate` [Probable] |
| Blocking content on ad error | Blank screen, frustrated user | App waits for an ad that never arrives | ALWAYS resume content on error; SDK auto-resumes [Verified] |

---

## VAST anti-patterns

| Anti-pattern | Symptom | Cause | Solution |
|---|---|---|---|
| Wrapper chains >5 levels | Timeout, error 302/303 | Excessive redirect chain | Enable SSU; limit maxRedirects to 3-5 [Verified: IAB spec] |
| Missing correlator | Ads do not rotate, competitive exclusion does not work | Without correlator, GAM cannot deduplicate | Generate correlator (timestamp) at content start [Verified] |
| Correlator reused across content | Incorrect competitive exclusion across content | Same correlator = GAM thinks it is the same page view | New correlator per content |
| VPAID creative on CTV | Error 901, creative does not render | VPAID blocked on CTV | VAST linear only; verify creative type before serving |
| MediaFile HEVC only | Playback fails on legacy devices | HEVC not supported on Tizen <2017, etc. | Always include H.264 fallback |

---

## SSAI anti-patterns

| Anti-pattern | Symptom | Cause | Solution |
|---|---|---|---|
| Client-side beaconing in SSAI without SDK | Beacons do not fire | Player does not know where the ads are in the stitched stream | Server-side beaconing or integrate DAI SDK [Verified] |
| SCTE-35 without duration | Ad breaks without timing control | Marker signals start but not duration | Include DURATION in SCTE-35 splice_insert |
| Ad creative not transcoded to stream format | Glitches, black frames, audio desync | Codec/bitrate/sample rate mismatch | SSAI provider must transcode to stream format |
| No house ads/slates on live | Dead air during no-fill | No fallback when there is no demand | Always have slates/house ads available for live |

---

## Signals and identity anti-patterns

| Anti-pattern | Symptom | Cause | Solution |
|---|---|---|---|
| rdid passed as PPID | Semantic confusion, incorrect targeting | Different fields: device vs publisher ID | Keep separate [Verified] |
| idtype hardcoded without platform detection | Incorrect idtype on some platforms | Code does not differentiate Samsung vs Fire TV vs Android TV | Detect platform dynamically |
| is_lat always 0 (hardcoded) | Privacy violation, DSPs may block | App ignores user settings | Read actual value from OS API |
| Device ID cached indefinitely | Stale rdid after user resets it | App does not refresh the ID | Get device ID fresh each session or at least every X hours |
| cust_params with double encoding | GAM does not parse key-values | `%253D` instead of `%3D` | Encode only once |
| cust_params >2000 chars | URL truncated, params lost | Too many key-values or long values | Prioritize params; use Content ID instead of passing everything |
| Consent string cached without refresh | Expired or user-changed consent | App does not re-query CMP | Re-fetch consent on each ad request or at least every 24h |
| app-ads.txt not published or invalid | DSPs block inventory | Missing authorization verification | Publish and validate with IAB crawler |

---

## Platform-specific

### Samsung Tizen
| Anti-pattern | Symptom | Cause |
|---|---|---|
| Assuming >1GB RAM available on Tizen 2.4 | OOM crash during ad playback | Tizen 2.4 has ~512MB total; IMA+OM+creative buffer can exceed it |
| Not disabling OMID on Tier 3 | Degraded performance | OM SDK consumes significant CPU/RAM |
| Unmanaged WebView memory leaks | App becomes slow after N ad breaks | WebView accumulates memory; perform periodic cleanup |

### LG webOS
| Anti-pattern | Symptom | Cause |
|---|---|---|
| Using webOS APIs without checking version | API call fails silently | APIs change between webOS versions |

### HbbTV
| Anti-pattern | Symptom | Cause |
|---|---|---|
| Assuming localStorage is available | User ID and config fail silently | Not all HbbTV profiles support localStorage |
| Timeout >5s | Viewer frustrated, broadcaster rejects | CPU/RAM very limited on HbbTV |
| Many SSPs in Open Bidding | Excessive latency | Resources very limited; limit to 1-2 SSPs |

---

## Diagnosis by symptom

| Symptom | First hypothesis | Second hypothesis | Third hypothesis |
|---|---|---|---|
| No-fill (0 impressions) | App not registered/claimed in GAM | Invalid app-ads.txt | Incorrect msid/store ID |
| Low fill (<40%) | Incomplete signals (rdid, consent) | Floors too high | Over-segmentation of content bundles |
| High error rate (>5%) | Excessive wrapper chains | Creative CDN issues | Incompatible creative codec |
| High latency (>5s) | Open Bidding with too many SSPs | Remote creative CDN | SSU not enabled |
| Low VCR (<80%) | Creative buffering (excessive bitrate) | Defective VAST wrapper | Audio/video desync |
| Mid-rolls not firing | `ContentPlayhead.getCurrentTime()` returns a cached value or is not implemented | Ad Rules misconfigured or cue points missing in MRSS | Player paused or `AdsManager` not started |
| Black screen | No-fill without fallback/house ads | SSAI stitching failure | SCTE-35 markers missing |
| Discrepancy >15% | SSP IVT filter | Timezone mismatch in reports | Tracking pixel blocked |
