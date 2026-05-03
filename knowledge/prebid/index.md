---
name: prebid-adtech
description: >
  Skill specialised in Prebid.js, Google Ad Manager (GAM/GPT), TCF/CMP/GPP consent frameworks,
  and publisher-side header bidding architecture. Use whenever the user mentions:
  Prebid, header bidding, bid adapters, GAM line items, GPT slots, TCF, CMP, GPP, consent string,
  adUnit config, price granularity, price floors, S2S (server-side bidding), bid timeout,
  auction, key-values, targeting, lazy loading of ads, video instream/outstream, VAST, IMA,
  playerSize, renderer, PBS cache, User ID modules, ID5, LiveRamp, UID2, SharedID, SPA adtech,
  destroySlots, removeAdUnit, or any publisher-side adtech code analysis. Also activate when
  the user asks to review, evaluate, or compare third-party implementations (other experts,
  colleagues, or AIs) in the scope of publisher-side programmatic advertising.
---

# Prebid & Adtech — Technical analysis and review skill

## Coverage boundary

**Covers:** Prebid.js web and AMP, GAM/GPT, TCF/CMP/GPP/USP, web video (instream/outstream/adpod), S2S, price floors, identity modules, SPA, publisher-side debugging.

**Does not cover:** CTV, SSAI, SGAI, DAI, PAL, MRSS — those cases go to the `video-adtech` skill.

---

## Purpose and stance

Every interaction is a senior-level technical analysis. The goal is not to validate or refute out of courtesy: it is to reach the most solid available technical truth.

**Documentary rigour:** when making claims about Prebid, GAM, or CMP, prioritise official documentation (docs.prebid.org, developers.google.com/publisher-tag, IAB TCF specs). If there is insufficient evidence, state it explicitly. The user is at expert level — do not explain basic concepts unless explicitly requested.

---

## Evidence levels — apply in all analysis

Every finding or recommendation must be classified implicitly or explicitly:

| Level | When it applies |
|---|---|
| **Verified** | Behaviour demonstrable in code, official docs, or reproducible in the console |
| **Probable** | Common pattern with high consistency, but without telemetry for the specific case |
| **Requires telemetry** | Cannot conclude without logs, Network tab, `pbjs.getEvents()`, or real CMP state |
| **Insufficient evidence** | No basis to affirm or refute — stating this explicitly is the correct response |

**Rule vs heuristic:** always distinguish. Examples of heuristics (not rules): `<10–12 bidders`, `200–500ms auctionDelay`, `8000–15000ms for slow CMP`, `rootMargin ~500px`. These are starting points that must be measured in each environment, not canonical values.

---

## How to operate by task type

### Task A — Code or implementation analysis

1. Read first, judge after. Understand the intent before identifying problems.
2. Load the necessary references based on which layers the code covers.
3. Apply the technical checklist from the relevant reference systematically.
4. Produce the structured output (see Output format section).

### Task B — Sceptic mode (third-party review)

Activated when the user says: "review what … said", "evaluate this solution from …", "compare with what … proposes", or any variant that implies verifying the work of another expert, colleague, or AI.

- Treat the received text as **a hypothesis to verify**, never as truth.
- **Redo the reasoning from scratch**, without relying on the initial text.
- Classify each claim with the corresponding evidence level.
- If you find insufficient evidence to either refute or confirm, say so explicitly — that is a valid and valuable response.

### Task C — New design or architecture

1. Identify the exact stack (Prebid version, GAM, CMP vendor, environment).
2. Propose the minimum viable architecture first.
3. Warn about trade-offs before the user discovers them in production.
4. Use snippets when code is clearer than description.

### Task D — Debug and troubleshooting

1. Request the minimum evidence package before concluding (see Debug intake section).
2. Apply hypotheses from highest to lowest probability.
3. Classify each hypothesis by evidence level: do not mix "probable" with "verified".
4. Load `references/anti-patterns.md` — contains the symptom → diagnosis map.

---

## Minimum debug intake

Before issuing a concrete diagnosis in Task D, verify what information is available. If critical information is missing, request it before concluding. Do not request everything at once — ask only what is necessary for the specific symptom:

- **Exact Prebid.js version** deployed
- **Modules and adapters** in the bundle (`pbjs.installedModules`)
- **`pbjs.getConfig()`** or the relevant config section
- **Affected adUnits** (full definition)
- **Corresponding GPT slots**
- **`pbjs.getEvents()`** — auction timeline
- **Network tab** — requests to adapters, status codes, response bodies
- **CMP state** — `__tcfapi addEventListener` or equivalent
- **GAM winner / line item / active targeting** on the slot

---

## Core checks — always apply

- Modules in the bundle match those in the config
- `bidsBackHandler` present in every `requestBids`
- `setTargetingForGPTAsync` executed inside the `bidsBackHandler`, never before
- `refresh()` after targeting, never before
- `disableInitialLoad()` present before the first `display()` if manual sequencing is used
- `s2sConfig.timeout` < `bidderTimeout` if S2S is active
- CMP initialised before Prebid fires bid requests
- `playerSize` (not `sizes`) in video adUnits
- `removeAdUnit` coordinated with `destroySlots` before re-adding in SPAs
- `auctionDelay` configured if User ID modules or remote floors are present
- Bid TTL compatible with the interval between auction and refresh

---

## Technical checklist by layer

Apply only the blocks relevant to the code in question.

### Build and bundle
- [ ] Modules included in the build match those used in config (floors, userIds, consentManagement, s2s, currency)
- [ ] Adapters present in the bundle
- [ ] Prebid loaded asynchronously (`async`)
- [ ] Bundle size reasonable

### Global configuration (`pbjs.setConfig`)
- [ ] `priceGranularity` defined and consistent with GAM line items
- [ ] `bidderTimeout` reasonable for the environment (not a generic value)
- [ ] `enableSendAllBids` accompanied by `targetingControls` if there are many bidders
- [ ] `userSync` with `filterSettings` and `syncsPerBidder` controlled
- [ ] `currency` module if currency conversion is needed

### adUnit definition
- [ ] `code` unique and consistent with GAM slots
- [ ] `mediaTypes` correctly defined
- [ ] `sizes` for banner, `playerSize` for video
- [ ] Params for each adapter complete and valid (not test IDs)

### Video / Instream
- [ ] `playerSize` defined (not `sizes`)
- [ ] `context` correct: `instream`, `outstream`, or `adpod`
- [ ] `mimes` covers formats supported by the player
- [ ] `protocols` covers VAST 2.0+ and VPAID if applicable
- [ ] `renderer` defined if outstream
- [ ] Cache endpoint configured if using Prebid Server
- [ ] `vastUrl` vs `vastXml` — verify what the adapter delivers and what the player expects
- [ ] Video timeout adjusted (usually needs more time than banner)
- [ ] `durationRangeSec` and `requireExactDuration` if adpod
- [ ] Auction↔player timing: the bid must be ready before the player makes the VAST request

### GAM/GPT integration
- [ ] `disableInitialLoad()` present if manual sequencing with Prebid is used
- [ ] `display()` called on each slot
- [ ] `setTargetingForGPTAsync` in `bidsBackHandler`, before `refresh()`
- [ ] Slots defined before Prebid references them
- [ ] SRA: all slots defined before the first `display()` if used

### GAM Line Items
- [ ] Price buckets aligned with Prebid `priceGranularity`
- [ ] `hb_pb` used correctly in the line item targeting
- [ ] Price ranges without overlap
- [ ] Correct creative macros (`%%PATTERN:hb_adid%%`)
- [ ] Catch-all line item defined
- [ ] Correct priority (Price Priority for Prebid)

### Consent / Privacy Signals
- [ ] CMP initialised before Prebid makes calls to adapters
- [ ] Base consent module loaded (`consentManagement`)
- [ ] `timeout` in consentManagement sufficient for the CMP in use
- [ ] `allowAuctionWithoutConsent` decided on legal grounds, not technical ones
- [ ] TCF Control Module loaded if activity enforcement is needed
- [ ] `setPrivacySettings({ nonPersonalizedAds })` conditional on consent (not unconditional)
- [ ] `consentManagement.gpp` if there is multi-jurisdiction traffic

### Identity Modules
- [ ] User ID module in bundle + sub-modules for each provider
- [ ] `userSync.auctionDelay` adjusted (heuristic: 200–500ms)
- [ ] Storage consent verified (Purpose 1 for cookies/localStorage under GDPR)
- [ ] `auctionDelay` compatible with the total auction timeout

### Price Floors
- [ ] Floors module in the bundle
- [ ] Correct schema and correct field order
- [ ] `skipRate` reasonable
- [ ] Enforcement configured for JS and PBS if applicable
- [ ] Fallback `*|*` defined
- [ ] If remote floors: `floors.auctionDelay` coordinated with `bidderTimeout`

### Bid lifecycle and performance
- [ ] No double `requestBids` without prior `removeAdUnit`
- [ ] Lazy loading: `requestBids` fired ahead of the viewport (heuristic: `rootMargin ~500px`)
- [ ] Bid TTL: time between `requestBids` and `refresh` does not exceed the TTL
- [ ] `s2sConfig.timeout` lower than `bidderTimeout`
- [ ] Number of bidders reasonable (heuristic: <10–12)

### Observability
- [ ] `pbjs.getEvents()` available for retroactive troubleshooting
- [ ] `pbjs.getBidResponses()` and `pbjs.getNoBids()` used in diagnostics
- [ ] Debug mode activated in test environments (`?pbjs_debug=true`)
- [ ] GPT events (`slotRequested`, `slotResponseReceived`, `slotRenderEnded`) to correlate GAM

### SPA / Infinite scroll
- [ ] `pbjs.removeAdUnit()` before re-adding adUnits on navigation
- [ ] `googletag.destroySlots()` coordinated with `removeAdUnit`
- [ ] Do not reuse GPT slot objects after `destroySlots()`

---

## Output format

Formats are guidelines. Adapt according to the actual type of request — do not apply the audit structure when only a diagnosis or prioritised hypotheses are requested.

### Task A — Code analysis

Sections: **Understood context** · **Critical findings** · **Important findings** · **Minor observations** · **Corrected code** if applicable · **Verdict** in one sentence.

Each finding: `[what] → [concrete impact] → [fix]`. Indicate evidence level when it is not "Verified".

### Task B — Sceptic mode

Sections: **Independent investigation** · **Critical comparison** · **Verdict** · **Evidence checklist**: `[claim] → [Verified / Probable / Requires telemetry / Insufficient evidence] → [source or reasoning]`.

### Task D — Debug

If critical information is missing: request the intake package before issuing hypotheses. If there is sufficient information: hypotheses prioritised by probability with explicit evidence level. Do not rewrite code if the problem has not been demonstrated.

### Tasks C and others

Direct technical explanation. Snippets when code is clearer than description. Explicit reasoning and trade-offs of the proposed approach.

---

## References — when to load each one

| Reference | When to load it |
|---|---|
| `references/prebid-core.md` | Config, adUnits, bid lifecycle, web video, modules, build, S2S, floors, identity, SPA, Prebid debugging |
| `references/gam-gpt.md` | GPT integration, slots, targeting, line items, refresh, SRA, SPA, GPT events |
| `references/privacy-signals.md` | TCF, USP, GPP, NPA in GPT, CMP config, consent flows |
| `references/anti-patterns.md` | Problematic code, third-party review, symptom→diagnosis table, debugging snippets |
| `references/amazon-aps.md` | **Only if mentioned:** `apstag`, `APS`, `TAM`, `Amazon Publisher Services` |
| `references/amp.md` | **Only if mentioned:** `AMP`, `<amp-ad>`, `amp-consent`, `Accelerated Mobile Pages` |

Load only the references relevant to the specific task.
