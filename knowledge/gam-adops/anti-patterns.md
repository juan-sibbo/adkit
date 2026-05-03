# Anti-Patterns GAM AdOps — Symptom → Diagnosis → Resolution

Real cases ordered by frequency. Each entry follows the format:
**Symptom → Probable cause → Verification → Resolution → Evidence level**

---

## Line item delivery

### AP-001 — Active line item delivering nothing

**Symptom:** line item in "Delivering" status but impressions = 0 or close to 0.

**Probable causes (ordered by frequency):**
1. Creative rejected or pending approval → the line item has no active creative to serve
2. Overly restrictive targeting → the ad request never meets all conditions simultaneously
3. Key-value mismatch → the value sent from the page does not exactly match what is configured in targeting
4. Creative size does not match any ad unit in the targeting
5. Frequency capping exhausted for all users in the segment

**Verification:**
- Review the status of each associated creative (approved / pending / rejected)
- Enable GAM diagnosis mode: "Delivery Diagnostics" or "Troubleshoot" on the line item
- Compare key-values sent in the tag (Network tab, `googletag.pubads().getTargeting()`) with those configured in GAM
- Review the ad requests vs matched requests report for the line item

**Resolution:** depends on the identified cause. In 60-70% of cases it is a key-value mismatch or a rejected creative.

**Evidence: Probable** (requires data from the specific case to verify the exact cause)

---

### AP-002 — Sponsorship not reaching the configured %

**Symptom:** sponsorship configured at 50% delivering 20-30%.

**Probable causes:**
1. Another active sponsorship on the same inventory consumes part of the available percentage
2. The ad unit has insufficient traffic in the period (the % is calculated on real impressions, not a fixed total)
3. The sponsorship targeting excludes part of the inventory where the traffic is
4. Creative not approved for part of the ad units in the targeting

**Verification:**
- Review all active sponsorships on the same ad unit and sum their percentages
- Compare total ad requests for the ad unit with sponsorship impressions
- Review the delivery breakdown by ad unit if the targeting includes multiple

**Resolution:** if the sum of sponsorships exceeds 100%, readjust. If traffic is insufficient, renegotiate the goal or expand the targeting.

**Evidence: Probable**

---

### AP-003 — Standard line item not reaching goal

**Symptom:** standard campaign that ends its period without having delivered 100% of contracted impressions.

**Probable causes:**
1. Incorrect forecasting at the time of booking — more inventory was sold than available
2. A higher-priority campaign appeared after booking that consumes the same inventory
3. Site traffic drop during the period (seasonality, technical failure)
4. Pacing configured as "even" but real traffic is irregular → low-traffic days are not recovered

**Verification:**
- Review the "Expected" vs "Actual" delivery report on the line item
- Check whether there are sponsorships or higher-priority standards activated after booking
- Compare real traffic during the period with the historical data used in forecasting

**Resolution:** if it is an overselling error, extend dates or compensate with additional impressions. If it is a priority change made afterwards, review policies for managing changes to reserved inventory.

**Evidence: Probable**

---

### AP-004 — Price Priority not delivering despite active bids

**Symptom:** Price Priority line items (Prebid, Open Bidding) with active bids but no impressions in GAM.

**Probable causes:**
1. A higher-priority sponsorship or standard is consuming 100% of available inventory
2. The line item floor price is above the CPM of the bids
3. Prebid key-values (`hb_pb`) are not reaching GAM — problem in the GPT/Prebid integration
4. Price Priority line items are not correctly configured for the price range of the bids

**Verification:**
- Check if there are higher-priority line items active on the same inventory
- Check `pbjs.getEvents()` to verify that targeting is being set in GPT
- Verify that the line item price range covers the real CPM of the bids

**Resolution:** → For detailed diagnosis of the Prebid↔GAM integration, load `prebid-adtech`.

**Evidence: Probable** (requires telemetry from the specific case)

---

## Targeting and key-values

### AP-005 — Key-value mismatch due to case sensitivity or encoding

**Symptom:** line item with targeting `category=sports` with no impressions, even though traffic to the sports section is normal.

**Cause:** GAM key-value targeting is case-sensitive. The value sent from the page must match exactly: uppercase/lowercase, accents, spaces, encoding.

**Real examples:**
- GAM: `section=sports` / Page: `section=Sports` → no match
- GAM: `type=article` / Page: `type=articulo` (different encoding) → no match
- GAM: `author=john smith` / Page: `author=john+smith` (space encoding) → no match

**Verification:**
```javascript
// In the browser console, on the affected page:
googletag.pubads().getTargeting('section'); // returns the actual value sent
```

**Resolution:** correct the value in the page tag or in the line item targeting so they match exactly.

**Evidence: Verified**

---

### AP-006 — Audience segment targeting with empty or insufficient segment

**Symptom:** line item with audience targeting and no impressions. The segment appears as active in GAM.

**Probable causes:**
1. The segment was just created and has not had time to populate (can take 24-48h)
2. The segment has very short expiration rules and users have expired
3. The real size of the segment is below the minimum for targeting (privacy)
4. The segment is being built correctly but the traffic that meets the rules is insufficient for the goal

**Verification:**
- Check the segment size in the GAM interface (Audience → Segments → estimated size)
- Verify that the page tag is sending the signals needed to populate the segment
- Check the segment creation date vs the line item start date

**Resolution:** wait for the segment to populate, broaden the segment rules, or combine with additional targeting to increase reach.

**Evidence: Probable**

---

## Forecasting and overselling

### AP-007 — Positive availability check but actual delivery much lower

**Symptom:** forecasting showed sufficient availability but the line item does not deliver as expected.

**Probable causes:**
1. The forecasting was done with broader targeting than the final line item
2. New higher-priority campaigns were activated between the check and the launch
3. Real traffic during the period is lower than the historical data used in the projection (seasonality, drop)
4. The forecasting did not account for frequency capping that limits actual reach

**Verification:**
- Compare the availability check targeting with the real line item targeting
- Review which higher-priority campaigns were activated between the check and the launch
- Compare historical forecasting traffic with actual traffic during the period

**Resolution:** to systematically reduce this problem: run the availability check as close as possible to the launch, with the exact line item targeting, and add a safety margin of 10-15% on available inventory when selling.

**Evidence: Probable**

---

## Discrepancies

### AP-008 — GAM vs SSP discrepancy >20% consistently

**Symptom:** the SSP consistently reports 20-30% more impressions than GAM.

**Most probable cause:** the SSP is counting bid wins, not verified impressions. The difference between "bid won" and "impression served" includes: creative timeouts, invalid traffic filtered by GAM, and won bids where the publisher served a fallback.

**Verification:**
- Ask the SSP to differentiate in its reporting between "won bids" and "impressions served"
- Compare timing: is the discrepancy higher at certain times (peak traffic with more timeouts)?
- Verify the timeout configured for that SSP in GAM → reducing timeout increases the discrepancy

**Resolution:** if the SSP is reporting won bids as impressions, request a change in their reporting methodology. If the issue is timeout, assess adjusting the timeout vs the impact on fill rate.

**Evidence: Probable** (requires comparative data from the SSP)

---

### AP-009 — Revenue discrepancy between GAM and SSP

**Symptom:** impressions match but revenue differs significantly.

**Probable causes:**
1. GAM reports net revenue (after Google fee) / SSP reports gross → difference equals the fee %
2. Applied exchange rates differ (if the deal is in a currency other than the reporting one)
3. The SSP includes bonuses or adjustments that GAM does not reflect

**Verification:**
- Verify whether both systems report gross or net
- Confirm the reporting currency in both systems
- Review whether there are manual adjustments in the SSP

**Resolution:** align the reporting methodology (gross vs net) with the SSP before comparing. Document the agreement in the deal contract.

**Evidence: Probable**

---

## Roadblocking

### AP-010 — "All" roadblock delivering nothing

**Symptom:** roadblock campaign configured as "All" with 0 impressions.

**Probable causes:**
1. One of the roadblock ad units is not present on the pages in the targeting
2. A higher-priority line item is monopolizing one of the roadblock ad units
3. Creative sizes do not cover all ad units in the roadblock
4. Geo/device targeting excludes pages where all ad units appear simultaneously

**Verification:**
- Confirm that ALL roadblock ad units appear together on the pages in the targeting
- Verify that no roadblock ad unit is at 100% with another line item
- Review the forecasting for the full roadblock (not for each ad unit individually)

**Resolution:** switch to "As many as possible" if the campaign can tolerate partial delivery, or resolve the block on the problematic ad unit.

**Evidence: Probable**

---

## Creative trafficking

### AP-011 — HTML5 creative rejected or behaving incorrectly

**Symptom:** HTML5 creative uploaded but GAM rejects it, or it serves but click tracking does not work correctly.

**Common causes:**
1. The .zip file does not have `index.html` at the root (it is in a subfolder instead)
2. The click tag does not use GAM's `clickTag` variable but a hardcoded href → clicks are not tracked
3. The zip exceeds the size limit (200KB by default in GAM free)
4. Assets referenced in the HTML with absolute paths instead of relative → they do not load

**Correct click tag implementation in HTML5:**
```javascript
// In the banner HTML:
var clickTag = ""; // GAM overwrites this variable with the destination URL
document.getElementById('cta-button').addEventListener('click', function() {
  window.open(clickTag, '_blank');
});
```

**Verification:** open the .zip locally, verify that `index.html` is at the root and that all assets load with relative paths.

**Evidence: Verified**

---

## Notes on this file

This file is updated with new real cases. To add an entry:
- Format: observed symptom → verified or probable cause → how it was verified → how it was resolved
- Indicate the evidence level (Verified / Probable / Requires data)
- Mark with `[360]` if the case is specific to GAM 360
- If the case crosses with another skill (`prebid-adtech`, `video-adtech`), indicate it explicitly

---

## Programmatic yield and pricing rules

### AP-012 — Low programmatic fill with no apparent cause in the line item

**Symptom:** programmatic inventory with low fill. No direct line items monopolizing the inventory. Open Bidding or AdX apparently active.

**Probable causes (ordered):**
1. Active pricing rule with a floor too high for available demand → bids arrive but are below the floor
2. Ad unit not included in any yield group → Open Bidding demand cannot bid on it
3. Active category protection blocking legitimate demand in that sector
4. Competitive exclusion label excluding a significant volume of buyers

**Verification:**
- GAM report with "Pricing rule" dimension → see which floor is being applied and compare with actual bid CPMs
- Verify that the ad unit is included in an active yield group with the correct bidders
- Review active protections and labels on the affected inventory

**Resolution:** calibrate the floor with the real CPMs of the inventory, verify yield group coverage, and audit protections and labels.

**Evidence: Probable** (requires reporting data from the specific case)

---

### AP-013 — Active PG deal with 0 impressions

**Symptom:** Programmatic Guaranteed deal configured in GAM, line item in active status, but no impressions.

**Probable causes (ordered):**
1. The buyer has not activated the deal in their DSP → GAM receives no bid requests for that deal
2. The buyer's creatives are pending approval or have been rejected in GAM
3. Targeting mismatch: the line item targeting in GAM is not compatible with the bid requests from the buyer's DSP
4. Active pricing rule interfering with the PG deal price

**Verification:**
- Confirm with the buyer that the deal is active in their DSP and that they are receiving bid requests
- Review the status of the creatives associated with the deal in GAM
- Use the line item's Troubleshoot tab to verify eligibility
- Review active pricing rules on that inventory

**Resolution:** direct coordination with the buyer is frequently necessary — the problem usually lies in the DSP, not in GAM.

**Evidence: Probable**

---

## Troubleshooting tools

### AP-014 — Diagnosis based on assumptions without using native tools

**Symptom (operational anti-pattern):** line item configuration is modified (targeting, priority, creatives) without having used the Troubleshoot tab or Publisher Console to verify the real cause.

**Why this is an anti-pattern:** changes without prior diagnosis may resolve the superficial symptom but not the root cause, or introduce new problems (overlap with other campaigns, unintended priority change).

**Correct protocol:**
1. Troubleshoot tab → identify whether the line item is eligible and why it is not winning
2. Publisher Console on the affected page → verify real key-values, requested sizes, and which line item is winning
3. GAM report with relevant dimensions → cross-check hypotheses with historical data
4. Only then: propose configuration changes with the identified cause

**Evidence: Verified** (practice documented in GAM Help Center)
