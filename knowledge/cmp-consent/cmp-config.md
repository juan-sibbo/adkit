# CMP Config — Publisher-side configuration checklist

Focus on Didomi as the reference platform. Principles are agnostic; UI paths and API parameters are Didomi-specific unless otherwise indicated.

---

## Checklist by layer

### Vendor list

- [ ] GVL version up to date — verify in Didomi dashboard → Vendors → GVL version `[Didomi]`
- [ ] All active SSPs and DSPs of the publisher present in the vendor list
- [ ] Vendors not in the GVL declared as custom vendors with correct purposes
- [ ] Disabled or legacy vendors removed from the list (reduces user trust and TC string size)
- [ ] `legitimateInterests` purposes declared for vendors that require them as lawful basis
- [ ] Analytics and measurement vendors (GA, Adobe, etc.) included if used

### Purpose and feature mapping

- [ ] Purposes P1–P4 active if personalized advertising is used
- [ ] Purposes P7–P10 active if measurement, User ID, or research modules are used
- [ ] Special Features (SF1 geolocation, SF2 fingerprinting) only active if there is validated legal basis
- [ ] `purposeLegitimateInterests` configured for publishers using LI as lawful basis for P2–P10
- [ ] Unused purposes disabled — fewer purposes = less banner friction = better consent rate
- [ ] Purpose stack reviewed by legal/DPO — not just by AdOps

### Regulation configuration `[Didomi]`

- [ ] GDPR regulation active for European traffic
- [ ] Jurisdiction detection correctly configured — do not activate GDPR on US traffic unless required
- [ ] CCPA/CPRA regulation active if there is California traffic
- [ ] GPP enabled if there is US multi-state traffic `[GPP]`
- [ ] `consentableData` not blocking vendors that should be active by default
- [ ] "Apply publisher restrictions" — verify it does not restrict purposes the publisher needs active

### UI / banner configuration

- [ ] Banner appears on first scroll or load — not artificially delayed (GDPR compliance risk)
- [ ] "Reject all" button visible and accessible with the same prominence as "Accept all" (GDPR requirement)
- [ ] Granular configuration option available
- [ ] Banner language matches the page language — not just geolocation
- [ ] Link to privacy policy present and correct
- [ ] Banner does not block access to content (dark patterns — risk of sanction)

### Consent storage

- [ ] First-party cookie configured with publisher's root domain (not subdomain) `[Didomi]`
- [ ] Cookie TTL sufficient (IAB recommends 13 months for GDPR)
- [ ] `euconsent-v2` is the IAB standard cookie — verify it is what `__tcfapi` reads
- [ ] `didomi_token` and `didomi_popup_seen` with TTL consistent with consent policy `[Didomi]`
- [ ] Alternative storage in localStorage if cookies are blocked — verify the fallback works

### Technical integration with Prebid

- [ ] CMP script loaded **before** Prebid in the `<head>` — without defer/async if timing is critical
- [ ] `__tcfapi` stub available immediately (before the full CMP script loads)
- [ ] `consentManagement` configured in `pbjs.setConfig` with timeout consistent with measured real latency
- [ ] `allowAuctionWithoutConsent` — decision made with legal criteria, not technical
- [ ] Correct event configured: `tcloaded` for returning users, `useractioncomplete` for new ones
- [ ] Prebid TCF Control Module loaded if enforcement of activities by purposes is required `[TCF]`

### Technical integration with GAM/GPT

- [ ] NPA (Non-Personalized Ads) activated conditionally — only when consent was not given, not by default
- [ ] `setPrivacySettings({ nonPersonalizedAds: true })` only when `purposeConsents[2] === false` or similar
- [ ] If `googletag.pubads().setRequestNonPersonalizedAds(1)` is used — verify it is called after the CMP resolves, not before

---

## Didomi — platform-specific configuration `[Didomi]`

### Didomi SDK v2 — key parameters

```javascript
window.didomiConfig = {
  app: {
    vendors: {
      iab: { all: true },              // Include all IAB GVL vendors
      include: ['custom-vendor-id'],    // Additional custom vendors
    },
    gdprAppliesGlobally: false,        // true only if publisher wants GDPR on all traffic
  },
  consent: {
    duration: 13 * 30,                 // TTL in days (13 months)
  },
  languages: {
    enabled: ['es', 'en', 'fr'],
    default: 'es',
  },
};
```

### Didomi API — diagnostic methods

```javascript
// Vendor consent status
window.Didomi.getUserConsentStatusForVendor('vendor-id') // true/false/null

// Purpose consent status
window.Didomi.getUserConsentStatusForPurpose('cookies') // true/false/null
// IAB equivalents: 'iab.1', 'iab.2', etc.

// Current TC string
window.Didomi.getUserConsentToken() // Returns the TC string

// Verify if user has taken action
window.Didomi.getUserStatus().consent.status // 'pending', 'complete', 'partial'

// List of vendors with consent
window.Didomi.getUserStatus().vendors.consent.enabled // array of vendor IDs
```

### Didomi + Prebid — recommended integration

```javascript
// Wait for Didomi to be ready before initializing Prebid
window.didomiOnReady = window.didomiOnReady || [];
window.didomiOnReady.push(function(Didomi) {
  // Didomi is ready — Prebid can initialize
  // Prebid's consentManagement module will read the TC string via __tcfapi
});
```

Alternative: rely on Prebid's `consentManagement` module with sufficient `auctionDelay` — no explicit integration needed if the CMP script loaded before Prebid.

---

## Frequent configuration anti-patterns

### 1. Legitimate interest globally disabled

**Symptom:** vendors with LI lawful basis receive a "no consent" signal even though the user has not rejected.

**Cause:** CMP configuration that globally disables `legitimateInterests` as a "conservative" measure. This blocks purposes that vendors declare via LI, which includes many DSPs.

**Impact:** CPM drops on SSPs requiring P2/P3 via LI to bid. Fill rate drops.

### 2. Publisher-restricted purposes P1-P4

**Symptom:** TC string has user consent but vendors do not receive the active purposes.

**Cause:** the publisher has applied "publisher restrictions" in the CMP configuration that override the user's consent for certain purposes.

**Verification:** decoded TC string → `publisherRestrictions` field. If present with restricted purposes, that is the cause.

### 3. CMP script loaded after Prebid

**Symptom:** `__tcfapi` not available when Prebid initializes → `consentManagement` module fails or launches without consent.

**Cause:** CMP script added at the end of the body or with `defer` while Prebid is in the `<head>`.

### 4. Incorrect cookie domain

**Symptom:** returning users treated as new users on a different subdomain.

**Cause:** consent cookie saved at `www.publisher.com` but user visits `news.publisher.com`. The cookie is not accessible on different subdomains if not configured on the root domain.

**Fix `[Didomi]`:** configure `domain: '.publisher.com'` (with leading dot) in Didomi's storage configuration.
