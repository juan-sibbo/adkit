# Measurement CTV — Technical reference

## Table of contents
1. OMID (Open Measurement) on CTV
2. Inferred Viewability: GAM methodology
3. Tracking beacons: lifecycle
4. GAM vs SSP/DSP discrepancies
5. VCR (Video Completion Rate)
6. Frequency capping with Device IDs

---

## 1. OMID (Open Measurement) on CTV

**What it is:** IAB Tech Lab standard (v1.3+) that unifies viewability/fraud/brand safety measurement. One SDK, multiple vendors.

**Without OMID:** integrating 5 separate measurement SDKs.
**With OMID:** IMA SDK includes OM SDK built-in. Vendors provide verification scripts via `<AdVerifications>` in VAST 4.x.

**Flow:**
1. GAM generates VAST with `<AdVerifications>` node
2. IMA SDK parses it, detects vendors (Moat, IAS, DoubleVerify)
3. SDK loads OM SDK + verification scripts
4. Vendors measure viewability, fraud, brand safety
5. Vendors report to their platforms

**GAM configuration:** Admin > Video > Third-party verification > add providers

**Measured metrics:**
- **Viewability** (MRC standard): 50% pixels visible, 2s continuous
- **Fraud detection**: invalid traffic, device spoofing, geo-spoofing
- **Brand safety**: content verification

**CPM impact:** OM-verified inventory can command +10-15% CPM vs unverified.

**CTV limitation:** on devices without OM SDK support → Inferred Viewability (see §2).

---

## 2. Inferred Viewability: GAM methodology

**What it is:** GAM methodology for devices/environments where direct OMID measurement is not possible.

**When it is used:**
- Legacy CTV devices without OM SDK support
- Gaming consoles (restricted environments)
- Set-top boxes with limited SDKs

**Methodology [Probable — GAM does not publish exact internal details]:**
- In-stream video inventory type (linear ad in content stream)
- Assumption: CTV = full screen, no other windows/tabs → 100% in-screen during playback
- If ad plays ≥2s continuously → counted as viewable (MRC standard for video)
- If skip/error before 2s → not viewable

**Reporting in GAM:**
- Dimension: "Measurement Source with Active View" [Verified]
- Values: "Direct" (OM SDK direct measurement) vs "Inferred"
- Filter by Measurement Source = "Inferred" to see CTV inventory with this methodology

**Inferred limitations — what it CANNOT detect:**
- If the TV is on but nobody is watching
- Viewer attention (watching TV vs phone)
- Audio level (TV muted)
- Sophisticated fraud (bot that plays 2s+)

**Pricing impact:** Inferred inventory may have a slight discount vs OM-verified (~5% CPM), but is accepted by the industry for CTV.

---

## 3. Tracking beacons: lifecycle

**Timeline of a 30s ad:**
```
0.00s → impression (ad begins rendering)
0.00s → start (playback begins)
7.50s → firstQuartile (25%)
15.0s → midpoint (50%)
22.5s → thirdQuartile (75%)
30.0s → complete (100%)
```

**Each beacon = GET request to tracking URL.** Response ignored (fire-and-forget).

**Multiple tracking pixels:** VAST can contain multiple URLs per event (GAM + advertiser + third-party verification). All fire simultaneously.

**Error beacon:** `<e>` node with `[ERRORCODE]` macro. Fires if the ad fails.

**In SSAI:** beacons must fire server-side if the player has no visibility of ads in the stream. See ssai-sgai-dai.md §2.

---

## 4. GAM vs SSP/DSP discrepancies

**Acceptable range:** 0-10% discrepancy [industry heuristic].
**Investigate if:** >10-12%.
**Serious problem if:** >15%.

**Common causes:**

| Cause | Effect | Who counts more |
|---|---|---|
| Different impression pixel timing | GAM fires on VAST response; DSP on playback start | GAM (counts ads that never played) |
| DSP IVT filters | DSP filters invalid traffic more aggressively | DSP (counts less) |
| DNS-based ad blockers (Pi-hole) | Block tracking domains | GAM (DSP pixel blocked) |
| Network timeouts | GAM pixel fires OK; DSP pixel times out | GAM (counts more) |
| Attribution window | GAM counts if starts before midnight; DSP if completes | Depends on event and timezone |
| Timezone mismatch | Reports in UTC vs local time | Artificial mismatch |

**Reconciliation process:**
1. Pull reports with same dates, same timezone
2. Compare day-by-day
3. Identify outliers (>10% on a specific day → investigate)
4. Agree on source of truth: average, GAM, DSP, or neutral third party
5. Credit/debit for under/over-delivery

---

## 5. VCR (Video Completion Rate)

**Formula:** (Completions / Starts) × 100

**Typical CTV ranges** [industry heuristic, no specific verifiable source]:
- CTV: significantly higher than web video
- Factors: lean-back experience, typically no skip button, full screen

**Factors affecting VCR:**
- Ad position: pre-roll > early mid-roll > late mid-roll > post-roll
- Creative quality: buffering → drop-off
- Duration: 15s typically has higher completion than 60s
- Ad load: too many ads → viewer fatigue → abandonment

**Low VCR as diagnostic signal:**
- <80% in CTV → investigate: broken creatives, defective VAST wrapper, codec issues, excessive bitrate for the device/network

---

## 6. Frequency capping with Device IDs

**Mechanism:** GAM tracks how many times a device ID (rdid) has seen a creative/advertiser.

**GAM configuration:** Line Item Settings > Frequency Caps
- Creative level: max N per day
- Advertiser level: max N per day
- Campaign level: max N per week

**CTV limitations:**

| Limitation | Impact | Mitigation |
|---|---|---|
| Requires valid rdid with is_lat=0 | Without rdid → no frequency cap | Session ID (sid) for intra-session cap |
| Household-level, not user-level | 3 viewers × cap 3/day = 9 household exposures | Conservative caps (5-7/day for household) |
| Not cross-device | 2 TVs = 2 device IDs = independent caps | No native solution |
| User can reset ID | Cap lost on reset | Minimal impact (few users do it) |
