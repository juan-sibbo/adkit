# AdTech Diagnostic Patterns Reference

Publisher-side diagnostic patterns for building prompts that force structured analysis. Load this file when the prompt domain is AdTech.

## When to load this file

- User asks to debug Prebid, GAM, VAST, deal, CMP/consent
- User asks to review publisher-side AdTech code
- User describes a programmatic symptom without specifying the funnel layer
- User wants a prompt for another agent (Claude Code, Codex) touching publisher-side ad tech

## Core principle

Every publisher-side symptom has a **causal chain with multiple layers**. A poorly built AdTech prompt accepts the first symptom as the cause. A well-built prompt forces the model to walk the funnel in order without skipping layers.

---

## Funnel 1 — Deal not spending / not scaling

Mandatory order. Skipping steps produces incorrect diagnoses.

```
1. Supply chain / basic eligibility
   - ads.txt / sellers.json includes the SSP and correct seat
   - app-ads.txt if there is in-app/CTV traffic
   - Deal active on both sides (publisher and buyer)
   - Buyer seat authorized to see the deal

2. Avails — is there inventory that matches?
   - Deal targeting vs real inventory (geo, device, format, site/app list)
   - Deal floor vs publisher's effective floor (GAM pricing rules, UPR)
   - Frequency caps, advertiser/category blocklists, brand safety
   - Competitive exclusions in GAM

3. Listening — is the buyer receiving requests?
   - Avg QPS sent to SSP vs what SSP sends to buyer
   - SSP filtering by QPS, timeout, shaping
   - DSP's supply path optimization (SPO) — is this path prioritized?

4. Bidding — is the buyer bidding?
   - DSP bid rate on received requests
   - No-bid reason: exhausted budget, stricter DSP targeting, target CPM, creative not approved
   - Creative approval status in the SSP and in GAM

5. Delivery — does the bid win?
   - Win rate vs floor
   - Discrepancy between SSP-side (wins) and publisher-side (served impressions)
   - Creative render (VAST errors, timeout, viewability threshold)
```

**Common flag:** "the deal has 0 impressions" without checking layer 1. High probability of a supply chain problem before a bidding problem.

---

## Funnel 2 — Low fill rate in Prebid

```
1. Upstream consent
   - TC string present and valid before pbjs.requestBids()?
   - Bidder vendors declared in the CMP?
   - TCF purposes correctly mapped?
   - GPP string present if there is US multi-state traffic?

2. Prebid config
   - Bid adapters loaded in the build?
   - Reasonable bidderTimeout (1000-2000ms typical, lower in CTV)
   - Correct setConfig: priceGranularity, currency, userSync, floors module
   - User ID modules initialized before first auction?

3. adUnits
   - mediaTypes correctly defined (banner, video, native)
   - sizes / playerSize consistent with the ad slot
   - bids array with valid parameters for each bidder
   - ortb2Imp for contextual signals

4. Auction runtime
   - pbjs.onEvent('bidWon') and 'bidResponse' firing?
   - No-bid reasons in the response
   - cpm in Prebid vs cpm in GAM targeting — correct granularity?

5. GAM integration
   - Prebid line items configured with all buckets of the price granularity
   - Key-values hb_pb, hb_bidder, hb_adid reaching GAM?
   - TCF labels applied to line items?
   - setTargetingForGPTAsync() called before googletag.display()?
```

**Common flag:** upgrading priceGranularity from `dense` to `high` without updating line items in GAM → Prebid bids but never wins in GAM.

---

## Funnel 3 — VAST error / ad does not play (video/CTV)

```
1. Distinguish architecture
   - Client-side (IMA SDK, Prebid video) — DevTools, console, network
   - SSAI/SGAI/DAI (Google DAI, AWS Elemental, etc.) — server-side logs, beacons
   - HbbTV — broadcaster emulator or real device

2. VAST response
   - HTTP status of VAST URL
   - VAST version (2/3/4)
   - Wrappers: depth, timeouts between wrappers
   - Known VAST errors (100-900): parse, incompatible mediafile, no ads, timeout

3. MediaFile
   - Codec supported by the player? (h264/aac typical, HEVC problematic)
   - Reasonable bitrate for the bandwidth?
   - Duration within ad pod window
   - Valid HLS/DASH manifest if applicable

4. Beacons and tracking
   - Impression pixel fires?
   - Quartiles (start, firstQuartile, midpoint, thirdQuartile, complete)
   - Error pixel — does it fire before impression? → pre-roll failure
   - OMID viewability if applicable

5. CTV-specific
   - app-ads.txt for the bundle ID
   - IDFA/GAID/RIDA/AFAI/TIFA sent if applicable + consent
   - PAL nonce correctly generated (if SDK-less)
   - Secure Signals in ortb2
```

**Common flag:** confusing "black screen" with VAST error. Black screen can be: VAST OK but creative not loading, empty ad pod after filtering, or SSAI fallback misconfigured. Requires distinguishing the layer.

---

## Funnel 4 — Consent / CMP not working

```
1. Banner appearing?
   - CMP script loads before any advertising tag
   - No JS errors blocking its render
   - Geolocation correctly detected (jurisdiction)

2. TC string
   - Generated after user interaction?
   - getTCData() returns valid structure?
   - cmpStatus: 'loaded' → 'stub' → 'loaded' → 'loaded' (yes, twice)
   - gdprApplies correct for the jurisdiction

3. Propagation to Prebid
   - consentManagement module loaded
   - pbjs.setConfig with correct cmpApi and timeout
   - Bidders receive TC string in OpenRTB request (user.ext.consent)
   - GDPR flag in request (regs.ext.gdpr)

4. Propagation to GAM
   - googletag.pubads().setPrivacySettings({restrictDataProcessing}) if applicable
   - TCF labels on GAM line items
   - Non-personalized ads flag

5. GPP (US multi-state)
   - GPP string generated for the correct jurisdiction
   - Applicable sections present (us_national, us_ca, us_co, etc.)
   - usp_v1 legacy if still applicable
```

**Common flag:** Prebid starts before the CMP resolves → auction without consent → bidders do not bid or bid without signals. Use `consentManagement.allowAuctionWithoutConsent: false` and review load order.

---

## Funnel 5 — GAM line item not delivering

```
1. Basic status
   - Line item active, within dates, with approved creative
   - Delivery indicator in GAM UI — what does it say?

2. Priority and competition
   - Line item type (Sponsorship / Standard / Network / Bulk / Price Priority / House)
   - Other higher-priority line items competing for the same inventory
   - Roadblocking correctly configured if it is a sponsorship

3. Targeting
   - Ad unit targeting — does it include the ad unit where it should serve?
   - Key-values — do they match what the page sends?
   - Geo, device, audience segments
   - Exclusion labels (TCF, competitive, brand safety)

4. Inventory and forecasting
   - Line item forecast availability
   - Competing line items consuming the same supply
   - Pacing mode (ASAP / Even / Frontloaded)

5. Creative
   - Creative size matches ad unit
   - Creative approved by buyer/trafficking team
   - Third-party tag renders correctly (if applicable)
   - SafeFrame vs friendly iframe expectations
```

**Native diagnostic tools:**
- **Ads Inspector** (GAM Publisher UI) — by URL, simulates inventory match
- **Publisher Console** (`?google_console=1` in the URL) — render-side debugging
- **Troubleshoot tab** on the line item — delivery indicator explained

---

## Funnel 6 — Discrepancy SSP vs GAM vs Buyer

```
1. Define the discrepancy
   - SSP UI says X, GAM says Y, Buyer says Z — magnitude?
   - Typical discrepancy <10% is normal, >20% indicates a problem

2. Typical causes in order of frequency
   - Timing: GAM counts on render, SSP on win, Buyer on impression served
   - Different viewability threshold between parties
   - Different reporting timezone
   - Filtered impressions (IVT, post-bid brand safety)
   - Creative failing to render after win

3. Reconciliation
   - Compare same day, same TZ, same definition (impressions vs valid vs billable)
   - Revenue vs revenue reconciling with the same currency conversion

4. Red flags
   - Growing discrepancy over time → config drift
   - Discrepancy only in one device/geo → targeting issue
   - Discrepancy coincides with deploy → code regression
```

---

## Signals that MUST NEVER be missing in an AdTech prompt

If you are building a diagnostic prompt and the user has not provided these data points, **ask for them** (within the limit of 3 questions):

| If the prompt is about... | Always ask for... |
|---|---|
| Prebid | Version, mode (client/server), list of loaded modules |
| GAM | 360 or non-360, whether it is trafficking/forecasting/yield/discrepancy |
| VAST | Client-side or SSAI/DAI/SGAI, device/player, VAST version |
| Deal | Deal ID, SSP, type (PG/PD/PA), buyer seat ID |
| CMP | Vendor (Didomi/OneTrust/Usercentrics), framework (TCF v2.2 / GPP), jurisdiction |
| CTV | App bundle ID, store (Roku/FireTV/etc.), SDK-less or IMA/PAL |
| Specific bidder | Adapter version, concrete params, endpoint |

---

## Anti-patterns when building AdTech prompts

- Asking "why isn't my Prebid working?" without indicating version or modules
- Giving the symptom of one layer as the cause of another ("no revenue" ≠ "no bids")
- Pasting Prebid code without indicating whether it is in production or staging
- Assuming the problem is in Prebid when it could be in the upstream CMP
- Mixing publisher-side diagnosis with buyer-side diagnosis (different source of truth)
- Not distinguishing between "the deal is misconfigured" and "the deal is correct but there is an inventory problem"
- Asking for VAST diagnosis without distinguishing client-side vs SSAI
