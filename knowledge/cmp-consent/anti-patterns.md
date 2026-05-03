# Anti-patterns — CMP Consent Flows

Symptom → diagnosis table for quick diagnosis. Expand with knowledge-capture entries.

---

## Signal drop and TC string

### Empty TC string in bid requests even though the user accepted

**Type:** anti-pattern

**Symptom:** Prebid adapters send bid requests without a TC string or with `gdprApplies: false` on European traffic where the user did interact with the banner.

**Context:** Stack with Prebid.js + consentManagement module + Didomi or similar. Occurs especially on first load.

**Root cause:** Race condition — Prebid launches the auction before the `consentManagement` module receives the `tcloaded` event from the CMP. This occurs when the CMP script loads after Prebid or when `auctionDelay` is 0 or very low.

**Resolution:**
1. Verify that the CMP script loads before Prebid in the HTML.
2. Increase `auctionDelay` in `consentManagement` to a value that reflects the real measured latency (see `references/consent-latency.md`).
3. Verify with `pbjs.getEvents()` that `auctionInit` occurs after the `tcloaded` event.

**Verification:** `__tcfapi('getTCData', 2, cb)` in console → `tcString` must be present and non-empty.

**Evidence level:** Verified — documented and reproducible pattern.

**Metadata:** Captured: 2025-04 · Versions: Prebid.js >= 5.x, TCF 2.x · Status: Active

**Tags:** tc-string, race-condition, auctionDelay, consentManagement, signal-drop

---

## Vendor list configuration

### Active vendors in the auction do not receive consent signal

**Type:** anti-pattern

**Symptom:** A new SSP or DSP does not bid or bids without a TCF signal. The publisher assumes the vendor is configured but it does not appear in the TC string as a vendor with consent.

**Context:** Vendor recently incorporated into the stack. Not in the GVL or added after the CMP's vendor list was configured.

**Root cause:** The vendor is not in the CMP's active vendor list. This may be because: (1) it is not in the GVL and was not added as a custom vendor, or (2) it is in the GVL but the publisher's vendor list was not updated to include it.

**Resolution:**
1. Verify whether the vendor is in the GVL (iabeurope.eu/vendor-list/).
2. If it is in the GVL: update the CMP's vendor list to include it. `[Didomi]` In Didomi → Vendors → search by name or ID.
3. If it is not in the GVL: add as a custom vendor with the purposes declared by the vendor.

**Evidence level:** Verified.

**Metadata:** Captured: 2025-04 · Status: Active

**Tags:** vendor-list, gvl, custom-vendor, signal-drop, new-ssp

---

## Consent latency

### Auctions launched without consent for new users with `allowAuctionWithoutConsent: true`

**Type:** behavioral-note

**Symptom:** In log analysis, a % of auctions (especially from new users) launches with an empty TC string or without `purposeConsents`. TCF-compliant bidders return no-bid or reduced CPM.

**Context:** Publisher with significant European traffic, high % of new users or Safari/ITP, `allowAuctionWithoutConsent: true` in Prebid config.

**Root cause:** Expected behavior when `auctionDelay` expires before the user interacts with the banner. This is not a bug — it is a configuration decision with compliance and revenue implications.

**Resolution:** There is no single resolution — it is a trade-off:
- Increase `auctionDelay` → more users wait → improves fill but increases page latency
- Reduce banner friction → more users accept faster → reduces the window of auctions without consent
- Review whether `allowAuctionWithoutConsent: true` is the correct policy with the legal team

**Evidence level:** Verified — behavior documented in Prebid docs.

**Metadata:** Captured: 2025-04 · Status: Active

**Tags:** consent-latency, allowAuctionWithoutConsent, new-users, auction-timing, compliance

---

## Legitimate interest

### CPM drops after enabling "maximum privacy" in CMP configuration

**Type:** anti-pattern

**Symptom:** The publisher enables a "maximum privacy" or "conservative mode" option in the CMP and programmatic CPM drops significantly even though the consent rate has not changed.

**Root cause:** The "maximum privacy" option globally disables `legitimateInterests` for all purposes. Many vendors (especially DSPs) use LI as the lawful basis for P2-P7. By disabling LI, they receive a negative signal for those purposes even though the user has not actively rejected them.

**Resolution:**
1. Decode the TC string before and after the change — compare `purposeLegitimateInterests`.
2. Identify which vendors use LI as the lawful basis for the affected purposes.
3. Assess with legal whether disabling LI globally is a requirement or a choice.
4. If it is a choice, quantify the CPM impact before keeping it active.

**Evidence level:** Verified — mechanism documented in TCF 2.x spec.

**Metadata:** Captured: 2025-04 · Versions: TCF 2.x · Status: Active

**Tags:** legitimate-interest, CPM-drop, publisher-restrictions, tcf, lawful-basis
