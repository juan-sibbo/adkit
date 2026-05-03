# TCF Signal Flow — CMP → Prebid → GAM

## The complete flow

```
User arrives on the page
        │
        ▼
[1] CMP script loaded (async)
        │
        ▼
[2] CMP initializes → __tcfapi stub available
        │
        ▼
[3] Does user have a cookie/localStorage with prior consent?
        ├── Yes → [4a] CMP resolves in <50ms (returning user)
        └── No  → [4b] Consent banner shown → waits for user interaction
                        │
                        ▼
                [4c] User accepts/rejects/configures
                        │
                        ▼
[5] CMP emits tcloaded or useractioncomplete event
        │
        ▼
[6] TC string built and available via __tcfapi('getTCData', 2, cb)
        │
        ├──→ [7a] Prebid: consentManagement module receives TC string
        │           └── If auctionDelay has expired: auction launched without consent
        │
        ├──→ [7b] GPT/GAM: googletag receives NPA signal or full consent
        │
        └──→ [7c] Vendors: each SSP/DSP receives TC string in bid request
```

---

## Failure points by stage

### [1-2] CMP script does not load or loads late

**Symptoms:** `__tcfapi` not defined in console, Prebid launches without consent, vendors without signal.

**Frequent causes:**
- CMP script blocked by ad blocker or corporate network firewall
- Script loaded synchronously blocking the parser → loads late → Prebid already launched
- Conflict with another script that defines `__tcfapi` before the real CMP (stub incorrectly implemented)

**Verification:** `typeof __tcfapi` in console before and after the script loads. If it returns `undefined`, the script has not loaded.

### [3-4] Consent cookie/localStorage unavailable or corrupt

**Symptoms:** banner appears for users who should be returning, consent rate artificially low.

**Frequent causes:** `[Didomi]`
- `euconsent-v2` corrupt or with version mismatch (v1 TC string saved, CMP expects v2)
- Incorrect cookie domain — cookie saved on subdomain, not root domain
- SameSite/Secure policy preventing reading in iframe context
- ITP in Safari deleting the cookie before the declared TTL

**Verification:** Application tab → Cookies → look for `euconsent-v2` (IAB) or `didomi_token` / `didomi_popup_seen` `[Didomi]`. Check domain, expiration, and value.

### [5] CMP event does not fire or fires with incorrect data

**Symptoms:** Prebid waits indefinitely, `tcloaded` never reaches the `consentManagement` module.

**Frequent causes:**
- CMP configured to emit only `useractioncomplete` but Prebid expects `tcloaded`
- CMP in "soft opt-in" mode marking implicit consent without emitting the correct event
- Race condition: Prebid listener registered after the event → it is missed

**Verification:**
```javascript
__tcfapi('addEventListener', 2, (tcData, success) => {
  console.log('eventStatus:', tcData.eventStatus);
  console.log('tcString:', tcData.tcString);
  console.log('gdprApplies:', tcData.gdprApplies);
});
```
`eventStatus` must be `tcloaded` (returning) or `useractioncomplete` (after interaction).

### [6] Malformed TC string or with incorrect purposes

**Symptoms:** vendors receive signal but do not bid, CPM drops, Prebid adapters ignore the auction.

**How to decode the TC string:**
- https://iabeurope.eu/tcf-2-0/ → TCF Consent String Decoder
- Library: `@iabtechlab/consent-string` (npm)
- Didomi console → "TC String" tab `[Didomi]`

**What to verify in the decoded string:**
- `version` = 2 (not 1)
- `gdprApplies` = true if we are on European traffic
- `purposeConsents` — map of purposes with consent given (1 = granted)
- `purposeLegitimateInterests` — purposes with legitimate interest as lawful basis
- `vendorConsents` — vendors with consent (by GVL ID)
- `isServiceSpecific` — if true, the string is valid only for that domain

**Frequent problem — legitimate interest vs consent:**
Many vendors declare purposes via legitimate interest (LI) in the GVL. If the publisher globally blocks LI in the CMP configuration, those vendors appear as "without consent" even though the user has not actively rejected them. Verify `purposeLegitimateInterests` in addition to `purposeConsents`.

### [7a] Prebid receives TC string but launches before it is ready

See `references/consent-latency.md` for full analysis.

**Quick verification:**
```javascript
pbjs.getEvents().filter(e => e.eventType === 'auctionInit')
// Compare timestamp with the CMP's useractioncomplete event
```

---

## Vendor list — critical points

### GVL (Global Vendor List) version

The CMP downloads the GVL periodically. If it uses an old version:
- New vendors do not appear in the banner UI
- Vendors with updated purposes show outdated purposes
- Generated TC string may not be valid for vendors that migrated to new purposes

`[Didomi]` — the active GVL version can be verified in the Didomi dashboard → "Vendor list" → "GVL version".

### Custom vendors vs GVL

Vendors not in the GVL must be declared as custom vendors. If an SSP or DSP is not in the GVL and is not declared as a custom vendor, it will not receive a consent signal even if the user has accepted.

**Frequent anti-pattern:** publishers using a relatively new SSP (outside the GVL) but not having added it as a custom vendor. The SSP operates without a TCF signal and some DSPs reject it.

### Purposes and publisher-side mapping

The publisher can restrict purposes even if the user has accepted them. In Didomi this is configured at the "regulation" level and can globally disable purposes. `[Didomi]`

**Verify that:**
- The purposes needed for the auction (P1, P2, P3, P4 minimum) are not restricted at the regulation configuration level
- `legitimateInterestsPurposes` declared for vendors that require it as lawful basis

---

## GPP (Global Privacy Platform)

`[GPP]` — applies when there is multi-jurisdiction traffic (US states + GDPR).

**GPP string structure:**
- Header: which sections are present
- TCF section (GDPR): embedded TC string
- US sections: CPRA (California), VCDPA (Virginia), etc.

**Jurisdiction detection — how it works:**
The CMP detects the user's jurisdiction (IP geolocation + headers) and activates the corresponding regulation. The most frequent problems:

1. **False positive GDPR on US traffic** — incorrect geolocation → GDPR banner for American users → unnecessarily low consent rate
2. **False negative GDPR on European traffic** — the opposite → compliance risk
3. **Multi-jurisdiction badly handled** — user with VPN activating two regulations simultaneously → malformed GPP string

**Jurisdiction detection verification:** `[Didomi]`
```javascript
window.Didomi.getUserConsentStatusForPurpose('cookies') // undefined if GDPR does not apply
// vs
__tcfapi('getTCData', 2, (data) => console.log(data.gdprApplies))
```
