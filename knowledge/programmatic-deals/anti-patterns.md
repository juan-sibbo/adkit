# Anti-patterns — Programmatic Deals (publisher-side)

Entries organized by funnel step. Each entry in knowledge-capture format.

---

## Step 3 — Insufficient eligible inventory

### AP-PD-301
- **Type:** Anti-pattern
- **Symptom:** Active deal, buyer confirmed listening, but spend well below the commercially promised volume
- **Context:** PMP or PG negotiated with "high" estimated volume, buyer starts buying but at a rate well below expectations
- **Root cause:** The forecast or commercial expectation was based on the publisher's broad inventory. The deal ended up restricted to a narrow combination of format + geo + device + ad unit + domain that actually represents a small fraction of that inventory. The buyer listens correctly, but there is very little to buy.
- **Resolution:** Request an avail estimate from the SSP with exactly the deal filters (not a general estimate). Compare with the promised volume. If there is a gap → renegotiate the deal scope or broaden the targeting.
- **Verification:** SSP avail estimate with exact filters << promised volume
- **Evidence level:** Verified
- **Tags:** `avails`, `forecasting`, `commercial-mismatch`, `supply`, `PMP`, `PG`

### AP-PD-302
- **Type:** Anti-pattern
- **Symptom:** Deal in the correct SSP, but buyer says they "don't see enough requests" and the SSP also sees few bids
- **Context:** Publisher with inventory across multiple SSPs; deal created in the wrong SSP for the target inventory
- **Root cause:** The publisher's inventory that the buyer is actually interested in lives in a different SSP from where the deal was configured. The deal is technically active but pointing to the wrong place.
- **Resolution:** Identify which SSP the target inventory actually lives in (by volume, by floor, by format type) and move or replicate the deal there.
- **Verification:** Avails in the SSP where the deal is are minimal; avails in the correct SSP are significant
- **Evidence level:** Probable
- **Tags:** `avails`, `SSP-selection`, `supply`, `multi-SSP`

---

## Steps 4–5 — Buyer activation / supply path

### AP-PD-401
- **Type:** Anti-pattern
- **Symptom:** Active deal in SSP, zero bids, buyer says they are "looking at it" or "have it in setup"
- **Context:** Newly activated PMP or PG deal, any buyer/DSP
- **Root cause:** The deal is in "Pending" or "Saved" status in the buyer's DSP — configured but without an active insertion order or line item with budget pointing to it. The SSP cannot see the DSP's internal state. "We have it activated" does not always mean the same thing for the SSP and the buyer.
- **Resolution:** Ask the buyer for a screenshot or explicit confirmation of: (1) deal in "Active" status — not just "added", (2) active line item / IO with assigned budget pointing to the deal, (3) buyer's line item dates match the deal dates.
- **Verification:** Buyer confirms status with evidence → bids start appearing within <2h
- **Evidence level:** Verified
- **Tags:** `buyer-activation`, `DSP`, `pending`, `TTD`, `DV360`, `Xandr`

### AP-PD-402
- **Type:** Anti-pattern
- **Symptom:** SSP sees bid requests with the correct deal ID, but buyer says they "don't receive requests" or "volume is very low"
- **Context:** Any SSP/DSP combination
- **Root cause:** The buyer is targeting the SSP but not the correct seat, or the exchange/supply source is not enabled in the DSP for that publisher. The deal reaches the buyer only if the buyer is listening to exactly that seat on that SSP. A DSP may have multiple integrations with the same SSP.
- **Resolution:** Confirm with the buyer the exact seat ID the SSP is using to send the deal. Compare with the seat the buyer has configured. For TTD: verify Merchant ID. For DV360: verify the deal is associated with the correct IO that has that inventory source enabled.
- **Verification:** Buyer updates seat or supply source → request volume increases in buyer UI
- **Evidence level:** Verified
- **Tags:** `buyer-activation`, `seat-id`, `supply-path`, `TTD`, `DV360`, `exchange`

### AP-PD-403
- **Type:** Anti-pattern
- **Symptom:** Active deal, buyer listening, but SSP reports very few bid requests sent to the buyer (the problem is before the buyer)
- **Context:** Deal with very specific targeting in the SSP
- **Root cause:** The deal targeting in the SSP is filtering almost all inventory before sending requests to the buyer. Too specific geo, ad unit restricted to one only, domain list too short, or limited time slot. The buyer never sees enough inventory because the SSP does not offer it to them.
- **Resolution:** Review the deal targeting in the SSP and gradually broaden the most restrictive filters. Ask the SSP for an impression breakout by targeting parameter to identify which one is cutting the most.
- **Verification:** Targeting broadened → increase in bid requests sent to buyer → increase in bids
- **Evidence level:** Verified
- **Tags:** `targeting`, `supply`, `bid-requests`, `SSP-config`

### AP-PD-404
- **Type:** Anti-pattern
- **Symptom:** Deal ID confirmed in the SSP, buyer says they have it configured, but no bids appear; when crossing the ID character by character there is a discrepancy
- **Context:** Any SSP with alphanumeric Deal IDs (especially Triplelift, Magnite, IX, PubMatic); also in deals communicated by email or PDF
- **Root cause:** The copied Deal ID includes a trailing space, an em dash instead of a hyphen, an incorrect uppercase letter, or a special character altered by the email client. Deal IDs are case-sensitive and most SSPs do not normalize — one different character is a different ID.
- **Resolution:** Ask the SSP for the Deal ID in plain unformatted text (not PDF, not HTML email). Share it with the buyer in the same format. Ask the buyer to confirm the ID copied directly from their DSP.
- **Verification:** Buyer shows the Deal ID as it appears in their DSP → compare character by character with the SSP's
- **Evidence level:** Verified
- **Tags:** `deal-id`, `format`, `case-sensitive`, `buyer-activation`

### AP-PD-405
- **Type:** Anti-pattern
- **Symptom:** Active deal, correct seat, bid requests reaching the buyer, but buyer does not bid on specific publisher domains (the problem is selective by domain)
- **Context:** Buyers using domain allowlists or app bundle allowlists in their DSP (common in TTD, DV360, Xandr); publisher with subdomains, brand domains, or international domains not in the original deal
- **Root cause:** The buyer has a domain or app bundle allowlist in their DSP that does not include the exact domain the bid request is coming from. New domains, subdomains, or international variants (e.g., `brand.co.uk` vs `brand.com`) do not appear on the buyer's allowlist even though the publisher considers them the same inventory.
- **Resolution:** (1) Ask the SSP for the domain breakout in the deal's bid requests. (2) Confirm with the buyer whether they have an active allowlist and whether those domains are included. (3) Coordinate with the buyer to add the missing domains, or adjust the deal targeting to exclude domains the buyer does not accept.
- **Verification:** Buyer updates their allowlist → bids appear on previously blocked domains
- **Evidence level:** Probable
- **Tags:** `allowlist`, `domain-whitelist`, `app-bundle`, `TTD`, `DV360`, `buyer-side`

### AP-PD-406
- **Type:** Anti-pattern
- **Symptom:** Deal with apparently correct dates, but buyer reports the "deal is not active when they try to buy" or there are unexpectedly short delivery windows
- **Context:** Deals between publisher and buyer in different time zones; common in deals with Spanish/European publishers and buyers operating in UTC or US timezones
- **Root cause:** The deal was configured with dates in the SSP's timezone (usually UTC) but the buyer interpreted the dates in their local timezone, or vice versa. A deal starting January 1st at 00:00 UTC has been active for an hour for a buyer in CET, but a deal ending December 31st at 23:59 CET has already expired in UTC. The difference can make the deal appear inactive during the publisher's peak traffic hours.
- **Resolution:** Confirm with the SSP the timezone in which the deal dates are configured. Communicate the equivalent dates in the buyer's timezone. If there is a discrepancy, adjust the deal dates.
- **Verification:** SSP timezone confirmed → dates recalculated → buyer confirms they match their expectations
- **Evidence level:** Probable
- **Tags:** `timezone`, `dates`, `buyer-activation`, `UTC`, `CET`

---

## Step 6 — Buyer bids but at low volume or low CPMs

### AP-PD-601
- **Type:** Anti-pattern
- **Symptom:** Low bid rate, buyer bids occasionally but consistently below the floor CPM
- **Context:** PMP with floor configured in the SSP
- **Root cause (a):** Floor calculated on net CPM (publisher revenue) but communicated to the buyer as gross CPM — or vice versa. The SSP fee difference creates a systematic discrepancy. The buyer thinks they are bidding above the floor, but the SSP applies the floor on a different basis. Example: €3.00 net floor with 20% SSP fee → the buyer needs to bid €3.75 gross to beat the floor, but if they were told "the floor is €3.00" they will bid €3.10 and consistently lose.
- **Root cause (b):** The floor is not competitive with the real market for that inventory with that buyer. The buyer has an internal bid ceiling lower than the deal floor.
- **Resolution (a):** Confirm with the SSP whether the configured floor is net or gross. Recalculate the equivalent floor on the basis the buyer uses. Communicate the floor to the buyer on the same basis the SSP uses to evaluate bids.
- **Resolution (b):** Check market CPMs for that inventory (open auction or similar deals). Adjust the floor or renegotiate with the buyer.
- **Verification:** Win rate near zero with many bids recorded in the expected CPM range
- **Evidence level:** Probable
- **Tags:** `floor`, `CPM`, `net-gross`, `win-rate`, `pricing`

### AP-PD-602
- **Type:** Anti-pattern
- **Symptom:** Buyer bids well in bid rate, acceptable win rate, but total spend is low
- **Context:** Deal with high potential volume but actual spend much lower
- **Root cause:** The buyer has a low daily budget assigned to the deal, restrictive pacing, or aggressive frequency caps limiting the number of impressions per user. The deal works technically but is limited by the buyer's internal DSP parameters.
- **Resolution:** Ask the buyer for explicit confirmation of: daily budget assigned to the deal, pacing configured, frequency cap if applicable, and whether there are KPI goals (e.g., CTR, viewability) that are slowing spend due to performance. Some DSPs (Amazon DSP, DV360, TTD) expose bid loss reason codes in their reports — ask the buyer to share them if available: frequency limit, viewability threshold, or audience mismatch codes are direct diagnostics without needing to infer.
- **Verification:** Buyer reviews internal parameters → spend increases in 24-48h; or bid loss reason codes identify the exact restriction
- **Evidence level:** Probable
- **Tags:** `budget`, `pacing`, `frequency-cap`, `DSP-internals`, `bid-loss-reasons`, `buyer-side`

---

## Steps 7–9 — Delivery / Creative / GAM

### AP-PD-701
- **Type:** Anti-pattern
- **Symptom:** SSP reports wins correctly, but GAM does not record impressions for that deal
- **Context:** Any deal with a corresponding line item in GAM
- **Root cause:** The PG/PD line item in GAM is paused, has the wrong deal ID mapped, or the yield group does not have the SSP/buyer as an eligible source. The SSP wins the auction, but GAM rejects the impression or does not associate it with the correct line item.
- **Resolution:** Verify in `gam-adops`: (1) active line item with correct deal ID, (2) active yield group with the SSP as a source, (3) discrepancy between SSP wins and GAM impressions as a diagnostic signal.
- **Verification:** GAM Ads Inspector on a page view with the active deal — confirm whether the deal creative is being served
- **Evidence level:** Verified
- **Tags:** `GAM`, `line-item`, `yield-group`, `discrepancy`, `gam-adops`

### AP-PD-702
- **Type:** Anti-pattern
- **Symptom:** Active deal, buyer confirmed listening with bids, but zero or very few impressions delivered
- **Context:** Any SSP with a creative review process; especially common for non-standard formats (native, outstream, skin, high impact)
- **Root cause:** The buyer's creative has not passed the SSP's editorial review process, is rejected for ad quality reasons, or is blocked by content categories. The SSP wins the auction but blocks the impression before serving it. There is not always an active notification to the publisher — the deal simply does not deliver without apparent explanation.
- **Resolution:** Check the creative status in the SSP panel for the deal. If it is "pending review" or "rejected", identify the reason and coordinate with the buyer to fix it or escalate with the SSP account manager.
- **Verification:** SSP creatives panel → approval status; rejection reason if any
- **Evidence level:** Verified
- **Tags:** `creative-review`, `ad-quality`, `brand-safety`, `native`, `outstream`

**SSPs with mandatory or frequent creative review:**

| SSP | Affected formats | Typical review time | Notes |
|---|---|---|---|
| Triplelift | Native (all) | 24-72h | Mandatory editorial review before first delivery |
| Teads | Outstream video, inRead | 24-48h | Video quality and landing page review |
| Xandr | Display, video, native | 24-48h | Automated ad quality + manual review if flagged |
| PubMatic | Display, video | 24h | Automated brand safety; categories blocked by publisher |
| Magnite | Display, video, CTV | 24h | Automated brand safety; some publishers have their own lists |
| Adagio | Display, outstream | 24-48h | Attention-compatibility review of the format |

### AP-PD-703
- **Type:** Anti-pattern
- **Symptom:** SSP records win correctly, GAM does not record impression, no obvious rendering error on the page — the win "disappears" between the SSP and GAM
- **Context:** Any deal, especially on inventory with active SafeFrame or cross-domain restrictions by the publisher
- **Root cause:** The SSP injects a tracking or rendering wrapper (JS, iframe, pixel) that GAM's container blocks or does not execute. SafeFrame restricts access to the DOM and external resources — a wrapper that works in an open iframe fails in SafeFrame without a visible error to the publisher. Also occurs when the SSP injects resources from domains not authorized by the publisher's CSP (Content Security Policy).
- **Resolution:** (1) Force rendering of the deal creative on a test page under the same conditions (SafeFrame / friendly iframe). (2) Open DevTools → Network → look for blocked resources (403/404) or CORS/CSP errors. (3) Look for cross-domain JS errors in the console. (4) If the problem is SafeFrame: coordinate with the SSP to make the creative compatible, or evaluate whether the ad unit can be served in a friendly iframe for that specific deal.
- **Verification:** Rendering test on an isolated page → JS error or blocked resource identified in DevTools
- **Evidence level:** Verified
- **Tags:** `creative-wrapper`, `SafeFrame`, `CSP`, `rendering`, `discrepancy`, `SSP-GAM`

---

## Step 10 — Commercial problem, not technical

### AP-PD-1001
- **Type:** Anti-pattern
- **Symptom:** Deal technically correct at all steps, buyer listening and bidding, but total spend persistently low weeks after activation
- **Context:** PMP or PG signed, complete setup, but no volume scaling
- **Root cause:** The buyer accepted the deal commercially but without a real volume commitment, or was doing a limited test. The problem is not technical — it is a mismatch between what the publisher understood from the deal and the buyer's real intent. This happens frequently with deals signed under commercial pressure or with buyers who are "testing the inventory".
- **Resolution:** Separate technical validation (the deal works) from commercial validation (the buyer intends to spend). Escalate to the commercial conversation: what is the actual budget assigned? Is there a CPM or performance goal that is not being met? Is the buyer waiting to see improvements in some metric before scaling?
- **Verification:** The problem disappears when the commercial commitment is clarified — not when a technical change is made
- **Evidence level:** Verified
- **Tags:** `commercial`, `buyer-commitment`, `test`, `mismatch`, `non-technical`

### AP-PD-1002
- **Type:** Anti-pattern
- **Symptom:** Buyer scales with similar publishers but not with this one
- **Context:** PMP deal with buyer active in the market
- **Root cause:** The buyer has a specific restriction for the publisher: brand safety, blocked content categories, viewability or attention score below their threshold, or the inventory is simply not performing well (CTR, conversions) and the DSP is optimizing away.
- **Resolution:** (1) Check with the SSP whether there are creative rejection or brand safety reasons specific to the publisher. (2) Ask the buyer for performance metrics for this publisher's inventory vs others. (3) If it is a DSP with automatic optimization: the buyer may need to adjust their bidding strategy or give the algorithm more time.
- **Verification:** Buyer shares performance data → the failing metric is identified
- **Evidence level:** Probable
- **Tags:** `brand-safety`, `performance`, `viewability`, `attention`, `DSP-optimization`

---

## Supply chain — lateral checks (prerequisites)

### AP-SC-001
- **Type:** Anti-pattern
- **Symptom:** Deal not delivering with DV360 or The Trade Desk even though it appears correctly configured in the SSP
- **Context:** Any deal, buyers with strict ads.txt validation
- **Root cause:** ads.txt does not include the correct line for the SSP (incorrect account ID, absent, or misspelled domain). DV360 and TTD have strict validation — they silently reject inventory that does not pass.
- **Resolution:** Verify `https://[publisher-domain]/ads.txt`, find the SSP line, confirm account ID with the SSP dashboard. Fix and wait 24-48h for re-crawling.
- **Verification:** ads.txt validator → SSP line missing or with incorrect account ID
- **Evidence level:** Verified
- **Tags:** `ads.txt`, `DV360`, `TTD`, `supply-chain`, `prerequisite`

### AP-SC-002
- **Type:** Anti-pattern
- **Symptom:** Active deal in Xandr, zero delivery, everything seems correct
- **Context:** Deal with Xandr as SSP or Xandr as buyer DSP
- **Root cause:** The publisher's member ID in Xandr does not match the seller_id declared in ads.txt. Xandr validates the supply chain strictly — a mismatch silently blocks the deal.
- **Resolution:** Verify the publisher's member ID in the Xandr panel → confirm it matches the ads.txt line (`appnexus.com, [member_id], DIRECT`) and Xandr's sellers.json.
- **Verification:** Fix ads.txt with the correct member ID → propagation 24-48h → deal starts delivering
- **Evidence level:** Verified
- **Tags:** `Xandr`, `member-id`, `ads.txt`, `supply-chain`
