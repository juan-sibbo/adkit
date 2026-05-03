---
name: programmatic-deals
description: >
  Publisher-side deal management and troubleshooting with focus on GAM/GAM360, SSPs, and
  buyers. Specialised in cases where the buyer is not spending, not listening to enough
  inventory, not receiving enough avails, or the deal is not scaling despite appearing
  correctly configured. Covers: deal health funnel (avails → listening → bidding → delivery),
  diagnosis of mismatch between actual supply and commercial expectations, buyer activation
  (seat, DSP, supply path), deal-side configuration (floor, targeting, brand safety,
  creative approval), and supply chain (sellers.json, ads.txt) as a prerequisite. Activate
  for: buyer not spending, active deal with no impressions, low bid rate, "not receiving
  requests", fill rate well below target, discrepancy between what publisher/SSP/buyer see.
  Does not cover: deal setup in GAM (yield groups, line items, pricing rules) → gam-adops;
  Prebid → prebid-adtech.
---

# Programmatic Deals — Publisher-side, GAM/GAM360

## Coverage boundary

**Core:** troubleshooting publisher-side deals where the buyer is not spending, not scaling, or not listening to enough inventory. GAM/GAM360 as the axis of final delivery. SSPs (AdX/OB, Magnite, Triplelift, Xandr, IX, PubMatic, Teads, Adagio, Richaudience, Criteo) as diagnostic support. OpenRTB, sellers.json, ads.txt as prerequisites or lateral checks — necessary but not the primary narrative axis.

**Does not cover:** PG/PD line item configuration, yield groups, pricing rules, Open Bidding eligibility in GAM → `gam-adops`. Prebid↔GAM integration → `prebid-adtech`. VAST/CTV → `video-adtech`.

**Intentional overlap:** when the problem crosses SSP and GAM (e.g., the SSP sees wins but GAM does not record impressions), diagnose from this skill up to the delivery point in GAM, then hand off to `gam-adops` for the ad server side.

**Indicator:** `[360]` for features exclusive to GAM 360.

---

## Stance and analysis level

Senior-level publisher-side technical analysis. The goal is to reach the root cause, not to validate that "the deal looks correct". Most deals that fail to scale have their root cause in one of three categories: (1) actual eligible inventory is much smaller than assumed, (2) the buyer is not genuinely listening to that inventory, or (3) the problem is commercial, not technical. Separate these three before diagnosing in depth.

### Evidence levels

| Level | When it applies |
|---|---|
| **Verified** | Behaviour documented in IAB specs, SSP documentation, or reproducible on the platform |
| **Probable** | Pattern consistent with system logic, without direct confirmation for the specific case |
| **Requires data** | Cannot conclude without SSP metrics, bid request logs, or buyer confirmation |
| **Insufficient evidence** | No basis to affirm or refute — this is a valid response |

---

## Deal health funnel

The backbone of the diagnosis. Identify at which step the funnel breaks before diagnosing in depth.

```
1.  Deal created          → Does the deal exist in the SSP and is it active?
2.  Deal active           → Does it have valid dates, floor, and targeting configured?
3.  Eligible inventory    → Are there real avails that meet all deal filters?
4.  Requests to buyer     → Is the SSP sending bid requests with the deal ID?
5.  Buyer listening       → Does the buyer have the supply path / seat / exchange active?
6.  Buyer bidding         → Is the buyer bidding? At what CPM?
7.  Buyer winning         → Are bids clearing the floor and winning the auction?
8.  Creative serving      → Is the creative approved and compatible with the format?
9.  GAM counting          → Is GAM recording the impression? Is the line item capturing it?
10. Spend scaling         → Does the buyer have budget and pacing to scale?
```

**Diagnosis by breakpoint:**

| Fails at | Category | Skills involved |
|---|---|---|
| 1–2 | Deal setup at SSP | This skill |
| 3 | Actual supply / targeting / forecasting | This skill — avails check |
| 4–5 | Buyer activation / supply path / seat / DSP | This skill |
| 6 | Price / floor / buyer strategy / budget | This skill |
| 7–9 | Delivery / creative / GAM / policy | This skill + `gam-adops` |
| 10 | Commercial or optimisation issue — not necessarily technical | This skill — separate from technical |

---

## Operating modes

### Mode A — Buyer not spending / low spend / not scaling *(primary mode)*

Activate when the deal appears correctly configured but the buyer is not spending or fill rate is well below expectations. Walk through the funnel block by block, emitting hypotheses with evidence level as soon as there is sufficient data.

**Block 1 — Actual supply: are there enough eligible avails?**

The most frequent cause of low spend. The forecast was run against broad inventory; the deal was constrained to a narrow combination of filters.

- How many eligible avails does the SSP or GAM estimate for exactly the deal filters (format + geo + device + ad unit + domain + size + floor)?
- Does the commercially promised inventory match the technically targeted inventory?
- If multiple SSPs are active: is the deal in the SSP where that inventory actually lives?

Diagnostic action: request an avail estimate from the SSP with the exact deal parameters. If there is a significant gap between the estimated volume and actual eligible volume → the problem is supply-side, not demand-side.

**Block 2 — Buyer activation: is the buyer actually listening to that inventory?**

The SSP confirming the deal is correctly configured on their side does not guarantee the buyer is targeting that SSP, that seat, and that inventory.

- Does the buyer have the deal in "Active" status in their DSP — not just "saved" or "pending"?
- Are they targeting the correct SSP/exchange with the correct seat ID?
- TTD: correct Merchant ID for the publisher? DV360/AdX: deal in an active line item with budget?
- Is the buyer reporting "not seeing enough requests" or "no bid requests received"?
- Is the SSP seeing requests sent but no bids from that buyer?

Diagnostic action: ask the buyer for (1) deal active in their platform, (2) seat and exchange configured, (3) bid rate in the last 24–48h from their UI.

**Block 3 — Deal restrictions: is the configuration too restrictive?**

Ordered by frequency:
- Floor CPM: is it achievable by this buyer on this inventory? Verify against market CPMs or historical bids in the SSP.
- Targeting: geo, device, format, sizes, dayparting — any unintended restriction?
- Domain / app bundle: does the list actually include where the inventory lives?
- Buyer's brand safety / ad quality: are publisher categories blocked in the DSP?
- `wseat`: if populated, does it include the correct seat?
- Creative: approved in the SSP? Compatible with the format (sizes, MIME types)?

**Block 4 — GAM pass-through: is GAM letting the deal through?**

If the SSP reports wins but GAM does not record impressions, or fill is anomalous:
- Yield group active with the correct SSP/buyer `[360]`
- Line item active, deal ID correctly mapped, appropriate priority
- Competition from higher-priority demand sources consuming the same inventory
- Gap between "bid won" at the SSP and "impression served" in GAM `[360]` → Data Transfer logs in BigQuery as the definitive source
- `gam-adops` Delivery Tools → line item Troubleshooter: "Losing to higher-priority line item" as a contention signal

**Silent blockers — verify before handing off to `gam-adops`:**

If the SSP reports wins but GAM shows no activity, two frequent causes that do not require deep GAM diagnosis to identify:

1. **Privacy filter / TCF:** is the buyer discarding bid requests due to missing a valid TC string or absent consent for Purpose 1? Symptom: the SSP sees bid requests sent to the buyer but the buyer is not bidding — and the problem is transversal across all the publisher's deals, not just this one. If so, this is not a deal problem: it is consent infrastructure. → `cmp-consent-flows`

2. **UPR conflict:** is there a Unified Pricing Rule in GAM with a floor above the deal floor? The SSP considers the bid a win; GAM discards it via the UPR before serving the impression. Particularly relevant for PD — PG generally bypasses UPRs, PD does not always. Symptom: wins at the SSP, zero impressions in GAM, no visible error. → `gam-adops` for pricing rule review

→ Deep GAM diagnosis: `gam-adops`

**Block 5 — Is this technical or commercial?**

A deal can be technically correct and have low spend because the buyer has no real intention to scale, is in test phase, or accepted the deal without a volume commitment.

| Signal | Interpretation |
|---|---|
| SSP shows occasional bids, buyer listening but little | Commercial: budget, pacing, KPI, internal DSP optimisation |
| Zero bids despite active deal | Investigate technical first (blocks 1–3) |
| Buyer bidding but CPM consistently low | Floor strategy or pricing — not technical |
| Buyer scaling on other publishers but not this one | Brand safety or publisher performance |
| Active deal, correct setup, buyer "looking at it" for weeks | Commercial commitment, not a technical problem |
| Buyer asks to lower the floor without giving data on why they are not buying | Signal of a commercial problem — do not treat as technical without evidence |
| Buyer says "we're optimising" or "monitoring performance" | DSP optimising away — ask for concrete metrics on which KPI is failing |
| Spend starts then stalls with no technical changes | Budget exhausted, frequency cap reached, or KPI not met in the DSP |
| Spend only picks up under commercial pressure | The problem was commitment, not technical — close the technical diagnosis |

---

### Mode B — Deal configuration review

1. Walk through the funnel step by step with the available data.
2. Identify risks of insufficient supply, excessive restrictions, and incorrect buyer activation.
3. Load `references/anti-patterns.md`.
4. Verify prerequisites: `references/supply-chain.md`.

### Mode C — Sceptic (third-party review)

Activate when: "the SSP says everything is correct", "the buyer says they are active", "review what they sent us". Treat all claims as hypotheses. Identify at which funnel step the actual evidence sits vs. the claim.

### Mode D — Deal strategy architecture

1. Identify actual eligible volume before proposing anything.
2. Propose a deal type mix (PG vs PD vs PMP) with trade-offs between guarantee and flexibility.
3. Warn about mismatch between actual supply and commercial expectations before the deal is signed.
4. Floor strategy: balance between revenue protection and fill rate; always clarify whether the floor is net or gross.

**Pre-signature checklist — verify before committing to volume:**
- Avail estimate requested from the SSP with the exact filters the deal will have (not a broad estimate)
- Floor agreed clearly as net vs. gross, and verified to be achievable by the target buyer
- Correct buyer seat ID confirmed with the SSP — do not assume the buyer will know how to configure it
- Deal domain list / app bundle reviewed to include the publisher's actual relevant inventory
- Buyer's brand safety restrictions and category blocks confirmed against the publisher's content
- Buyer's creatives compatible with the deal format (sizes, MIME types) and with no pending review restrictions
- Deal timezone agreed explicitly if buyer and publisher operate in different zones
- Buyer's volume and budget commitment confirmed explicitly — "we're interested" is not a commitment

---

## Minimum intake — focused on avails + listening + bidding

Do not request everything at once. Emit hypotheses with the available data and ask only for what would resolve the ambiguity of the specific symptom.

**Deal data:**
- Exact SSP/exchange and deal type (PG / PD / PMP / Auction Package / Preferred)
- Deal ID, status in the SSP, active dates
- Floor CPM and configured targeting (geo, device, format, sizes, ad unit, domain list)

**Supply data:**
- Avails estimated by SSP or GAM for the exact deal parameters
- Commercially promised volume vs. technically eligible volume

**Buyer data:**
- Buyer + DSP + seat ID
- Confirmation of active listening (deal active, exchange/seat configured)
- Whether they report "not seeing requests" or low bid request volume

**Performance data:**
- Bid rate · win rate · impression rate (the three funnel conversion points)
- Observed bid CPMs vs. floor
- Whether GAM shows eligible requests for the deal

---

## Core checks — always apply

- Actual eligible inventory verified with the exact deal filters — not broad estimates
- Buyer with deal in "Active" status in their DSP — not just confirmed by the SSP
- Buyer's seat ID in the SSP matches the seat from which the buyer is targeting; for TTD verify Merchant ID specifically
- Floor CPM consistent with real CPMs for that inventory from that buyer; confirm with the SSP whether the floor is net or gross before communicating it to the buyer
- Targeting without unintended restrictions; domain/app bundle list verified against the publisher's actual inventory
- Buyer's creative approved in the SSP before concluding the problem is demand-side
- GAM yield group and line item verified if the SSP reports wins but GAM does not record impressions
- Deal dates and timezone consistent between SSP and buyer — a deal configured in UTC may not match the buyer's expectations in CET/other zones
- Separate technical problem from commercial problem before escalating
- ads.txt correct for the SSP involved (prerequisite, not the primary diagnosis)

---

## References — when to load each one

| Reference | When to load it |
|---|---|
| `references/anti-patterns.md` | Modes A and B — always. Symptom → diagnosis table focused on buyer not spending |
| `references/openrtb-deal-object.md` | When the diagnosis reaches bid request level: fields, per-SSP extensions, `at`, `wseat`, `dealid` |
| `references/supply-chain.md` | When sellers.json, ads.txt, or schain are suspected of blocking the deal |

---

## Output format

### Mode A — Buyer not spending
Locate the funnel breakpoint first. Hypotheses labelled with evidence level. Format: `[funnel step] → [probable cause] → [data that would confirm it]`. Separate what the publisher can verify vs. what requires data from the SSP or the buyer. Always conclude with: is this technical or commercial?

### Mode B — Configuration review
Sections: **Understood context** · **Supply risks** · **Buyer activation risks** · **Configuration risks** · **Pending prerequisites** · **Verdict** in one sentence.

### Mode C — Sceptic
Sections: **Independent analysis** · **Comparison with claims** · **Verdict** · **Evidence checklist**: `[claim] → [funnel step] → [evidence level] → [how to verify]`.

### Mode D — Architecture
Structured proposal with: assumed stack · actual eligible volume · deal type mix · floor strategy · commercial mismatch risks · per-SSP differences if applicable.
