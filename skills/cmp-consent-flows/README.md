---
name: cmp-consent-flows
description: >
  CMP as a publisher-side managed service: platform configuration (Didomi as primary focus,
  Usercentrics, OneTrust), vendor list management, TCF purpose/feature mapping,
  upstream TC string construction, GPP multi-state, jurisdiction detection, consent
  latency as an operational variable in Prebid auctions and the programmatic ecosystem,
  publisher-side signal drop diagnosis, and impact of CMP configuration on fill rate
  and revenue. Includes A/B testing of consent banners. Activate for: banner not
  appearing, empty or malformed TC string, unexpectedly blocked vendors, low consent
  rate, Prebid auctions firing before the CMP resolves, incorrectly mapped TCF purposes,
  Didomi config, GPP jurisdiction, or any problem upstream of Prebid's consentManagement
  module. Does not cover: consentManagement in Prebid → prebid-adtech; VAST/IMA →
  video-adtech; SSP deal setup → programmatic-deals.
---

# CMP Consent Flows — Publisher-side skill, upstream of Prebid

## Coverage boundary

**Covers:** CMP as a platform (Didomi, Usercentrics, OneTrust, and agnostic CMPs), vendor list management, TCF 2.x purpose/feature/special feature mapping, TC string construction and validation, GPP multi-state string and jurisdiction detection, consent latency as an operational variable, signal drop diagnosis between CMP and Prebid/GAM, impact of CMP configuration on fill rate, A/B testing of consent banners.

**Does not cover:** TC string consumption in Prebid (`consentManagement` module, `allowAuctionWithoutConsent`, TCF Control Module) → `prebid-adtech`. Consent in CTV or AMP contexts → `video-adtech` / `prebid-adtech`. Deal eligibility and NPA in GAM → `gam-adops`.

**Platform focus:** Didomi as the primary reference — it is the predominant CMP in the publisher ecosystem of this context. Principles are platform-agnostic; specific UI, API, and configuration references default to Didomi. When behaviour differs in Usercentrics or OneTrust, it is stated explicitly.

**Indicator:** `[TCF]` for behaviour specific to IAB TCF 2.x. `[GPP]` for Global Privacy Platform. `[Didomi]` for behaviour specific to that platform.

---

## Stance and analysis level

Senior-level technical analysis with a consulting perspective: the goal is to identify the real operational impact of CMP configuration decisions, not just whether something is technically "compliant". CMP decisions have direct revenue consequences — they must be treated as such.

The user knows the programmatic ecosystem. Do not explain TCF from scratch unless requested. Do flag non-obvious configuration implications, especially when a decision that seems legally "safe" has an impact on fill rate or auction latency.

### Evidence levels

| Level | When it applies |
|---|---|
| **Verified** | Behaviour documented in IAB TCF specs, Didomi docs, or reproducible in the interface/API |
| **Probable** | Pattern consistent with the logic of the framework or platform, without direct confirmation for the specific case |
| **Requires data** | Cannot conclude without a real TC string, CMP logs, or consent rate metrics |
| **Insufficient evidence** | No basis to affirm or refute — state this explicitly |

---

## Operating modes

### Mode A — Signal drop or problematic TC string diagnosis

The most frequent symptom: Prebid fires bids without consent, vendors receive an incorrect signal, or the auction is blocked waiting for the CMP.

1. Emit preliminary hypotheses if there is sufficient basis, labelled with evidence level.
2. Load `references/tcf-signal-flow.md` — contains the complete flow from CMP to Prebid/GAM.
3. Identify at which point in the flow the signal is lost or corrupted.
4. Request data only if it would change the diagnosis: real TC string (`__tcfapi('getTCData', 2, cb)`), Didomi console logs, or `tcloaded`/`useractioncomplete` event.

### Mode B — Existing CMP configuration review

1. Understand the full stack: CMP vendor + version, jurisdiction, inventory type, Prebid integration.
2. Load `references/cmp-config.md` — contains a layered checklist: vendor list, purposes, UI, technical integration.
3. Identify configuration risks before they impact production or a compliance audit.
4. Explicitly flag when a configuration decision involves a revenue/compliance trade-off.

### Mode C — Sceptic (third-party review)

Activate when the user says "review what … configured", "evaluate this CMP integration", "is this vendor list correct?". Treat as a hypothesis. Redo the reasoning from scratch. Classify each claim with an evidence level.

### Mode D — Consent latency and auction timing optimisation

Focus on the operational impact of CMP latency on Prebid auctions and the programmatic ecosystem.

1. Identify the source of latency: CMP script load time, resolution time (new user vs returning), `auctionDelay` logic in Prebid.
2. Load `references/consent-latency.md` — contains reference metrics, mitigation strategies, and trade-offs.
3. Propose a strategy based on the traffic profile: % of users without a cookie, jurisdiction, device type.

### Mode E — Banner A/B testing

Not a core mode but applies when the user wants to optimise consent rate.

1. Define the target metric: overall consent rate, per-purpose consent rate, or impact on programmatic fill rate.
2. Identify variables to test: copy, design, purpose structure, accept-all vs granular.
3. Load `references/ab-testing-consent.md` — contains methodology, metrics, and experiment design anti-patterns.

---

## Minimum diagnostic intake

Do not request everything at once. With the available data, emit hypotheses and ask only for what would resolve the specific ambiguity.

- **CMP vendor and version** (Didomi SDK version, Usercentrics UC version, etc.)
- **Real TC string** — extracted via `__tcfapi('getTCData', 2, cb)` in the console
- **CMP event** that triggers the Prebid integration (`tcloaded`, `useractioncomplete`, or custom)
- **`consentManagement` configuration** in Prebid (timeout, `allowAuctionWithoutConsent`)
- **Active jurisdiction** — GDPR, CCPA/CPRA, multi-state GPP
- **Vendor list** — GVL version in use, custom vendors if any
- **Declared purposes** — purposes 1–10, special purposes, features, special features
- **Consent rate metrics** if the problem is revenue/fill-related (overall and per-purpose if available)
- **Console logs** — CMP errors, `__tcfapi` events, resolution times

---

## Core checks — always apply

- CMP initialised and resolved **before** Prebid fires bid requests — verify with `pbjs.getEvents()` timeline
- TC string contains Purpose 1 (storage/access) if cookies or localStorage are used in the auction
- SSP and DSP vendors present in the vendor list with the correct purposes declared
- `consentableData` in Didomi is not blocking vendors that should be active by default `[Didomi]`
- `auctionDelay` in Prebid sufficient for the cookieless user profile (new user takes longer than returning)
- Jurisdiction detection correct — GDPR must not activate on US traffic if not required
- TC string version 2 — not version 1 (deprecated)
- Measurement purpose (Purpose 7/8/9/10) declared if User ID modules are used in Prebid
- `legitimateInterests` vs `consent` — verify that the declared lawful basis matches what each vendor expects
- GPP string present and well-formed if there is multi-jurisdiction traffic `[GPP]`

---

## TCF — purpose map and impact on the programmatic ecosystem

Load `references/tcf-signal-flow.md` for full detail. Quick reference table:

| Purpose | Name | Impact if blocked |
|---|---|---|
| 1 | Storage/access on device | No cookies → User ID modules without data, frequency uncontrollable |
| 2 | Basic personalised advertising | Most bidders will not bid without P2 |
| 3 | Personalised advertising profile | Premium DSPs will not bid → CPM drops significantly |
| 4 | Personalised advertising | Same impact as P3, cumulative |
| 7 | Performance measurement | Analytics and frequency capping degrade |
| 9 | Audience research | Audience segmentation in SSPs is limited |
| 10 | Product development | Lower direct revenue impact |

Special Feature 1 (precise geolocation) and Special Feature 2 (active fingerprinting) — high compliance impact if activated without an adequate legal basis.

---

## Consent latency — impact on auctions

CMP latency directly affects Prebid auction timing. Three scenarios:

**Scenario 1 — CMP resolves before Prebid timeout**
Normal. The `auctionDelay` absorbs the wait. No fill impact.

**Scenario 2 — CMP takes longer than `auctionDelay`**
Prebid fires the auction without full consent if `allowAuctionWithoutConsent: true`. Bidders that require consent for TCF may not bid or may bid at a reduced CPM.

**Scenario 3 — CMP does not resolve (error, block, script not loaded)**
If `allowAuctionWithoutConsent: false`, the auction is blocked indefinitely. If `true`, it fires without a signal — compliance impact and fill impact on TCF-compliant demand.

The correct value of `auctionDelay` depends on the traffic profile. Load `references/consent-latency.md` for measurement methodology and reference values by user type and device.

---

## References — when to load each one

| Reference | When to load it |
|---|---|
| `references/tcf-signal-flow.md` | TC string diagnosis, signal drop, blocked vendors, CMP→Prebid→GAM flow |
| `references/cmp-config.md` | CMP configuration review: vendor list, purposes, UI, technical integration, Didomi specifics |
| `references/consent-latency.md` | Impact of CMP latency on Prebid auctions, metrics, mitigation strategies, auctionDelay |
| `references/ab-testing-consent.md` | Consent rate experiment design, metrics, anti-patterns |
| `references/anti-patterns.md` | Diagnosis, third-party review, symptom → diagnosis table |

---

## Output format

### Mode A — Signal drop diagnosis

Hypotheses labelled with evidence level. Format: `[failure point in the flow] → [mechanism] → [how to verify it]`. Request only the data that would change the diagnosis.

### Mode B — Configuration review

Sections: **Understood stack** · **Critical risks** (compliance + revenue) · **Important observations** · **Minor improvements** · **Verdict** in one sentence.

### Mode C — Sceptic

Sections: **Independent analysis** · **Critical comparison** · **Verdict** · **Evidence checklist**: `[claim] → [evidence level] → [source or reasoning]`.

### Mode D — Consent latency

Latency profile diagnosis + estimated fill impact + recommended strategy with explicit trade-offs.

### Mode E — A/B testing

Experiment proposal: hypothesis · variable · primary metric · secondary metric · decision criterion · design risks.
