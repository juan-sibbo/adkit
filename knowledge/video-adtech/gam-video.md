# GAM Video — Technical reference

## Table of contents
1. App registration and prerequisites
2. Ad Rules and Master Video Tag
3. Ad Pods: fixed vs dynamic, bumpers
4. Competitive exclusion and correlator
5. Pod bidding (AdX)
6. MRSS → GAM (Video Content Sources)
7. Content ID vs cust_params
8. WTA compliance in GAM
9. Server-side Open Bidding
10. Server-Side Unwrapping (SSU) config
11. Publisher Provided Signals (PPS) in GAM
12. Yield groups and demand channels

---

## 1. App registration and prerequisites

Verify FIRST in low fill diagnosis:

- [ ] App registered in GAM: Inventory > Apps > app claimed and active
- [ ] Correct Store ID by platform (format varies: Roku numeric, Samsung with G prefix, etc.)
- [ ] app-ads.txt published on developer domain and validated (IAB crawler)
- [ ] Ad unit configured as video type (not display)
- [ ] `msid` parameter (bundle ID / app ID) arriving correctly in the VAST tag
- [ ] `an` (app name) present in the tag

**Consequence of invalid app-ads.txt:** DSPs block inventory → fill rate drops drastically.

---

## 2. Ad Rules and Master Video Tag

**Ad Rules:** GAM engine that defines when/where/how many ads to insert.

**Components:**
- **Ad break timing:** pre-roll (0:00), mid-rolls (every X min or at cue points), post-roll (end)
- **Pod config:** max duration, max ads per pod, allowed durations (15s, 30s)
- **Restrictions:** no mid-roll before min 5, no post-roll if content <10min, minimum spacing between mid-rolls

**Master Video Tag:** single URL with `ad_rule=1` containing all configuration. Passed to the IMA SDK once; SDK manages all ad breaks automatically based on GAM's VMAP response.

Mandatory parameters in the tag:
- `iu` (ad unit path)
- `sz` (size, typically 640x480)
- `env=vp` (video player)
- `gdfp_req=1` (GAM request)
- `output=xml_vast4` (or xml_vast3)
- `correlator` (timestamp for cache-busting and competitive exclusion)
- `ad_rule=1` (if using Ad Rules)
- `cmsid` + `vid` (if using Video Content Sources)

---

## 3. Ad Pods: fixed vs dynamic, bumpers

**Fixed duration pod:** always X seconds (e.g. 120s). If not enough ads → fill with house ads or leave a gap.
**Dynamic duration pod:** duration varies based on fill. Better UX (no forced house ads) but less predictable revenue.

**Bumpers:** brief slates (3-5s) before/after the pod.
- **Recommended implementation:** app-side (does not consume an ad request, does not affect metrics)
- **Alternative implementation:** GAM as a maximum-priority line item (counts as an impression)

---

## 4. Competitive exclusion and correlator

**Competitive exclusion labels:** in GAM, Admin > Inventory > Competitive Exclusion Labels. Assign to advertisers that should not appear in the same pod.

**Correlator:** numeric parameter (timestamp) in the ad tag.
- Same correlator = same "page view" / content session [Verified]
- GAM uses correlator for:
  - Competitive exclusion within the pod (not serving 2 ads with same label)
  - Roadblocking (serving ads from the same campaign together)
  - Deduplication (not serving the same creative twice in the same pod)
- **New correlator = new content** [Verified]
- **Anti-pattern:** reusing correlator across different content → competitive exclusion applied incorrectly

---

## 5. Pod bidding (AdX)

- Buyers can bid for a specific position within the pod
- Premium positions: 1st (highest attention) and last (recency effect)
- Automatically enabled in AdX for CTV [Probable]
- Differential pricing configuration by position via line items with pod position targeting

---

## 6. MRSS → GAM (Video Content Sources)

- GAM ingests MRSS feeds via Video > Content Sources [Verified]
- Sync frequency configurable [Verified]
- Mandatory fields: `<guid>` (content ID), `<title>`, `<media:content>` (URL + duration)
- Recommended fields: `<media:category>`, `<media:rating>`, `<media:keywords>`
- Cue points from MRSS: `<media:cuepoint time="300"/>` [Verified]
- Ingestion latency: typically 1-2 hours post-crawl

---

## 7. Content ID vs cust_params

**Preferred method (GAM native):** Content ID via MRSS
- Ad request only passes `vid=CONTENT_ID` + `cmsid=CMS_ID`
- GAM automatically enriches with Content metadata
- Advantage: short URL, centralized metadata

**Alternative method:** metadata in cust_params
- `cust_params=genre%3Ddrama%26rating%3Dtv_ma`
- Advantage: flexible, does not require pre-registration
- Limitation: URL length ~2000 chars, manual encoding

**When to use each:** Content ID if the catalog is stable and pre-registered in MRSS; cust_params if content is very dynamic or third-party.

---

## 8. WTA compliance in GAM

- Configurable at network level: Admin > WTA settings
- If active: GAM requires ads to be able to show transparency
- CTV: many user agents do not support WTA rendering → `wta=0`
- If `wta=0` and no non-personalized creatives → empty VAST [Probable]
- See ima-sdk.md §6 for detailed diagnosis

---

## 9. Server-side Open Bidding

**What it is:** server-side auction in GAM where multiple SSPs/exchanges compete.

**Difference from client-side header bidding:**
- Client-side (Prebid): device runs auction in parallel → 1-3s latency, CPU intensive
- Server-side (Open Bidding): GAM server runs auction → lower latency [heuristic: 200-500ms, depends on partners and network]

**Typical CTV Europe partners:** Xandr (Microsoft), Magnite, Index Exchange, PubMatic

**Configuration:** Admin > Yield Management > Yield Partners

**Trade-off:** more partners = more competition = better yield, but also potentially more latency. On CTV with an aggressive timeout, limit to 3-5 partners.

---

## 10. Server-Side Unwrapping (SSU)

See vast-vmap.md §5 for technical detail.

**Configuration in GAM:** Admin > Video > Server-side ad unwrapping
- Enable for video ads
- Max redirects configurable
- Timeout per redirect configurable

---

## 11. Publisher Provided Signals (PPS) in GAM

**Configuration:**
1. Admin > Publisher Provided Signals: define custom signals, map to IAB taxonomy
2. Admin > Programmatic Setup > Demand Channels: enable sharing per channel (Authorized Buyers, Open Bidding, Open Auction)
3. Signals are automatically passed in the OpenRTB bid request

**Signal types:**
- Contextual: genre, rating, content keywords
- Audience: first-party segments (high-value viewers, binge watchers)
- Technical: device type, OS, connection type

**Impact:** more signals → DSPs bid better → higher CPMs, more competition in auction, better fill.

---

## 12. Yield groups and demand channels

**Yield groups:** groupings of yield partners (Open Bidding) for specific inventory.
- Create yield group by inventory type (CTV Sports, CTV VOD, etc.)
- Assign specific SSPs per group
- Configure floors per yield group

**Demand channels:** types of demand that access the inventory
- Google Ads
- Display & Video 360
- Authorized Buyers
- Open Bidding partners

**Verify in low fill diagnosis:** are there active yield groups for the CTV inventory type?
