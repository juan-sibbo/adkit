---
name: broadcaster-onboarding
description: Orchestrates the complete setup of a new broadcaster client (typical Spanish regional TV) on the Sibbo stack. Activate when the user says "new client", "onboarding for [TV]", "let's onboard [TV]", "sign up [name]", "we're starting with [TV]", or when they mention a regional TV station not yet in the ecosystem. Does not cover post-onboarding daily operations (that is gam-adops and programmatic-deals).
---

# Broadcaster Onboarding

**Version:** 1.0.0
**Status:** Stable
**Last updated:** 2026-04-27
**Depends on:** gam-adops, cmp-consent-flows, video-adtech, programmatic-deals, client-context-loader
**Used by:** Agent β (Broadcaster Onboarder)

> Turns the onboarding of a new broadcaster client into a guided flow
> of minimum viable configuration + a concrete task list with owners.

---

## Initial questionnaire (5 questions, in this order)

Before generating any artifact, collect this data. Ask all 5 questions in a single message:

```
To start the onboarding for [CLIENT] I need 5 pieces of information:

1. Ad delivery model:
   a) CSAI only (IMA SDK client-side in all environments)
   b) CSAI on web + SSAI/DAI on CTV
   c) SSAI/DAI only

2. CTV devices to monetize right now (mark all that apply):
   Android TV / Fire TV / Samsung Tizen / LG webOS / tvOS / HbbTV / Roku / Chromecast

3. Monetization intermediary:
   a) Newixmedia (most common)
   b) Other: [which one]
   c) Direct (no intermediary)

4. First SSP to integrate? (the most urgent or the one the client already has a relationship with)
   PubMatic / Magnite / Xandr / other: [which one]

5. Does the TV have an OTT presence (store apps) in addition to web and DTT/HbbTV?
   Yes (specify stores: Google Play / App Store / Fire TV Store / Samsung / LG / other)
   No (web only + HbbTV if applicable)
```

---

## Onboarding flow by phase

### Phase 1 — Prerequisites (verify before configuring anything)

```
□ Client domain verified (DNS, HTTPS active)
□ Client's GAM 360 access confirmed (existing account or to be created)
□ Intermediary confirmed and sellers.json verified:
   - Newixmedia: https://newixmedia.com/sellers.json — search for the publisher
   - If not found: coordinate sign-up with Newixmedia before continuing
□ Client technical contact identified (who will implement the code)
□ Client CMS identified (for future content signals integration)
```

If any prerequisite fails → document in §6 of context.md as an open issue and do not advance to Phase 2 until resolved.

### Phase 2 — Supply chain (ads.txt / app-ads.txt)

Generate the ads.txt block for the client:

```
# ads.txt — [CLIENT] — generated [date]
# Intermediary: Newixmedia

# Newixmedia (primary intermediary)
newixmedia.com, [PUBLISHER_ACCOUNT_ID], RESELLER, [TAG_ID]

# SSP 1: [name]
[ssp_domain], [account_id], DIRECT, [TAG_ID]

# SSP 2 (if applicable)
[ssp_domain], [account_id], DIRECT, [TAG_ID]

# Google (Open Bidding)
google.com, pub-[XXXXXXXXXXXXXXXX], RESELLER, f08c47fec0942fa0
```

> **Note:** exact account_id and tag_id values must come from each SSP.
> Mark as [VERIFY] any values not known at this time.

If there are CTV apps, also generate `app-ads.txt` with the same structure + bundle IDs.

### Phase 3 — GAM 360: inventory structure

Define the ad unit architecture for the new client:

**Sibbo canonical structure for regional broadcaster:**

```
/[NETWORK_CODE]/[client]/
  ├── web/
  │   ├── preroll
  │   ├── midroll
  │   └── display/
  │       ├── billboard
  │       ├── megabanner
  │       └── robapaginas
  └── ctv/
      ├── preroll
      └── midroll
```

GAM configuration checklist:
```
□ Ad network created or access verified
□ Ad units created with the above structure
□ Basic key-values created: pos, env, rdid, idtype, is_lat, ppid
□ Ad rules configured for CTV (pre-roll + mid-roll if applicable)
□ Open Bidding enabled for the client's SSPs
□ Initial yield groups configured
□ Competitive exclusions defined (telecoms if applicable)
□ Initial pricing rule: minimum floor defined with the client
```

### Phase 4 — IMA SDK: per-device configuration

For each active CTV device, generate the base ad tag:

**CSAI template — generic CTV:**
```
https://pubads.g.doubleclick.net/gampad/ads
  ?iu=/[NETCODE]/[client]/ctv/preroll
  &sz=640x480
  &gdfp_req=1&output=vast&unviewed_position_start=1
  &env=vp&impl=s
  &correlator=[CACHEBUSTER]
  &rdid=[RDID]&idtype=[IDTYPE]&is_lat=[IS_LAT]
  &ppid=[PPID]
  &gdpr=1&gdpr_consent=[GDPR_CONSENT]
  &cust_params=env%3Dctv%26pos%3Dpre
```

Adapt macros per device (see §3.2.4 of the client's context.md).

**IMA SDK checklist per device:**
```
Android TV / Fire TV:
  □ IMA Android SDK integrated in the player
  □ ExoPlayer IMA extension configured
  □ TIFA/AFAI collected and passed to the ad tag
  □ Consent signal configured if there is a CMP in the app

Samsung Tizen:
  □ IMA HTML5 SDK loaded (verify Tizen OS version for ES6 compat)
  □ RIDA obtained via Samsung Ads API
  □ Load order: googletag before IMA SDK

LG webOS:
  □ IMA HTML5 SDK loaded
  □ ID_GENERATION permission declared in appinfo.json
  □ LGUDID obtained and passed to the ad tag

tvOS / Apple TV:
  □ IMA tvOS SDK integrated with AVPlayer
  □ ATT prompt implemented before the first ad request
  □ PPID configured as the primary signal (IDFA unreliable)

HbbTV:
  □ IMA HTML5 SDK (verify terminal capability)
  □ Designed for degraded mode (no return channel on DTT)
  □ No RDID — alternative signal defined (PPID or IP)
```

### Phase 5 — CMP and consent signals

```
□ CMP identified (Didomi / OneTrust / other / none)
□ TCF v2.2 configured for web
□ TC string passed correctly in web ad tags
□ For CTV apps: consent managed at app level (see §3.2.5 of context.md)
□ SSP and GAM vendor list declared in the CMP
□ TCF purposes correctly mapped (see cmp-consent-flows for details)
```

### Phase 6 — First SSP: initial deal setup

```
□ Publisher account created in the SSP
□ Publisher seller ID verified in the SSP's sellers.json
□ Inventory mapped in the SSP platform
□ Initial floor configured (aligned with GAM pricing rule)
□ Initial deal template created (open PMP as a starting point)
□ Bid request test verified (SSP logs show traffic)
```

### Phase 7 — PPID: implementation plan

Since PPID is critical especially for tvOS and HbbTV:

```
□ Identity partner selected (LiveRamp / ID5 / own hashed email)
□ Hashing method defined (SHA-256 of registered user email)
□ Environments prioritized for implementation:
   Priority 1: tvOS (IDFA not available — PPID is the only signal)
   Priority 2: HbbTV (no RDID — PPID or IP as the only alternative)
   Priority 3: Web (complements cookies)
□ Implementation timeline agreed with the client's technical team
```

---

## Artifacts generated by this skill

Upon completing the onboarding, produce:

1. **`[client]-context.md`** — context file filled with all onboarding data (upload to Drive at `Sibbo/clients/[CLIENT]/context.md`)
2. **`[client]-ads.txt`** — draft ads.txt for client review
3. **`[client]-app-ads.txt`** — if there are CTV apps
4. **Task list in Zoho Projects** — one task per pending checklist item, with owner and estimated date
5. **Slack draft** — initial message for the client channel or for internal communication

---

## Onboarding closure checklist

```
□ context.md created and uploaded to Drive
□ ads.txt / app-ads.txt generated and sent to client for approval
□ Ad units in GAM verified (manual impression test)
□ IMA SDK on at least one CTV device generating bid requests
□ First bid received from the SSP (even if zero wins — confirms inventory is visible)
□ knowledge-capture activated with any anti-patterns detected during onboarding
```

---

## Onboarding anti-patterns

- **Advancing without verified sellers.json:** inventory will not be eligible for programmatic until the publisher appears correctly
- **Configuring IMA SDK before having the base ad tag:** the ad tag defines the macros — without it the SDK does not know what to request
- **Not defining a minimum floor:** inventory will go into open auction without price protection
- **Ignoring PPID on tvOS from the start:** fill rate on Apple TV will be structurally low until PPID is active

---

## Changelog

### v1.0.0 — 2026-04-27
**Type:** MAJOR (creation)
**Origin:** Ecosystem audit — pain point P4
- Create: initial skill with 7-phase flow
- Create: 5-question initial questionnaire
- Create: ads.txt templates and ad tags per device
- Create: checklists per phase and per CTV device
- Create: output artifact list
