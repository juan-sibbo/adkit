# Line Items — Architecture, priorities and trafficking

## Priority hierarchy in GAM

GAM resolves which line item serves through a priority system. The evaluation order is deterministic: when two eligible line items exist, the one with higher priority wins. Within the same priority level, the system applies internal optimization (CTR history, goal pacing, etc.).

### Types and priorities table

| Type | Numeric priority | Typical use | Reserves inventory |
|---|---|---|---|
| Sponsorship | 4 | Exclusive sponsorships, 100% SOV or fixed % | Yes |
| Standard | 8 | Direct campaigns with guaranteed impressions | Yes |
| Network | 12 | Network monetization, no guarantee | No |
| Bulk | 12 | High-volume campaigns without strict guarantee | No |
| Price Priority | 16 | Programmatic demand (Open Bidding, Prebid) | No |
| House | 16 (effective minimum) | Self-promotions, fallback | No |

> **Note:** numeric values are indicative. GAM uses categories, not an open linear scale. The real priority within each category is managed with the line item's "priority" field (1–16 in some types).

**Critical operational rule:** Sponsorship and Standard reserve inventory and participate in forecasting. Network, Bulk, Price Priority, and House do not reserve — they compete in real time.

---

## Sponsorship

### When to use it
- Exclusive sponsorships (100% SOV on an ad unit or section)
- Fixed share of voice (e.g. 50% of impressions on the homepage)
- Branding campaigns with guaranteed presence

### Key configuration
- **Rate type:** % of day (percentage of daily impressions)
- **Goal:** percentage (not absolute impressions)
- Multiple sponsorships on the same inventory share the configured percentage. If the sum exceeds 100%, GAM does not prevent this in setup but delivery is affected.

### Common risks
- Sponsorship with very broad targeting can block delivery of all lower-priority line items on that inventory
- Not verifying the sum of % of active sponsorships before activating a new one → overselling
- Using sponsorship for campaigns that are actually standard (fixed impressions) → incorrect forecasting

---

## Standard

### When to use it
- Direct campaigns with a fixed number of guaranteed impressions
- Any campaign with a measurable delivery objective (impressions, clicks)

### Key configuration
- **Goal:** impressions or clicks
- **Rate type:** CPM or CPC
- **Pacing:** the system distributes delivery to reach the goal in the configured period. Options: even (linear), front-loaded, as fast as possible.
- **Priority within Standard:** 1–16 (lower number = higher priority). Allows managing premium vs standard campaigns within the same type.

### Pacing and delivery
- Even pacing: the system calculates a daily rate based on available inventory and remaining impressions.
- If the configured rate exceeds available inventory, the line item does not reach goal → review forecasting at the time of booking.
- Default overdelivery buffer: GAM can deliver up to 10% more than the goal to compensate for low-traffic days. Configurable.

---

## Price Priority

### When to use it
- Programmatic demand: Open Bidding, Prebid (header bidding), AdSense/Ad Exchange as demand
- Any integration where the bid price determines which ad serves

### Key configuration
- **Rate type:** CPM
- **Floor price:** minimum price below which the line item does not serve
- Price Priority competes in real time: GAM evaluates which line item of this type has the highest CPM on each impression

### Relationship with Prebid
- Prebid line items are Price Priority with `hb_pb` (price bucket) targeting
- The price bucket architecture (granularity) must align with the created line items → see `prebid-adtech`
- A common error: creating Price Priority line items with overlapping ranges → unpredictable delivery behavior

---

## House

### When to use it
- Self-promotions, own content, fallback campaigns
- Unsold inventory — House serves when no other line item is eligible

### Key configuration
- House has the lowest priority in the system. Never competes with real demand.
- Does not consume forecasting or reserve inventory.
- Can be used as a "safety net" to avoid blank impressions.

### Common risk
- House with overly broad targeting that serves on inventory that should be blank for measurement or testing.

---

## Roadblocking

Mechanism to serve multiple creatives simultaneously across multiple ad units on the same page.

### Types
- **One or more:** the creative serves on any available ad unit (default behavior)
- **As many as possible:** serves on all available ad units in the page view
- **All:** only serves if ALL targeted ad units are available in the page view. If one is missing, none serve.

### When to use "All"
Campaigns where the creative experience requires simultaneous presence (e.g. skin + banner + medium rectangle). The risk: if an ad unit is not on the page or is blocked by another line item, the campaign delivers nothing. Requires coordination with the technical team to verify ad units match.

### Forecasting and roadblocking
Forecasting for a roadblock considers the joint availability of all involved ad units — available inventory will be less than that of each individual ad unit. Verify the availability check for the complete roadblock, not for each ad unit separately. `[360]` Detailed forecasting with ad unit breakdown in roadblocks is more accurate in GAM 360.

---

## Ad Rules

Ad rules control the advertising experience at page or session level: maximum number of ads, spacing between ads, category competition.

### Rule types
- **Competitive exclusion:** prevent two advertisers in the same category (e.g. two banks) from appearing on the same page. Managed through competition labels on advertisers/orders.
- **Ad spacing:** minimum time or pages between impressions from the same advertiser.
- **Line item-level frequency capping:** limit how many times a user sees a specific line item in a period.

### Frequency: configuration levels
Frequency capping can be configured at three levels, and is applied cumulatively:
1. **Line item:** maximum N impressions per user within X time
2. **Order:** maximum N impressions from the set of line items in that order
3. **Ad unit / placement:** `[360]` frequency capping at inventory level

**Common risk:** overly restrictive frequency capping on small audiences → the line item quickly exhausts available impressions per user and stops delivering before reaching goal.

---

## Creative trafficking

### HTML5
- Upload as .zip file with index.html at the root
- Verify the creative passes GAM validation (size, referenced assets)
- Click tag must be implemented with GAM's `clickTag` variable, not a hardcoded href
- Maximum zip size: 200KB by default. `[360]` configurable limits.

### AMPHTML
- Specific format for AMP pages. Requires validation against the AMP validator.
- Only works on AMP inventory. Does not serve on standard pages.
- JavaScript restrictions: arbitrary JS is not allowed, only certified AMP components.

### Native
- Requires defining a native format in GAM that matches the page template
- Creative fields (headline, image, CTA, etc.) must be mapped to the template
- Common error: the native format in the creative does not match the one defined in the ad unit → does not serve

### Responsive
- The creative adapts to the ad unit size in real time
- Configure breakpoints correctly in the creative
- Verify the ad unit has "fluid" size or the specific sizes the responsive creative covers enabled

### Tracking macros
Standard GAM macros usable in creatives and tracking URLs:
- `%%CACHEBUSTER%%` — random number to prevent caching
- `%%CLICK_URL_ESC%%` — encoded click URL
- `%%PATTERN:key%%` — value of key-value `key` at the time of the impression
- `%%ADUNIT%%` — name of the ad unit that served
- `%%SITE%%` — site domain

### Creative approval
- Creatives go through automatic GAM review (content policies, size, format)
- A rejected creative blocks delivery of the line item if it is the only associated creative
- Add multiple creatives per size as backup
- `[360]` manual review available for creatives with special formats
