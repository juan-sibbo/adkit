# Routing Table — Knowledge Capture

Guide for determining which skill and file each finding belongs to. Load when the finding's domain is not immediately obvious or when it crosses more than one skill.

---

## Routing table by symptom/domain

### gam-adops → `references/anti-patterns.md`

Signals that the finding belongs here:
- The symptom occurs in the GAM interface or in GAM reports
- The root cause is in line item configuration, priorities, targeting, creatives, ad rules
- The diagnosis involves yield groups, pricing rules, labels, Open Bidding from GAM
- The discrepancy is analyzed from the GAM side (matched requests, unfulfilled, delivery curves)
- Tools involved: Troubleshoot tab, Ads Inspector, Publisher Console

### prebid-adtech → `references/anti-patterns.md`

Signals:
- The symptom is in the publisher-side auction (browser DevTools, Prebid debug mode)
- The root cause is in Prebid.js configuration (adUnits, bidders, timeout, price granularity)
- Involves GPT/googletag, macros `hb_pb`, `hb_adid`, `hb_bidder`
- TCF/CMP as consumer (module `consentManagement`, TC string reaching Prebid)
- A specific bid adapter behaves differently from what is documented

### video-adtech → `references/anti-patterns.md`

Signals:
- The symptom is visual: black screen, ad does not play, player stays on loading
- Malformed VAST response or specific error code
- IMA SDK does not initialize, event does not fire, content does not resume
- SSAI/DAI: ad is not inserted, markers not recognized
- CTV: rdid empty, ID signals not arriving, app-ads.txt invalid
- Environment: Smart TV, Roku, Fire TV, Android TV, HbbTV

### programmatic-deals → `references/anti-patterns.md`

*(Skill under construction — pre-routing for when it exists)*

Signals:
- The setup occurs in the SSP interface (not in GAM)
- OpenRTB deal object, fields `dealid`, `at`, `wseat`, `wadomain`
- Deal not delivering from the buyer side
- sellers.json, ads.txt, app-ads.txt as failed prerequisite

### cmp-consent-flows → `references/anti-patterns.md`

*(Skill under construction — pre-routing for when it exists)*

Signals:
- The CMP (Didomi, Usercentrics, OneTrust) does not show the banner or shows it incorrectly
- Malformed TC string or with incorrect purposes
- Measurable consent latency (the auction launches before the CMP resolves)
- Banner A/B test impacts consent rate unexpectedly
- GPP multi-state: jurisdiction detection fails

---

## Cross-skill cases

| Symptom | Primary skill | Secondary skill | Criterion |
|---|---|---|---|
| Direct line item competes with a Prebid bid that should not win | `gam-adops` | `prebid-adtech` | If root cause is in GAM priority → gam-adops; if in price bucket → prebid-adtech |
| Video ad rule in GAM blocks IMA SDK | `gam-adops` | `video-adtech` | Root cause in the rule → gam-adops; cause in the SDK → video-adtech |
| Open Bidding deal not delivering and discrepancy with SSP | `gam-adops` | `programmatic-deals` | GAM deal setup → gam-adops; SSP-side configuration → programmatic-deals |
| TCF string arrives empty to Prebid | `cmp-consent-flows` | `prebid-adtech` | If CMP does not generate the signal → cmp-consent-flows; if Prebid does not read it correctly → prebid-adtech |

In cross-skill cases: add the full entry in the primary skill and a brief cross-reference entry in the secondary.

---

## Cross-reference format

In the secondary skill, add a minimal entry pointing to the primary skill:

```
### [Brief symptom] — see [primary-skill]

**Cross-ref:** This pattern is documented in detail in `[primary-skill]/references/anti-patterns.md`.
**Routing signal:** [why it might be confused with this skill].
**Root cause:** belongs to [primary-skill] because [one-line reason].
```
