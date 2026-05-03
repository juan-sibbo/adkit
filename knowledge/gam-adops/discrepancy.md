# GAM vs SSP Discrepancies — Analysis and protocol

## What a discrepancy is and when it is normal

A discrepancy is the difference between impressions (or revenue) reported by GAM and those reported by an SSP, DSP, or other external measurement system.

**Acceptable discrepancy:** the industry accepts discrepancies of up to 10-15% as normal due to methodological differences between systems. Above that threshold, investigation is required.

**Direction of the discrepancy:**
- GAM reports more than the SSP: common. GAM counts when it serves the ad; the SSP may miss impressions due to timeout, invalid traffic filters, or tracking issues.
- GAM reports less than the SSP: less common. May indicate a revenue discrepancy (the SSP reports won bids that GAM did not correctly count) or integration issues.

---

## Types of discrepancy

### 1. Impression discrepancy (volume discrepancy)

**Most common causes:**

| Cause | Description | Likelihood |
|---|---|---|
| Different impression definition | GAM counts when the ad initiates. Some SSPs count when the buyer's impression beacon arrives. There is always latency between both events. | High |
| Different invalid traffic (IVT) filtering | GAM filters IVT before counting. SSPs may have different filters. | High |
| Timezone discrepancy | GAM and SSP reports may be in different time zones. Over short periods it can look like a real discrepancy. | Medium |
| Ad timeouts | The SSP registers the won bid but the creative does not arrive in time → GAM serves another ad or blank | Medium |
| Header bidding wrapper | In Prebid configurations, the wrapper may not correctly register which bid won → discrepancy between GAM and Prebid reporting | Medium |
| Tag / pixel issues | The SSP tag does not fire on all impressions (JS blocked, adblocker, implementation error) | Variable |
| Fallback impressions | GAM serves a fallback ad when the SSP returns no bid. The SSP does not count them; GAM does. | Variable |

### 2. Revenue discrepancy

**More sensitive than impression discrepancy.** A revenue discrepancy may indicate:

- **Floor price not respected:** the SSP is reporting the clearing price but GAM is applying a different floor
- **Fee discrepancy:** GAM and the SSP include or exclude fees at different points in the calculation (gross vs net)
- **Currency mismatch:** reports are in different currencies and the applied exchange rate differs
- **Bid caching:** `[360]` in environments with First Look or Dynamic Allocation, the bid used may be from the previous request if there is caching

---

## Discrepancy analysis protocol

### Step 1 — Scope the problem

Before diagnosing, determine:
- Is it an impression discrepancy, a revenue discrepancy, or both?
- What exact period? (day, week, month — verify the timezone of both systems)
- Which specific ad unit / placement / inventory?
- Which SSP or external system?
- Is the discrepancy percentage constant or does it vary by day/hour?

A constant percentage discrepancy suggests a systematic methodological difference. A variable discrepancy may indicate a technical problem.

### Step 2 — Compare intermediate metrics

Do not compare only total impressions. Break down:

```
GAM: Ad Requests → Matched Requests → Impressions → Clicks
SSP: Bid Requests → Bids → Won Bids → Impressions → Clicks
```

The point of divergence gives clues about the cause:
- Divergence in bid requests vs ad requests → problem in the tag integration
- Divergence in won bids vs GAM impressions → timeouts, invalid traffic, fallback
- Divergence only in revenue → fees, floors, currency

### Step 3 — Verify timezone and dates

```
GAM default: publisher timezone (configured in the network)
Most SSPs: UTC or US Eastern

1-2h difference can generate ~5-8% differences in daily reports
if traffic is not evenly distributed throughout the day.
```

Always compare complete periods (full weeks, not partial days) to eliminate timezone noise.

### Step 4 — Verify invalid traffic filters

GAM automatically filters IVT and does not count it in impressions. Some SSPs filter less aggressively or in post-processing. To verify:
- Enable the "Invalid Traffic" dimension in GAM reports (if available)
- Ask the SSP for their IVT filtering methodology
- `[360]` GAM 360 offers detailed IVT reporting filtered by type

### Step 5 — Review technical implementation

If the previous steps do not explain the discrepancy:
- Verify the SSP tag is correctly implemented on all affected ad units
- Check if there are adblockers affecting the SSP tag but not GAM's (or vice versa)
- In Prebid environments: verify `pbjs.getEvents()` shows won bids that match those reported by the SSP
- Check for unfired beacons in the Network tab for specific impressions

---

## Document and escalate

If the discrepancy consistently exceeds 15% and the previous steps do not explain it:
1. Document with data: period, percentage, direction, intermediate metrics compared
2. Verify whether the issue is bilateral or only in one direction
3. Escalate to SSP support with the documented data — most SSPs have a formal discrepancy reconciliation process
4. In revenue discrepancies with contractual implications, the usual source of truth is GAM (publisher-side), unless otherwise agreed with the buyer

---

## Discrepancy in mixed display + video environments

In publishers with mixed inventory, video discrepancies tend to be larger than display ones due to:
- Greater complexity of the technical chain (VAST, player, SSAI)
- More failure points (buffering, video viewability, completion rate)
- Differences between video impression definitions (started vs first quartile vs complete)

For video-specific discrepancies (VAST beacons, quartile tracking, SSAI) → also load `video-adtech/references/gam-video.md`.
