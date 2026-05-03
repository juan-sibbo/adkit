---
name: discrepancy-forensics
description: Forensic analysis of discrepancies between publisher/GAM, SSP, buyer/DSP, Prebid, video/CTV, app, and reporting datasets. Activate when numbers do not match, buyer reports low avails, SSP says X but GAM says Y, buyer/DSP sees little inventory, deal does not scale, bid requests disappear, fill/revenue differs across systems, or a closed evidence-ranked diagnosis is needed.
---

# Discrepancy Forensics

**Version:** 1.1.1  
**Status:** Stable — Funnel Forensics Lean  
**Last updated:** 2026-04-27  
**Domain:** GAM / SSP / DSP / Buyer / Prebid / CTV / Video / App / Reporting / Deals  
**Depends on:** gam-adops, programmatic-deals, prebid-adtech, video-adtech, cmp-consent-flows, incident-response, client-deliverable  
**Used by:** Agent α — Discrepancy Hunter

Isolates where the programmatic funnel breaks and produces evidence-ranked hypotheses, verification tests, and a forensic verdict.

---

# Core Principle

Do not start by asking “who is wrong?” Start by asking: **where does the opportunity disappear in the funnel?**

---

# Operating Modes

| Mode | Use when | Output |
|---|---|---|
| A. Dataset Mode | At least two datasets are available: GAM, SSP, DSP/buyer, Prebid logs, OpenRTB logs | Full cross-system calculation and breakpoint analysis |
| B. Questionnaire Mode | Raw data is missing or vendor support must answer | Directed questions by funnel stage |
| C. Hybrid Mode | Some numbers exist, but one party lacks exports | Partial calculation + support request package |

Default to Hybrid Mode if access is imperfect, which is common.

---

# 1. Intake & Scope Lock

Before calculations, lock the comparison scope.

```markdown
## Scope Lock

- Client / publisher:
- Deal ID / campaign / line item / order / buyer seat:
- GAM network ID:
- Ad unit path(s):
- Domain(s) / app bundle(s):
- Format: display / web video / CTV / app / native / audio
- Environment: web / mobile web / Android TV / tvOS / Samsung / LG / HbbTV / iOS / Android app
- Date range:
- Timezone:
- Currency:
- Revenue basis: gross / net / unknown
- Metric under dispute: avails / requests / bids / wins / impressions / fill / revenue / CPM / spend
- Parties compared: GAM / SSP / DSP / buyer / Prebid / player / logs
```

If scope is unclear, ask once:

```text
What exact period, inventory scope, format, and systems are we comparing? Without that, any discrepancy calculation may be misleading.
```

---

# 2. Data Quality Gate

Do not calculate until basic comparability is checked.

```markdown
## Data Quality Gate

- [ ] Same date range
- [ ] Same timezone or timezone conversion applied
- [ ] Same inventory scope
- [ ] Same deal ID / ad unit / domain / app bundle where relevant
- [ ] Same format/environment
- [ ] Currency aligned
- [ ] Gross vs net understood
- [ ] Revenue share / fees understood where relevant
- [ ] Impression definitions understood
- [ ] Reporting delay/freshness checked
- [ ] Test traffic separated from live traffic
- [ ] Sampling/filtering differences noted
```

If gate fails, label output:

- `PRELIMINARY — DATA NOT COMPARABLE`
- `PARTIAL — MISSING BUYER DATA`
- `PARTIAL — REPORTING DEFINITIONS NOT ALIGNED`
- `REQUIRES NORMALISATION BEFORE VERDICT`

---

# 3. Programmatic Funnel Map

Use this funnel to locate the break.

| Stage | Forensic question | Typical evidence |
|---|---|---|
| 1. Inventory | Does eligible inventory exist? | GAM availability, ad unit/domain/app traffic, content/device/geo breakdown |
| 2. Eligibility | Does inventory match targeting, consent, floors, deal, domain/app, device, geo, seat? | GAM targeting, UPR/floors, consent logs, app-ads.txt/ads.txt, deal config |
| 3. SSP Opportunity | Does SSP receive/generate the opportunity? | SSP bid requests, OpenRTB logs, adapter logs |
| 4. Buyer Visibility | Does buyer/DSP receive the request? | DSP request count, SSP-to-buyer logs, QPS/shaping data |
| 5. Buyer Bidding | Does buyer bid? | DSP bid rate, no-bid reasons, budget/targeting/creative status |
| 6. Auction / Win | Does bid win? | Win rate, clearing price, floor, competition, priority |
| 7. Render / Serve | Does won impression actually render/serve? | GAM impressions, VAST/player logs, creative render, timeout/error codes |
| 8. Reporting | Are systems counting the same thing? | Timezone, definition, gross/net, currency, freshness, deduplication |

---

# 4. Dataset Request — Mode A / Hybrid

Request only the missing datasets needed for the funnel stage under investigation.

```text
For the same period and scope, please provide any available exports:

1. GAM / publisher-side
- Ad requests / impressions served
- Unfilled impressions if relevant
- Revenue / CPM / fill rate
- Ad unit, device, geo, domain/app breakdown if available
- GAM timezone

2. SSP-side
- Bid requests
- Requests sent to buyer/DSP
- Bids received
- Wins
- Gross revenue
- Deal ID / seat / domain/app / device / geo breakdown if available

3. Buyer / DSP-side
- Bid requests received
- Bids sent
- Wins
- Spend
- No-bid/filtering reasons if available
- Campaign/seat/deal status

4. Logs if available
- OpenRTB samples
- Prebid debug output
- VAST/player errors
- HAR/network capture
- Request IDs / auction IDs
```

---

# 5. Normalisation

Apply and document normalisations before comparing.

| Dimension | Rule |
|---|---|
| Timezone | Convert all datasets to a shared timezone, preferably UTC, while preserving original reporting timezone |
| Date window | Align exact start/end; beware one-day offsets and daylight saving changes |
| Currency | Convert or label if mixed |
| Gross/net | Do not compare gross SSP revenue to net GAM revenue without marking basis |
| Impression definition | GAM render, SSP win, buyer served impression, billable impression may differ |
| Reporting delay | Mark if one system is real-time and another delayed |
| Scope | Exclude unmatched ad units, apps, domains, geos, formats, or test traffic |

Reference heuristics are not universal rules. Validate against platform, format, integration type, counting methodology, and commercial setup.

---

# 6. Core Metrics

Calculate only where denominator is non-zero and scope is aligned.

```text
GAM vs SSP impression discrepancy:
Δ_abs = GAM_impr - SSP_wins
Δ_pct = (GAM_impr - SSP_wins) / SSP_wins × 100

SSP vs Buyer request discrepancy:
Δ_abs = SSP_requests_to_buyer - Buyer_requests_received
Δ_pct = (SSP_requests_to_buyer - Buyer_requests_received) / SSP_requests_to_buyer × 100

Buyer bid rate:
bid_rate = Buyer_bids / Buyer_requests_received × 100

Buyer win rate:
win_rate = Wins / Buyer_bids × 100

SSP win rate:
ssp_win_rate = SSP_wins / SSP_bid_requests × 100

Render loss after win:
render_loss_pct = (SSP_wins - GAM_impressions) / SSP_wins × 100

CPM delta:
CPM_delta_pct = (CPM_A - CPM_B) / CPM_B × 100
```

## Reference heuristics

| Signal | Heuristic | Interpretation |
|---|---|---|
| GAM vs SSP impression discrepancy <15% | Often structurally explainable | Check definitions before escalating |
| GAM vs SSP impression discrepancy >25% | Investigate | Possible render/counting/scope issue |
| SSP vs Buyer request discrepancy >20% | Investigate | Possible filtering in transit, QPS, SPO, seat/deal routing |
| Buyer bid rate <1% | Very low bidding | Targeting, budget, CPM, creative, buyer filters |
| Buyer bid rate >50% but win low | Active buyer not winning | Floor, competition, priority, auction mechanics |
| Wins high but GAM impressions low | Post-win issue | Creative render, VAST, timeout, counting, player |

Do not treat thresholds as proof. They are triage accelerators.

---

# 7. Breakpoint Isolation Matrix

| Observed pattern | Likely breakpoint | First checks |
|---|---|---|
| GAM has avails but SSP sees few/no requests | Eligibility or SSP opportunity | Ad unit mapping, yield group, floors, wrapper, consent, supply path |
| SSP sees requests but buyer sees few/no requests | Buyer visibility | Seat, deal routing, QPS shaping, SPO, DSP filters, timeout |
| Buyer sees requests but bid rate is low | Buyer bidding | Budget, targeting, CPM target, creative approval, frequency cap, geo/device mismatch |
| Buyer bid rate high but win rate low | Auction/win | Floor, competition, priority, clearing price, bid below floor |
| SSP wins high but GAM impressions low | Render/serve | Creative, VAST/player, timeout, wrapper depth, counting mismatch |
| Impressions align but revenue differs | Reporting/commercial basis | Gross/net, currency, rev share, fees, timezone, spend vs revenue |
| Discrepancy segmented by geo | Eligibility/geotargeting | Geo mapping, buyer targeting, IP/device geo, regional consent |
| Discrepancy segmented by device/app | Device/app eligibility | app-ads.txt, bundle, IFA/RDID, ATT/LAT, SDK, platform support |
| Discrepancy segmented by time of day | Buyer pacing/budget | Daily budget, dayparting, frequency cap, QPS throttling |
| Discrepancy only on one deal ID | Deal setup/routing | Deal ID mapping, seat, buyer acceptance, floor, allowlist |
| Discrepancy across all demand | Systemic | Consent, GAM config, SSP outage, wrapper, reporting delay |

---

# 8. Format-Specific Modules

## Display Web

Check:

- hb_* targeting present in GAM
- Prebid auction completed before GAM request
- Ad unit mapping
- Lazy loading / refresh behaviour
- Consent gate
- UPR/floor conflicts
- ads.txt/sellers.json

## Web Video

Check:

- VAST URL response
- Wrapper depth
- VAST error codes
- Player timeout
- Autoplay / mute policy
- Ad rules / VMAP
- Creative approval
- Consent and content signals

## CTV / OTT

Check:

- App bundle/domain mapping
- app-ads.txt
- RDID/IFA presence and LAT/limited ad tracking
- Device/platform breakdown: Android TV, tvOS, Samsung, LG, HbbTV
- CSAI vs SSAI counting path
- VAST compatibility
- Content signals: cmsid, vid, duration, language, genre if relevant
- Buyer allowlist for app/bundle/device

## Mobile App

Check:

- App bundle
- SDK version
- IFA/IDFA/GAID availability
- ATT/LAT status
- app-ads.txt
- OM SDK / measurement if relevant
- Geo/device filters

## Deals / PMP / PG

Check:

- Deal active both sides
- Correct buyer seat
- Buyer acceptance / pending changes
- Deal ID mapping
- Floor / UPR conflict
- Domain/app/bundle allowlist
- Geo/device/format targeting
- Bid request visibility in DSP
- No-bid/filtering reasons

## Reporting-Only

Check:

- Data freshness
- Timezone
- Currency
- Gross/net
- Revenue share
- Billable vs served impressions
- Sampling
- Deduplication
- Different attribution dates

---

# 9. Mode B — Questionnaire Without Logs

Use when exports/logs are unavailable. Ask by funnel stage, not randomly.

## Block 1 — Data comparability

```markdown
- Are we comparing the exact same date range and timezone?
- Are the reports scoped to the same deal ID, ad units, domains/apps, geos, devices, and formats?
- Are revenue numbers gross or net?
- Are impressions counted on win, render, billable event, or buyer-served event?
- Is either report delayed, sampled, or filtered?
```

## Block 2 — Inventory and eligibility

```markdown
- What is the exact deal or campaign targeting: geo, device, format, domain/bundle, dayparting, audience, content?
- Does publisher inventory actually exist for that targeting in the checked period?
- Are floors/UPR higher than the buyer's effective bid or deal floor?
- Are there competitive exclusions, brand safety rules, frequency caps, or consent filters?
- Are ads.txt/app-ads.txt and sellers.json correct for the supply path?
```

## Block 3 — SSP opportunity and routing

Ask SSP:

```markdown
- How many eligible bid requests does the SSP see for this inventory/deal?
- How many requests are routed to the buyer seat/DSP?
- Is there QPS shaping, timeout filtering, traffic throttling, or SPO-related filtering?
- Is the deal ID mapped correctly to the buyer seat?
- Are domain/app/bundle/device/geo filters applied before sending to the buyer?
```

## Block 4 — Buyer visibility and bidding

Ask buyer/DSP:

```markdown
- How many bid requests are received for this deal/supply path?
- What is the bid rate?
- What are the main no-bid/filter reasons?
- Is campaign budget, pacing, targeting, frequency cap, or target CPM restricting bids?
- Is creative approved and eligible for this format/device?
- Is the supply path prioritised or deprioritised by SPO settings?
```

## Block 5 — Winning and serving

```markdown
- What is the win rate?
- Are bids losing due to floor, competition, or priority?
- Are SSP wins translating into GAM/player impressions?
- Are there creative render errors, VAST errors, timeouts, or wrapper-depth failures?
- Is the discrepancy only post-win?
```

Build the hypothesis ranking from these answers.

---

# 10. Evidence-Ranked Hypotheses

Produce hypotheses ordered by probability and evidence.

```markdown
| # | Hypothesis | Breakpoint | Evidence | Level | Verification test |
|---|---|---|---|---|---|
| 1 | [Most probable] | [Funnel stage] | [supporting data] | Verified / Probable / Possible / Ruled out | [test] |
| 2 | ... | ... | ... | ... | ... |
```

Evidence levels:

| Level | Meaning |
|---|---|
| Verified | Data confirms it directly |
| Probable | Strongly consistent; one additional test needed |
| Possible | Cannot be ruled out; more data needed |
| Ruled out | Data contradicts it |

---

# 11. Cheapest Verification Tests

For each Probable/Possible hypothesis, propose the cheapest useful test.

```markdown
## Verification Plan

### Test T1 — [name]
- Hypothesis tested:
- Action:
- Owner:
- Duration:
- Expected result if confirmed:
- Expected result if ruled out:
- Data needed:
```

Prefer tests that require minimal system changes. Do not recommend widening targeting, lowering floors, or changing production configuration before checking data comparability and supply chain.

---

# 12. Forensic Verdict

End with a concise verdict.

```markdown
## Forensic Verdict

Primary break point: [Inventory / Eligibility / SSP Opportunity / Buyer Visibility / Buyer Bidding / Auction-Win / Render-Serve / Reporting]

Confidence: High / Medium / Low

Most likely cause:
[Cause]

Evidence:
- [Evidence 1]
- [Evidence 2]

What is ruled out:
- [Ruled-out item]

What remains unconfirmed:
- [Pending item]

Next cheapest test:
[Test]

Owner:
[Publisher / SSP / Buyer / DSP / GAM / Engineering]

Recommended next action:
[Action]
```

If confidence is low, say so. Do not manufacture certainty.

---

# 13. Final Output

Produce `discrepancy_report.md` or inline report with:

1. Executive summary — 3 lines
2. Scope Lock
3. Data Quality Gate status
4. Normalisations applied
5. Funnel Map / breakpoint
6. Metrics calculated
7. Breakpoint Isolation Matrix result
8. Evidence-ranked hypotheses
9. Verification plan
10. Forensic Verdict
11. Next steps with owner
12. Partner/client questions if needed

Trigger:

- `client-deliverable` if a sendable client/buyer report is needed.
- `incident-response` if discrepancy indicates active production impact.
- `knowledge-capture` if a confirmed root cause creates a reusable anti-pattern.

---

# 14. Partner Communication Package

When data is missing, generate the relevant request.

## SSP request

```text
Subject: Data request for discrepancy analysis — [Client / Deal ID] — [Period]

Hi [Name],

We are investigating a discrepancy for [client/deal] during [period]. Could you please confirm the following for the same scope and timezone?

1. Eligible bid requests seen by the SSP
2. Requests routed to buyer seat [X]
3. Wins and gross revenue
4. Any QPS shaping, filtering, timeout, SPO, domain/app/bundle, geo, or device filters applied
5. If available, a breakdown by device, geo, domain/app, and hour

This will help isolate whether the break is before buyer visibility, at bidding, or after win/render.

Best,
[Signature]
```

## Buyer/DSP request

```text
Subject: Buyer-side visibility check — [Deal ID] — [Period]

Hi [Name],

We are comparing publisher/SSP reporting with buyer-side visibility for Deal ID [X] during [period]. Could you please confirm:

1. Bid requests received
2. Bid rate
3. Wins/spend
4. Main no-bid or filtering reasons
5. Campaign status, budget/pacing, targeting, creative approval, and floor/CPM constraints
6. Any domain/app/bundle, device, geo, or SPO filters affecting this supply path

We are trying to isolate whether the issue is request visibility, bidding, auction competitiveness, or post-win serving.

Best,
[Signature]
```

---

# Anti-Patterns

## AP-DF-001 — Comparing non-equivalent metrics

Bad:

```text
GAM impressions do not match buyer spend, so one system is wrong.
```

Good:

```text
First align impression definition, timezone, currency, gross/net basis, and reporting freshness before calling it an error.
```

---

## AP-DF-002 — Treating structural discrepancy as failure

Bad:

```text
There is an 8% difference between GAM impressions and SSP wins. This is anomalous.
```

Good:

```text
An 8% difference may be structurally explainable depending on counting methodology. Check definitions before escalating.
```

---

## AP-DF-003 — Widening targeting before checking supply chain

Bad:

```text
Buyer sees low avails, so let’s widen geo/device targeting.
```

Good:

```text
First check deal routing, seat, ads.txt/app-ads.txt, sellers.json, domain/app allowlist, and request visibility.
```

---

## AP-DF-004 — Accepting “buyer not bidding” without no-bid reason

Bad:

```text
The buyer is not interested in the inventory.
```

Good:

```text
Buyer bid rate is low. Request no-bid/filter reasons: budget, targeting, creative, floor, CPM target, SPO, frequency cap.
```

---

## AP-DF-005 — Ignoring segmentation

Bad:

```text
The discrepancy is 40%, so it is systemic.
```

Good:

```text
Segment by geo, device, format, domain/app, deal ID, and hour before deciding whether it is systemic.
```

---

## AP-DF-006 — Calling root cause too early

Bad:

```text
Root cause is floor mismatch.
```

Good:

```text
Floor mismatch is the leading hypothesis because buyer bid rate is high but win rate is low. Confirm by comparing bid CPMs against effective floors.
```

---

## AP-DF-007 — Confusing no avails with no bids

Bad:

```text
The buyer is not bidding, so the deal has no avails.
```

Good:

```text
Separate buyer visibility from bidding. First confirm whether requests reach the buyer; then analyse bid rate.
```

---

# Quality Rubric

| Score | Meaning |
|---:|---|
| 6 | Basic comparison; weak normalisation |
| 7 | Good calculations but limited breakpoint isolation |
| 8 | Solid cross-system discrepancy analysis |
| 9 | Funnel-based, evidence-ranked, with verification plan and partner questions |
| 9.5 | Forensic-grade: isolates breakpoint, avoids false certainty, and drives cheapest next test |

A 9+ analysis requires:

- Scope Lock completed
- Data Quality Gate checked
- Normalisations documented
- Funnel breakpoint identified or explicitly unknown
- Metrics calculated only on comparable data
- Hypotheses ranked by evidence
- Cheapest verification tests proposed
- Forensic Verdict included
- Unsupported claims avoided

---

# Optional API / Postman Integration

When available, use pre-configured API/Postman collections to automate data collection.

Examples:

- PubMatic: impression/win-rate report by deal ID
- Magnite: bidstream summary by deal ID / buyer seat
- Xandr/Microsoft Advertising: analytics report with deal ID filter
- GAM API: ad unit, line item, yield group, or report export

Treat API results as source data subject to the same Data Quality Gate.

---

# Changelog

## v1.1.1 — 2026-04-27

**Type:** Runtime lean revision

### Removed / reduced

- Removed `Design Decision` section.
- Reduced `Core Principle` to one operative rule.

### Kept

- Scope Lock
- Data Quality Gate
- Programmatic Funnel Map
- Breakpoint Isolation Matrix
- Format-specific modules
- Hybrid Mode
- Forensic Verdict
- Partner communication package
- Anti-patterns with bad/good examples
- Quality rubric

---

## v1.1.0 — 2026-04-27

**Type:** Major forensic upgrade

### Added

- Scope Lock
- Data Quality Gate
- Explicit Programmatic Funnel Map
- Breakpoint Isolation Matrix
- Format-specific modules: display, web video, CTV/OTT, mobile app, deals, reporting-only
- Hybrid Mode
- Forensic Verdict
- Partner communication package for SSP and buyer/DSP
- Anti-patterns with bad/good examples
- Quality rubric

### Changed

- Reframed from dataset comparison to funnel-breakpoint forensics
- Thresholds downgraded to reference heuristics, not universal rules
- Mode B questionnaire reorganised by funnel stage

### Integrations

- Use `gam-adops` for GAM/ad unit context
- Use `programmatic-deals` for PMP/PG/deal context
- Use `prebid-adtech` for wrapper/header bidding context
- Use `video-adtech` for VAST/CTV/player context
- Use `cmp-consent-flows` for privacy/consent issues
- Use `incident-response` if active production impact exists
- Use `client-deliverable` for sendable reports
