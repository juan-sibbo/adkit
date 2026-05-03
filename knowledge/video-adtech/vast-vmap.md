# VAST / VMAP — Technical reference

## Table of contents
1. VAST versions and compatibility
2. VAST InLine vs Wrapper structure
3. VAST error codes with CTV cause/solution
4. VMAP: structure and timeOffset
5. Wrapper chains and SSU
6. MediaFile selection
7. Tracking macros

---

## 1. VAST versions and compatibility

| Version | Key nodes added | CTV support |
|---|---|---|
| VAST 2.0 | Basic InLine/Wrapper structure, TrackingEvents | Universal, legacy |
| VAST 3.0 | AdVerifications (partial), skippable ads, icons | Broad |
| VAST 4.0 | AdVerifications (full, OMID), Mezzanine file, UniversalAdId, Category | Recommended for CTV |
| VAST 4.1 | Interactive Creative File improvements, server-side stitching signals | Growing support |
| VAST 4.2 | Minor improvements, clarifications | Emerging support |

**For CTV:** use `output=xml_vast4` in GAM. If the player is legacy, verify compatibility and use `output=xml_vast3` as fallback.

---

## 2. VAST InLine vs Wrapper structure

**InLine:** contains the complete ad (MediaFiles, TrackingEvents, creative).

**Wrapper:** contains a URL to another VAST (redirect). Each level adds latency (200-500ms per hop).

```xml
<!-- Wrapper example -->
<VAST version="4.0">
  <Ad>
    <Wrapper>
      <AdSystem>SSP_A</AdSystem>
      <VASTAdTagURI><![CDATA[https://ssp-b.com/vast?id=456]]></VASTAdTagURI>
      <Impression><![CDATA[https://ssp-a.com/impression]]></Impression>
      <e><![CDATA[https://ssp-a.com/error?code=[ERRORCODE]]]></e>
    </Wrapper>
  </Ad>
</VAST>
```

**Wrapper limit:** the IAB VAST spec recommends players support at least 5 levels. GAM configures `maxRedirects`. Beyond that → error 302 (wrapper limit reached).

---

## 3. VAST error codes — cause and solution in CTV

| Code | Name | Common CTV cause | Diagnosis | Solution |
|---|---|---|---|---|
| 100 | XML Parsing Error | SSP returns invalid XML | Validate VAST with VSI | Contact SSP; enable SSU (GAM validates XML server-side) |
| 101 | VAST Schema Error | Elements from incorrect version mixed | Inspect VAST response | Align VAST version in GAM and SSP |
| 200 | Trafficking Error | Misconfigured line item | GAM Publisher Console | Review targeting and creative assignment |
| 201 | Video Player Error | Unsupported codec (HEVC on legacy device) | Verify MediaFile codec vs device capabilities | Use universal H.264; 720p fallback |
| 202 | Ad Request Timeout | Network latency >timeout, slow Open Bidding | Measure time-to-first-byte of ad request | Reduce timeout; reduce SSPs in Open Bidding |
| 203 | Wrapper Timeout | Multiple redirects exceed time | Count wrappers in chain | Enable SSU; limit maxRedirects |
| 301 | Wrapper Limit | Exceeds configured maximum redirects | Count wrapper levels | Reduce chain; SSU |
| 302 | No VAST After Wrapper | Wrapper chain ends without InLine | Manually fetch each URL in the chain | Verify final endpoint; contact SSP |
| 303 | No Ad In VAST | Empty VAST (no-fill or WTA compliance) | Publisher Console: see no-match reasons | Verify WTA, non-personalized creatives available |
| 400 | General Linear Error | Problem with MediaFile | See 4xx subcodes | — |
| 401 | MediaFile Not Found | Creative deleted from CDN | curl -I of MediaFile URL | Re-upload creative |
| 402 | MediaFile Timeout | Slow CDN or very large creative | Measure size and download time | Optimize bitrate; use CDN with local edge servers |
| 403 | MediaFile Access Denied | Signed URL expired or geo-blocking | Verify response headers | Verify URL signing and CDN permissions |
| 405 | Unsupported MIME | WebM on device that only supports MP4 | Verify MediaFile MIME type | Use universal MP4/H.264 |
| 900 | Undefined Error | Catch-all | Review full SDK logs | Stack trace, error context |
| 901 | VPAID Error | VPAID creative on CTV | Verify creative type | VPAID is blocked on CTV; use linear VAST |

---

## 4. VMAP: structure and timeOffset

VMAP defines the ad break structure for a full content item.

```xml
<vmap:VMAP xmlns:vmap="http://www.iab.net/videosuite/vmap" version="1.0">
  <vmap:AdBreak timeOffset="start" breakType="linear">
    <vmap:AdSource><vmap:VASTAdData>...</vmap:VASTAdData></vmap:AdSource>
  </vmap:AdBreak>
  <vmap:AdBreak timeOffset="00:15:00.000" breakType="linear">
    <vmap:AdSource><vmap:AdTagURI><![CDATA[https://...]]></vmap:AdTagURI></vmap:AdSource>
  </vmap:AdBreak>
  <vmap:AdBreak timeOffset="end" breakType="linear">
    <vmap:AdSource>...</vmap:AdSource>
  </vmap:AdBreak>
</vmap:VMAP>
```

**timeOffset:** `start` | `end` | `HH:MM:SS.mmm` | `#N` (percentage position, rarely used).

**Relationship with Ad Rules:** when GAM has Ad Rules configured and `ad_rule=1` in the tag, GAM returns VMAP with the defined ad breaks. The IMA SDK parses VMAP automatically.

---

## 5. Wrapper chains and SSU

**Without SSU (client-side unwrapping):**
Device makes N sequential requests to resolve the wrapper chain.
Cumulative latency: N × (200-500ms).

**With SSU (Server-Side Unwrapping):**
GAM resolves the chain server-side and returns the final InLine VAST to the device.
Device makes just 1 request.

**Verified SSU capabilities:**
- Resolves wrappers up to the configured limit [Verified]
- Inspects chain for brand safety (blocks suspicious domains) [Probable]
- Requires the third-party ad server to be on the certified servers list [Verified]
- Configuration: Admin > Video > Server-side ad unwrapping

**Limitation:** SSU cannot resolve wrappers from ad servers not certified by Google.

---

## 6. MediaFile selection

The IMA SDK selects MediaFile based on:
1. **Codec compatibility** of the device (H.264 preferred, HEVC if supported)
2. **Detected bandwidth** (adaptive)
3. **Device screen resolution**
4. **Supported MIME type**

For CTV: always include MediaFile H.264/MP4 in 1080p and 720p as a minimum.

---

## 7. Tracking macros

| Macro | Substitution | Who substitutes |
|---|---|---|
| `[ERRORCODE]` | VAST error code | Player/SDK |
| `[CACHEBUSTING]` | Random value | Player/SDK |
| `[TIMESTAMP]` | Epoch timestamp | Ad server or player |
| `[CONTENTPLAYHEAD]` | Content position (HH:MM:SS.mmm) | Player |
| `[ASSETURI]` | URI of the MediaFile being played | Player/SDK |

**In GAM:** GAM macros (such as `%%CLICK_URL_ESC%%`) are substituted server-side. Standard VAST macros are substituted by the SDK/player client-side.
