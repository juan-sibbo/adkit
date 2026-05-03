# Entry Format — Canonical entry structure

Load before drafting any entry. Contains the canonical structure, sections of each target file, and examples by finding type.

---

## Entry types

Not every finding is an anti-pattern. Using the correct type keeps the target file sharp.

| Type | When to use |
|---|---|
| `anti-pattern` | Configuration or decision that seems correct but causes problems — the risk is not obvious |
| `verified-fix` | Resolution confirmed in production for a specific problem with identified root cause |
| `behavioral-note` | Documented behavior of a platform/tool that is not in the official docs or is counterintuitive |
| `debug-note` | Diagnostic pattern: symptoms → tool → what to look for; useful to accelerate future troubleshooting |
| `case-note` | Real finding but too local to generalize; does not go in main sections — separate section at the end of the file if captured |

---

## Canonical entry structure

```markdown
### [Descriptive title of the symptom — in the user's terms]

**Type:** [anti-pattern / verified-fix / behavioral-note / debug-note / case-note]

**Symptom:** [Unexpected behavior as experienced by the user. 1-2 sentences.]

**Context:** [Minimum stack needed to reproduce: platform, version, environment, inventory type. Omit anything that does not affect reproducibility.]

**Root cause:** [Technical mechanism that explains the symptom. Not just "what" but "why". If there is ambiguity, label it.]

**Resolution:** [What change resolves the problem. Actionable and ordered if there are steps.]

**Verification:** [How to confirm it is resolved — tool, log, expected behavior.]

**Evidence level:** [Verified / Probable / Requires data] — [one line explaining why that level]

**Metadata:** Captured: [YYYY-MM] · Versions: [if applicable] · Status: [Active / Pending revalidation / Historical] *(optional field — omit if no relevant version information)*

**Tags:** [keywords for search: symptom, platform, GAM feature, etc.]
```

---

## Mandatory vs optional fields

| Field | Mandatory | May be omitted if... |
|---|---|---|
| Symptom | Yes | — |
| Context | Yes | Only if 100% platform/version agnostic |
| Root cause | Yes | — |
| Resolution | Yes | — |
| Verification | Recommended | No obvious verification tool exists |
| Evidence level | Yes | — |
| Tags | Recommended | The symptom already acts as a sufficient tag |

---

## Sections of each anti-patterns.md file

Each `references/anti-patterns.md` has sections. Add the entry in the correct section, not just at the end of the file.

### gam-adops/references/anti-patterns.md

```
## Delivery and priorities
## Targeting and ad units
## Creatives and trafficking
## Programmatic yield (PG, PD, PA, Open Bidding)
## Pricing rules and floors
## Discrepancies and reporting
## Troubleshooting tools
## Mixed display+video environments
```

### prebid-adtech/references/anti-patterns.md

```
## adUnit and bidder configuration
## Auction and timeout
## GPT / googletag integration
## Price granularity and macros
## TCF/CMP from Prebid
## Specific bid adapters
## SPA and lazy loading
## User ID modules
```

### video-adtech/references/anti-patterns.md

```
## VAST and VMAP
## IMA SDK
## SSAI / DAI / SGAI
## CTV and Smart TV
## Players (Video.js, JW, Shaka, ExoPlayer)
## ID signals in CTV environments
## Ad pods and frequency in video
```

---

## Examples by finding type

### Example 1 — anti-pattern (gam-adops)

```markdown
### Sponsorship delivers zero impressions on ad unit shared with active price priority

**Type:** anti-pattern

**Symptom:** Sponsorship line item targeting a specific ad unit delivers 0 impressions
during the first hours of the flight despite having theoretical maximum priority.

**Context:** GAM non-360. Ad unit with multiple active line items including price priority
with a competitive floor CPM. Sponsorship without a competitive exclusion configured.

**Root cause:** A price priority with a high CPM can win the GAM auction before the
Sponsorship is evaluated if there is a configuration bug in the Sponsorship roadblocking
(master without mandatory companion). GAM evaluates roadblocking before priority in certain
flows — if the companion is not available, the Sponsorship is discarded and the price priority wins.

**Resolution:**
1. Verify roadblocking configuration: if the Sponsorship uses master+companion, check
   that the companion ad unit is declared in the same ad request.
2. If the inventory is display-only without a companion, remove the roadblocking configuration.
3. Add a competitive exclusion label to the price priority if the Sponsorship needs
   page exclusivity.

**Verification:** Troubleshoot tab → select the affected ad unit → verify that the Sponsorship
appears as "eligible" and not as "excluded due to roadblocking".

**Evidence level:** Verified — reproduced and confirmed in production.

**Metadata:** Captured: 2025-04 · Versions: GAM non-360 · Status: Active

**Tags:** sponsorship, price-priority, roadblocking, companion, delivery-0
```

### Example 2 — behavioral-note (prebid-adtech)

```markdown
### appnexus bid adapter ignores floor when usePrebidCacheKey is active

**Type:** behavioral-note

**Symptom:** AppNexus (now Xandr) bids win auctions below the floor declared
in the priceFloors module when the ad unit has video with usePrebidCacheKey: true.

**Context:** Prebid.js >= 7.x, priceFloors module active, appnexus bid adapter,
instream video ad unit.

**Root cause:** The AppNexus adapter in video mode with active cache sends the net CPM without
applying the floor enforcement from the adapter side. Prebid core floor enforcement
is applied post-bid, but if the adapter overwrites the CPM in response processing,
the check may be evaluated against the wrong value.

**Resolution:** Disable usePrebidCacheKey if it is not needed for the IMA setup.
Alternatively, force floor enforcement at Prebid core level with
`floors.enforcement.enforceJS: true`.

**Verification:** Prebid debug mode → filter appnexus bids → compare cpm vs floor
declared in the priceFloors module for the affected ad unit.

**Evidence level:** Probable — pattern observed in two publishers, not confirmed
as a documented bug in Prebid GitHub.

**Metadata:** Captured: 2025-04 · Versions: Prebid.js >= 7.x · Status: Pending revalidation

**Tags:** appnexus, xandr, price-floors, video, prebid-cache, floor-enforcement
```

### Example 3 — verified-fix (video-adtech)

```markdown
### IMA SDK does not fire AdStarted on Safari iOS 16+ with blocked autoplay

**Type:** verified-fix

**Symptom:** On Safari iOS 16+, the AdStarted event does not fire even though the VAST
loads correctly and the ad manager initializes without error.

**Context:** IMA SDK HTML5 version 3.x, Video.js 7.x or native player, iOS 16+,
pre-roll inventory.

**Root cause:** Safari iOS 16 tightened autoplay restrictions. If content playback
was not started by a user gesture, ad playback is also blocked — even with `playAdsAfterTime: 0`.
The IMA SDK does not propagate the error as an AdError but instead stays in a silent waiting state.

**Resolution:** Start the ad request inside the `click` or `touchend` event handler
of the user's play button. Do not call `adsLoader.requestAds()` on the player `ready` event
if there is no prior user gesture.

**Verification:** Safari Web Inspector → Console → look for autoplay policy messages.
Test with an explicit user gesture before starting the player.

**Evidence level:** Verified — behavior documented in Apple WebKit bugtracker
and reproducible in Safari iOS 16+.

**Metadata:** Captured: 2025-04 · Versions: IMA SDK 3.x, Safari iOS 16+ · Status: Active

**Tags:** ima-sdk, safari, ios-16, autoplay, AdStarted, pre-roll, Video.js
```

---

## Common errors to avoid

| Error | Correction |
|---|---|
| Abstract symptom ("the line item doesn't deliver") | Symptom in the user's terms with minimum context |
| Resolution without mechanism ("restart the line item") | Explain why the resolution works |
| Evidence level without justification | Always one line explaining the level |
| Entry added at the end of the file without a section | Locate the correct section and add it there |
| Generic tags ("gam", "prebid") | Tags that reflect the specific symptom |
