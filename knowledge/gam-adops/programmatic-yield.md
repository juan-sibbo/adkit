# Programmatic Yield — PG, Deals, Open Bidding, Yield Groups, Pricing Rules, Labels

## Deal types in GAM

### Programmatic Guaranteed (PG) `[360]`

Deal in which the buyer commits to purchasing a fixed volume of impressions at a fixed CPM, with specific targeting. From GAM's perspective, a PG deal behaves similarly to a guaranteed Standard line item, but the demand comes from a DSP.

**Configuration in GAM:**
- Created as a deal in Inventory > Deals, then associated with a Programmatic Guaranteed line item
- The line item inherits Standard (guaranteed) priority
- The creative is uploaded by the buyer via their DSP — the publisher only configures targeting and size

**Common issues:**
- Deal not delivering: verify the buyer has activated the deal in their DSP and that creatives are approved in GAM
- CPM not honoring the agreed price: check if there is an active pricing rule interfering with the PG floor (pricing rules should not override a PG price, but they can interact with the floor if misconfigured)
- Targeting mismatch: the line item targeting in GAM and the targeting configured in the buyer's DSP must be compatible — if the DSP does not send bid requests matching GAM targeting, there is no delivery

### Preferred Deals (PD)

The publisher offers the buyer preferential access to inventory at a fixed CPM, before the open auction, but without volume guarantee. The buyer can reject the impression.

**Priority behavior:**
- PD evaluates before Open Bidding and the open auction, but after guaranteed line items (Sponsorship, Standard, PG)
- If the buyer does not bid on a specific impression, it passes to the next waterfall stage

**Common issues:**
- Low fill in PD: the buyer is rejecting most impressions. Causes: deal targeting too broad for the buyer's real interest, audience frequency exhausted, or the agreed CPM is above the value the buyer actually assigns to the inventory
- Revenue below expectations: PD CPM is fixed, but if fill is low, actual revenue may be lower than what the open auction would have generated on those impressions

### Private Auction (PA)

Private auction where the publisher invites selected buyers to bid on their inventory with a minimum floor price. Unlike PD, the price is not fixed — the buyer with the highest bid wins.

**Key differences vs PD:**
- PA is an auction → variable price above a floor. PD is a fixed price → the buyer accepts or rejects.
- Multiple buyers can participate in a PA; a PD is one-to-one.

**Configuration in GAM:**
- Created as a deal in Inventory > Deals with type "Private Auction"
- The deal floor price must be consistent with the floor of active pricing rules on that inventory

---

## Open Bidding `[360]`

GAM 360 mechanism that allows third-party SSPs to bid in real time alongside AdX in the GAM auction, without requiring client-side header bidding implementation.

**Difference from Prebid header bidding:**
- Open Bidding is server-side: SSPs receive the bid request from GAM and respond directly
- Latency is lower than client-side Prebid, but the publisher has less control over each bidder's configuration
- For Prebid↔GAM integration (client-side) → `prebid-adtech`

**Yield groups in Open Bidding:**
Yield groups define which SSPs (exchange bidders) participate in the auction for a given inventory. A yield group groups ad units + exchange bidders and defines participation conditions.

**Common issues:**
- Exchange bidder not delivering: verify the yield group includes the affected ad units and the exchange bidder has the correct format enabled (banner, video, native)
- Low Open Bidding CPM: may indicate the active pricing rule floor is too high for that bidder, or the bidder has no qualified demand for that inventory
- Open Bidding vs SSP discrepancy: see `discrepancy.md`

---

## Yield Groups

A yield group is the configuration unit that connects inventory (ad units) with programmatic demand (Open Bidding exchange bidders, or AdSense). It defines which demand can bid on which inventory.

**Structure:**
- Included ad units
- Enabled exchange bidders
- Format (display, video, native)
- Additional targeting (optional: geo, device)

**Critical check:** if an ad unit is not in any yield group, Open Bidding demand cannot bid on it. This generates unfulfilled impressions or exclusive dependence on AdX/direct line items.

**Common risk:** yield group configured with a subset of ad units (by error or historical inheritance) that leaves out inventory with relevant traffic.

---

## Pricing Rules

Pricing rules define price floors for programmatic traffic. GAM evaluates active pricing rules before the auction and applies the corresponding floor.

**Types:**
- **Network-level:** applied to all inventory on the network
- **Ad unit-level:** applied to specific ad units and their children
- **Specific targeting:** can include geo, device, deal ID conditions, etc.

**Evaluation order:** when multiple applicable pricing rules exist, GAM applies the most specific one. If there is a conflict between rules of the same specificity level, behavior can be unpredictable — document and avoid overlaps.

**Common risks:**
- Floor too high that blocks legitimate programmatic demand → low fill with no apparent cause in the line item
- Active pricing rule on PG or PD inventory that interferes with the deal price
- Inherited pricing rules from old configurations that nobody has reviewed and that are blocking revenue
- Floor configured in USD but traffic in a different currency with no conversion configured

**Verification:** in the GAM report, use the "Pricing rule" dimension to see which rule was applied to each impression and whether the floor is blocking bids.

---

## Labels, Competitive Exclusions and Protections

### Labels

Labels are tags assigned to advertisers, orders, or line items to control what can coexist on the same page or session.

**Primary use:**
- **Competitive exclusion:** prevent two advertisers in the same sector (e.g. two airlines, two banks) from appearing on the same page or in the same ad break
- The label is created in the network, assigned to the affected advertisers/orders, and the exclusion rule is configured

**Configuration:**
1. Create the label (Inventory > Labels)
2. Assign the label to the advertiser or to the order/line item
3. Configure the exclusion rule: how many ads with the same label can appear per page / per session

**Common risk:** incorrectly assigned labels that exclude legitimate demand. For example, an "automotive" label assigned to a tire advertiser that unintentionally excludes another spare parts advertiser that should not be considered a competitor.

### Advertiser/Category Protections

GAM allows configuring publisher-level protections to prevent certain advertiser categories from serving on their inventory (alcohol, gambling, politics, etc.).

**Important distinction:**
- **Labels/competitive exclusion:** controls coexistence between specific advertisers on the same page
- **Category protections:** blocks entire categories of programmatic demand from the publisher's inventory

**Common risk in troubleshooting:** an active category protection can explain low fill on a specific ad unit with no configuration error in the line item.

---

## Diagnostic programmatic reporting

Key dimensions and metrics for programmatic yield diagnosis in GAM:

| Dimension / Metric | Purpose |
|---|---|
| **Deal ID** | Break down delivery and revenue by specific deal |
| **Buyer network** | Identify which DSP or network is buying |
| **Pricing rule** | See which floor was applied on each impression |
| **Yield group** | Verify which yield group intermediated the impression |
| **Programmatic channel** | Differentiate Open Auction, PD, PA, PG, Open Bidding |
| **Ad request CPM** | Estimated CPM of the ad request (useful for calibrating floors) |
| **Unfilled impressions** | Impressions without a winner — break down by cause if available |
| **Match rate** | % of ad requests with at least one bid — low fill may originate here |
| **Requested ad sizes** | Sizes the slot is actually requesting — verify consistency with creatives |

**Note:** not all these dimensions are available in GAM non-360. The depth of programmatic reporting is significantly greater in GAM 360.

---

## Deal troubleshooting in GAM

### Deal not delivering at all

1. Verify the deal is active in both GAM and the buyer's DSP
2. Check that the buyer's creatives are approved in GAM
3. Verify the line item targeting in GAM is compatible with the DSP's bid requests
4. Review whether there are active pricing rules setting a floor above the deal CPM
5. Check labels/protections that could exclude the deal's advertiser

### Deal delivers but actual CPM is below the agreed amount

In PD and PA, price can vary if the buyer is bidding below the agreed CPM. In PG the price should be fixed — if there is variation, review whether revenue share adjustments are being applied.

### Open Bidding with unexpectedly low fill

1. Verify the yield group includes the affected ad units
2. Check if the active pricing rule floor is too high for that exchange bidder
3. Check the "buyer network" dimension in reporting to see if there are bids but they are not winning
4. Verify that the format (banner/video/native) is enabled in the yield group for that bidder
