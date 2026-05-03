# TCF / CMP / Privacy Signals — Technical reference

Covers four areas:
- **TCF/GDPR:** European consent framework, base module and TCF Control Module
- **USP/CCPA:** US Privacy string
- **GPP:** Global Privacy Platform
- **NPA in GPT:** serving configuration in GAM for non-Prebid demand

---

## TCF 2.x — Operational concepts

### The TC string

Encodes purposes, vendors, legitimate interests, and CMP metadata. For practical integration, consume the signal through the CMP API or Prebid modules — not by manually parsing the string. Any logic that depends on manually parsing the string is fragile and coupled to the framework version.

### Operational impact of TCF signals in Prebid

| TCF signal | Operational impact in Prebid |
|---|---|
| Purpose 1 | usersync, User ID modules, cookie/localStorage access |
| Purpose 2 | bidder participation in the auction (requires TCF Control Module) |
| Purpose 4 | sending first-party data to partners |
| Purpose 7 | analytics adapters |
| Special Feature 1 | precise geolocation |

The specific enforcement depends on which modules are loaded in the bundle.

---

## Base module vs TCF Control Module — critical distinction

**Base module (`consentManagement`):** obtains the signal from the CMP and incorporates it into auction objects. Adapters receive the TC string in the bid request. **Does not restrict activities on its own.**

**TCF Control Module** (renamed in Prebid 9.0, previously GDPR Enforcement Module): applies restrictions to specific activities based on the signal. It is what prevents an adapter from doing usersync without Purpose 1, or from participating in the auction without Purpose 2.

This distinction is frequently overlooked. A publisher that only loads the base module is propagating the consent string, but not applying any enforcement.

### Base module configuration

```javascript
pbjs.setConfig({
  consentManagement: {
    gdpr: {
      cmpApi: 'iab',
      timeout: 10000,
      defaultGdprScope: true,
      allowAuctionWithoutConsent: false
    }
  }
});
```

**On `allowAuctionWithoutConsent`:**
- `false` is the most conservative stance: if the CMP does not respond within `timeout` ms, the auction does not launch.
- `true` reduces monetization loss from CMP timeouts, but its legal adequacy depends on the applicable framework. Prebid does not provide legal advice on this decision.

**On timeout:** starting heuristic 8000–15000ms for slow CMPs. Measure actual latency before setting the value.

### TCF Control Module

Has its own separate configuration. Do not mix with the base block. Consult official Prebid documentation for the current syntax — it was renamed and reorganized in Prebid 9.0.

---

## CMP — Interaction with the IAB TCF API

### Preferred pattern: `addEventListener`

```javascript
window.__tcfapi('addEventListener', 2, function(tcData, success) {
  if (success) {
    console.log('GDPR applies:', tcData.gdprApplies);
    console.log('Consent string:', tcData.tcString);
    console.log('Purpose 1:', tcData.purpose && tcData.purpose.consents && tcData.purpose.consents[1]);
  }
});
```

`addEventListener` is the robust pattern: it works for both returning users (captures `tcloaded`) and new users (captures `useractioncomplete`). It is the pattern Prebid modules use internally.

`getTCData` appears in historical examples but was technically deprecated as the primary query method in TCF 2.2. For new logic, use `addEventListener`.

### Relevant CMP events

| `eventStatus` | Meaning |
|---|---|
| `tcloaded` | CMP loaded a previous TC string (returning user) |
| `cmpuishown` | The CMP modal is visible |
| `useractioncomplete` | The user took action (accept/reject) |

---

## NPA in GPT — correct usage criterion

### Modern API (preferred)

```javascript
googletag.pubads().setPrivacySettings({ nonPersonalizedAds: true });
```

`setRequestNonPersonalizedAds()` is the legacy API.

### Usage criterion

Do not force NPA unconditionally:

- For Prebid demand, the consent module already delivers the signal to adapters. It is not necessary to manually set NPA to control Prebid's behavior.
- `setPrivacySettings({ nonPersonalizedAds })` primarily affects demand served directly by GAM (non-Prebid).
- The exact logic of which TCF signals imply NPA depends on the publisher's policy, not a universal normative algorithm.

### Illustrative example (not a normative algorithm)

```javascript
window.__tcfapi('addEventListener', 2, function(tcData, success) {
  if (!success || !tcData.gdprApplies) {
    googletag.pubads().setPrivacySettings({ nonPersonalizedAds: false });
    return;
  }
  // The decision of which purposes imply NPA must be defined with legal counsel
  var servePersonalized = tcData.purpose.consents[1] && tcData.purpose.consents[3];
  googletag.pubads().setPrivacySettings({ nonPersonalizedAds: !servePersonalized });
});
```

---

## CCPA / US Privacy (USP)

```javascript
pbjs.setConfig({
  consentManagement: {
    usp: { cmpApi: 'iab', timeout: 1000 }
  }
});
```

```javascript
window.__uspapi('getUSPData', 1, function(uspData, success) {
  if (success) {
    console.log('US Privacy string:', uspData.uspString);
    // Format: "1YNN" → version=1, explicit_notice=Y, opt_out=N, lspa=N
  }
});
```

For publishers with traffic across multiple US states, consider Activity Controls and GPP/USNat modules.

---

## GPP (Global Privacy Platform)

```javascript
pbjs.setConfig({
  consentManagement: {
    gpp: { cmpApi: 'iab', timeout: 10000 }
  }
});
```

```javascript
window.__gpp('addEventListener', function(evt, success) {
  if (success && evt.pingData) {
    console.log('GPP string:', evt.pingData.gppString);
    console.log('Applicable sections:', evt.pingData.applicableSections);
  }
});
```

Prebid has control modules for USNat and US states that add specific enforcement. Verify which modules are needed based on the publisher's traffic.

---

## Correct privacy signals flow

```
1. CMP initializes and exposes the API (__tcfapi, __uspapi, __gpp)
       ↓
2. Base module obtains the signal and incorporates the TC string into auction objects
       ↓
3. TCF Control Module / Activity Controls (if loaded)
   restricts specific activities based on the signal
       ↓
4a. Auction proceeds with full signal
4b. Auction cancelled (allowAuctionWithoutConsent: false + timeout expired)
4c. Auction proceeds without signal (allowAuctionWithoutConsent: true)
       ↓
5. For non-Prebid demand served by GAM:
   setPrivacySettings({ nonPersonalizedAds }) per publisher policy
```

---

## Privacy signals debugging

```javascript
// Consent incorporated in the auction:
pbjs.onEvent('auctionInit', function(data) {
  console.log('GDPR data:', data.gdprConsent);
});

// Full history:
console.log(pbjs.getEvents());

// Vendor consents directly from the CMP:
window.__tcfapi('addEventListener', 2, function(tcData) {
  console.log('AppNexus/Xandr consent:', tcData.vendor && tcData.vendor.consents[32]);
  console.log('Rubicon consent:', tcData.vendor && tcData.vendor.consents[52]);
  // Vendor IDs may change — verify against the IAB Global Vendor List
});
```

Use `auctionInit`, `pbjs.getEvents()`, and direct CMP reading to separate whether the problem is in the signal, in the timing, or in the enforcement.

---

## Consent and lazy loading

When a new auction is launched for a lazy-loaded slot, the consent module reuses the already available signal. There is no need to re-read the CMP if Prebid is correctly configured.

**Warning:** if the user changes their CMP choice during the session, validate the refresh and lazy loading behavior in the specific implementation. Do not assume the entire chain reacts uniformly without specific tests.
