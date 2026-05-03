# HLS / DASH — Technical reference

## 1. Manifests and ad insertion points

### HLS (HTTP Live Streaming)
- Apple standard, universally supported on CTV
- Master playlist (`.m3u8`) → media playlists → segments (`.ts` or `.fmp4`)
- Ad insertion points marked by:
  - `#EXT-X-CUE-OUT` / `#EXT-X-CUE-IN` (legacy method)
  - `#EXT-X-DATERANGE` (modern method, HLS spec)
  - SCTE-35 signals embedded in segments or manifest

### DASH (Dynamic Adaptive Streaming over HTTP)
- ISO standard (MPEG-DASH), used on Android TV, Fire TV, some Smart TVs
- MPD (Media Presentation Description) → Periods → AdaptationSets → Representations → Segments
- Ad insertion points marked by:
  - `<Period>` boundaries (new Period = potential ad break)
  - `<EventStream>` with SCTE-35 messages
  - `<InbandEventStream>` for in-band signals

### Apple TV / tvOS
- HLS only (DASH not natively supported)
- HLS Interstitials (HLS spec extension) for native ad insertion — but rarely used for programmatic

---

## 2. SCTE-35 in streams

**SCTE-35** signals ad insertion opportunities in broadcast/cable, carried over to OTT/CTV via manifests.

**In HLS:**
```
#EXT-X-CUE-OUT:DURATION=120.0
[ad segments here]
#EXT-X-CUE-IN
```
Or via `#EXT-X-DATERANGE` with SCTE-35 attributes:
```
#EXT-X-DATERANGE:ID="splice-123",START-DATE="2025-01-01T20:15:00Z",
PLANNED-DURATION=120.0,SCTE35-CMD=0xFC...
```

**In DASH:**
```xml
<EventStream schemeIdUri="urn:scte:scte35:2013:xml">
  <Event duration="1200000" id="123">
    <scte35:SpliceInfoSection>
      <scte35:SpliceInsert spliceEventId="123" outOfNetworkIndicator="true"/>
    </scte35:SpliceInfoSection>
  </Event>
</EventStream>
```

---

## 3. Ad insertion in manifest (SSAI)

**How stitching works:**
1. SSAI provider receives the original content manifest
2. Provider detects SCTE-35 markers
3. Provider makes ad request to GAM
4. Provider receives VAST response with MediaFile URL
5. Provider replaces marked segments with ad segments (transcoded to same format)
6. Provider delivers modified manifest to device

**Requirements for correct stitching:**
- Ad creative must be transcoded to the same codec, bitrate, and segment format as the content
- Same segment duration (typically 2-6s)
- Same audio structure (sample rate, channels)
- If mismatch → glitches, audio desync, or black frames in transitions

---

## 4. Manifest problem diagnosis

| Symptom | Check in manifest | Probable cause |
|---|---|---|
| Mid-rolls not inserted | Are there SCTE-35 markers? | Encoder not inserting markers |
| Black screen at transition | Gap between content and ad segments? | Incorrect stitching, ad segments not aligned |
| Audio desync | Same codec/sample rate in content and ad? | Audio params mismatch |
| Ad cuts off | Slot duration vs ad duration? | Slot shorter than creative |
| Buffering in ad | Ad bitrate vs bandwidth? | Ad with excessive bitrate for the connection |

**Diagnostic tools:**
- `ffprobe` to inspect segment properties
- Manual manifest download and structure verification
- Player developer tools (if available) for segment timeline

---

## 5. CMAF vs fMP4 vs TS in SSAI

| Format | Container | CTV use | SSAI stitching |
|---|---|---|---|
| MPEG-TS (`.ts`) | Legacy | Android TV legacy, HbbTV, STBs | Tolerant of small PTS desync; easier to stitch |
| fMP4 (`.mp4` fragmented) | Modern | tvOS, Tizen 5.5+, webOS 5+ | Requires exact alignment of `tfdt` timestamps between content and ad |
| CMAF | fMP4 + HLS + DASH unified | Tier 1 devices | Single asset serves HLS and DASH; higher pipeline complexity |

**SSAI implications:**

- **MPEG-TS:** more tolerant stitching because the muxer can absorb small PTS gaps. Still the common denominator in legacy pipelines.
- **fMP4:** the SSAI provider must ensure the `Movie Fragment` of the first ad segment continues the PTS timeline of the last content segment. Discontinuity → black frames or freeze at transition.
- **CMAF:** if the provider does not support native CMAF, you need two parallel pipelines (HLS TS for legacy devices, DASH fMP4 for Android TV/Fire TV). Real operational cost.

**Container mismatch diagnosis:**
```bash
ffprobe -v quiet -print_format json -show_streams segment_content.ts
ffprobe -v quiet -print_format json -show_streams segment_ad.ts
```
Compare `codec_name`, `sample_rate`, `r_frame_rate` between content segment and ad segment.

---

## 6. HLS Interstitials (reference, do not use for standard programmatic)

- Apple HLS spec extension (`#EXT-X-DATERANGE` with `X-ASSET-URI`) for native insertion in AVPlayer
- **Not used in standard CTV programmatic** — no GAM/IMA SDK integration via this path
- AVPlayer with IMA SDK still uses CSAI: IMA manages the ad break, AVPlayer pauses content
- Relevant only if the publisher builds a custom player in tvOS without IMA SDK and wants native OS ad insertion
- Mention to explicitly rule out when someone asks about "native ad insertion in tvOS"
