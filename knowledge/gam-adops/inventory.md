# Inventory — Ad Units, Key-Values, Forecasting, Audience, PPID, PPS

## Ad Unit Hierarchy

GAM organizes inventory in an ad unit hierarchy. The typical structure is:

```
Network (root level)
└── Site / Property
    └── Section / Channel
        └── Position (leaderboard, mrec, interstitial...)
```

### Design principles
- **Appropriate granularity:** too many levels complicate trafficking and forecasting. Too few prevent precise targeting.
- **Consistent naming:** clear conventions (`site_section_position`) facilitate automation and reporting.
- **Parent vs child ad unit:** targeting on a parent ad unit includes all its children. Useful for full-section campaigns. Risk: if a campaign is booked on the parent, forecasting includes all child inventory.
- **Sizes defined in the ad unit:** creatives can only serve on ad units that declare their size. Verify ad unit sizes cover all creative formats to be trafficked.

### Placements
Placements are ad unit groupings to simplify targeting multiple positions. Useful for commercial packages ("all mrecs on the site") without selecting each ad unit individually.

**Limitation:** a placement is a selection alias, not an independent inventory entity. Forecasting on a placement sums the availability of all its ad units.

---

## Key-Values

Key-values are key=value pairs that enrich the information available at the time of the ad request, enabling precise targeting beyond the ad unit hierarchy.

### Key types
- **String:** text values (section, category, content type)
- **Number:** numeric values for ranges (age, price, score)
- **Boolean:** true/false (registered user, subscriber, first visit)

### Configuration
- Keys must be created in the network before they can be used in targeting
- Values can be predefined (closed list) or dynamic (the publisher sends any value)
- **Predefined:** more control, the system can report by value. **Dynamic:** more flexible, but forecasting cannot segment by specific value.

### Correct key-value verification in troubleshooting
It is not enough for the key to be "defined" in GAM. Verify three things:
1. The key exists in GAM and is active
2. The expected value is actually arriving in the ad request — use Publisher Console or `googletag.pubads().getTargeting('key')` in the console
3. The line item targeting evaluates as expected — case-sensitive, no encoding differences, accents, or spaces

**Common error:** GAM targeting is case-sensitive. `section=Sports` and `section=sports` are different values. Always verify the actual value sent from the page, not the expected one.

### Key-values and Prebid
Key-values `hb_pb`, `hb_bidder`, `hb_adid`, etc. are sent by Prebid.js via GPT targeting. They are key-values like any other from GAM's perspective. → See `prebid-adtech` for the generation logic.

---

## Forecasting

GAM forecasting estimates available inventory for a line item given its targeting, dates, and goal, considering existing commitments.

### Availability check
Before confirming a booking, always run an availability check with the exact line item configuration:
- Full targeting (ad unit, geo, device, key-values, audience)
- Exact dates
- Goal in impressions

The availability check returns: **available** (free impressions), **matched** (total impressions meeting the targeting in the period), and **contended** (committed to other line items).

### Forecasting limitations
- Forecasting is an estimate based on historical traffic. It does not guarantee delivery.
- Unforeseen traffic spikes, content changes, or new higher-priority campaigns can affect actual delivery.
- Forecasting does not verify whether the ad unit has a creative of the correct size → always verify.
- `[360]` GAM 360 offers forecasting with greater historical granularity and better projections for inventory with complex audience targeting.

### Inventory contention
When the availability check returns high contention, options:
1. Broaden targeting (more ad units, more geo, more devices)
2. Reduce the goal or extend the dates
3. Lower the priority if the campaign can tolerate non-guaranteed delivery
4. Review which existing line items are consuming the inventory

### Unfulfilled impressions
GAM reports impressions that had no eligible line item. A high volume indicates:
- Unmonetized inventory (house ads not configured, targeting gaps)
- Key-values sent with no line items consuming them
- Ad unit sizes with no creatives of that size in any active line item
- Ad units not included in any yield group (no Open Bidding demand)

---

## Audience Segments

### Segment types in GAM
- **First-party:** built from user behavior on the publisher's sites
- **Google audience:** Google segments available for targeting (interests, demographics) `[360]`
- **Third-party:** external DMP segments integrated via targeting `[360]`

### First-party segment configuration
1. Define segment rules in GAM (e.g. "users who visited /sports in the last 7 days")
2. Verify the GAM tag on the page sends the signals needed to populate it
3. The segment takes time to populate — it is not available instantly
4. Minimum segment size: GAM does not allow targeting on segments that are too small for privacy reasons

### Common risks
- Audience segment targeting on line items with high goals + small segment → insufficient delivery
- Segments with very short expiration rules in campaigns that need accumulation
- Not verifying segment size before selling inventory based on it

---

## PPID (Publisher Provided Identifier) `[360]`

**GAM 360 exclusive feature.** PPID allows the publisher to send a proprietary (hashed) identifier to GAM to enable cross-session frequency capping, audience targeting, and reach measurement without relying on third-party cookies.

### Implementation
```javascript
// Sending PPID via GPT — must be done before the first ad request
googletag.pubads().setPublisherProvidedId('HASHED_USER_ID');
```

- The identifier must be hashed (SHA-256 recommended) before being sent
- Do not send unhashed PII
- PPID must be sent before the first ad server call on each page view

### Use cases
- **Cross-session frequency capping:** without PPID, capping is based on cookies that are lost when the browser is closed or in private browsing
- **Sequential rotation:** guarantee a user sees a campaign's creatives in order `[360]`
- **Persistent audience segments:** identified users can belong to segments even without cookies
- **Reach & frequency reporting:** more accurate metrics of unique users reached `[360]`

### Limitations
- Only works for identified users (logged-in or with identification consent)
- On unauthenticated traffic, the system reverts to cookie-based behavior
- Full PPID integration with audience segments, sequential rotation, and advanced reporting requires GAM 360

---

## Publisher Provided Signals (PPS)

**Distinct from PPID.** PPS is the mechanism by which the publisher sends structured context and audience signals to GAM using standardized taxonomies (IAB Content Taxonomy, IAB Audience Taxonomy), without relying on third-party cookies or user identifiers.

### Difference from PPID
- **PPID:** persistent user identifier for frequency capping and audience targeting
- **PPS:** content and audience signals to enrich the ad request without individually identifying the user

### Implementation
```javascript
googletag.pubads().setPublisherProvidedSignals({
  taxonomies: {
    'IAB_AUDIENCE_1_1': ['6', '284'],   // IAB Audience Taxonomy v1.1
    'IAB_CONTENT_2_2': ['17', '614']    // IAB Content Taxonomy v2.2
  }
});
```

### Use cases
- Enrich the ad request for programmatic demand using contextual targeting
- Improve match rate on cookieless inventory
- Reporting by content/audience category with PPS-specific dimensions in GAM

### PPS reporting
GAM exposes specific dimensions for PPS in reports (Content labels, Audience segment labels). Useful for analyzing which categories generate the most revenue and calibrating signal coverage.

---

## Ad Load and Ad Serving Limits

### Ad serving limits
GAM imposes dynamic limits on the number of ads that can be served per page and session to protect the user experience.

**Indicators that the limit is being reached:**
- Unfulfilled impressions on high ad density inventory
- Slots consistently returning blank on pages with many ad units

### Ad load
There is no universally correct number. Factors:
- Content type and expected dwell time
- Editorial content to advertising ratio
- Mobile vs desktop experience (limits are stricter on mobile)
- Better Ads Standards policies

**Starting heuristic:** no more than 1 ad per "screen" of content on mobile. Measure fill rate and unfulfilled impressions to adjust.

### Ad refresh
Subject to Google policies:
- Minimum 30 seconds between refreshes
- Only allowed if the user has viewed the ad (viewability)
- Document the refresh in the publisher's ads policy

---

## GAM 360 vs GAM non-360 — Key operational differences

| Feature | GAM non-360 | GAM 360 `[360]` |
|---|---|---|
| Impression limit | 90M/month | No limit |
| Forecasting | Basic | Full, with segment breakdown |
| Audience segments | Limited | Full + DMP integration |
| PPID | Not available | Full (frequency capping, sequential rotation, reach reporting) |
| PPS | Available | Available + advanced reporting dimensions |
| Programmatic Guaranteed | No | Yes |
| Open Bidding | No | Yes |
| First Look | No | Yes |
| Session video ad rules | No | Yes (require user identifier) |
| Ad Exchange demand | Limited | Full |
| API access | Limited | Full |
| Custom reporting | Basic | Advanced, with custom dimensions |
| Detailed IVT reporting | No | Yes |
| Bulk trafficking | Manual | Bulk trafficking tools |

**Note:** migrating from GAM non-360 to 360 preserves ad units, orders, and line items, but requires review of API integrations and reporting configurations.
