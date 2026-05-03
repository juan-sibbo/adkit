# Anti-patterns in Publisher-Side AdTech

## Severity classification

🔴 **Critical:** revenue loss, legal non-compliance, or production failure
🟡 **Important:** performance or maintainability degradation
🟢 **Minor:** style, readability, or non-urgent technical debt

---

## PREBID

### 🔴 `requestBids` without `bidsBackHandler`

```javascript
// BAD
pbjs.requestBids({ adUnits: adUnits });
googletag.pubads().refresh(); // executes immediately, without waiting for bids

// GOOD
pbjs.requestBids({
  adUnits: adUnits,
  timeout: 1500,
  bidsBackHandler: function() {
    pbjs.setTargetingForGPTAsync();
    googletag.pubads().refresh();
  }
});
```

**Why it is critical:** without `bidsBackHandler`, the GAM refresh can occur before Prebid has set targeting. Prebid line items never win.

---

### 🔴 `pbjs.setConfig` called multiple times with partial configs

```javascript
// BAD: nested configs (like userSync.filterSettings) can be overwritten
pbjs.setConfig({ timeout: 2000 });
pbjs.setConfig({ priceGranularity: 'dense' });
pbjs.setConfig({ userSync: { ... } });

// GOOD: a single consolidated call before requestBids
pbjs.setConfig({
  bidderTimeout: 2000,
  priceGranularity: 'dense',
  userSync: { ... }
});
```

---

### 🔴 `s2sConfig.timeout` greater than `bidderTimeout`

```javascript
// BAD: Prebid cancels the auction before S2S responds
pbjs.setConfig({
  bidderTimeout: 1000,
  s2sConfig: { timeout: 1500 }
});

// GOOD
pbjs.setConfig({
  bidderTimeout: 1500,
  s2sConfig: { timeout: 1000 }
});
```

**Verified rule:** `s2sConfig.timeout` must always be strictly less than `bidderTimeout`.

---

### 🔴 Double `addAdUnits` without prior `removeAdUnit` (SPA / refresh)

```javascript
// BAD: duplicate bids and race conditions on every navigation
pbjs.addAdUnits(adUnits);
pbjs.requestBids({ ... });
// ... SPA navigation ...
pbjs.addAdUnits(adUnits);  // DUPLICATE
pbjs.requestBids({ ... });

// GOOD — option A: explicit removeAdUnit
pbjs.removeAdUnit('div-slot-1');
pbjs.addAdUnits(updatedAdUnits);
pbjs.requestBids({ ... });

// GOOD — option B: adUnitCodes to limit scope
pbjs.requestBids({
  adUnitCodes: ['div-slot-1', 'div-slot-2'],
  timeout: 1500,
  bidsBackHandler: function() { ... }
});
```

---

### 🟡 `enableSendAllBids` without `targetingControls`

```javascript
// BAD: key explosion (10 bidders × 6 keys = 60 key-values per slot)
pbjs.setConfig({ enableSendAllBids: true });

// GOOD
pbjs.setConfig({
  enableSendAllBids: true,
  targetingControls: {
    allowTargetingKeys: ['BIDDER', 'AD_ID', 'PRICE_BUCKET', 'SIZE', 'FORMAT']
  }
});
```

---

### 🟡 `auctionDelay: 0` with User ID modules

```javascript
// BAD: adapters bid without User ID → lower CPM
pbjs.setConfig({
  userIds: [{ name: 'id5Id', params: { partner: 173 } }],
  userSync: { auctionDelay: 0 }
});

// GOOD — starting heuristic: 200–500ms; adjust based on measured real latency
pbjs.setConfig({
  userSync: { auctionDelay: 300 }
});
```

---

### 🟡 Adapters with test params in production

```javascript
// BAD: CPM always ~$0.01 or null
{ bidder: 'appnexus', params: { placementId: 1234 } }
// Symptom: the adapter responds, but with a test CPM.
```

---

### 🟡 `mediaTypes` incorrectly defined for video

```javascript
// BAD: sizes in video (ignored or treated as banner)
{ mediaTypes: { video: { sizes: [[640, 480]] } } }

// GOOD
{
  mediaTypes: {
    video: {
      playerSize: [[640, 480]],
      context: 'instream',
      mimes: ['video/mp4', 'video/webm'],
      protocols: [1, 2, 3, 4, 5, 6, 7, 8]
    }
  }
}
```

---

## GAM / GPT

### 🔴 GAM refresh without a new Prebid auction

```javascript
// BAD: bid TTL may have expired
googletag.pubads().refresh([slot]);

// GOOD: always pair the refresh with a new auction
pbjs.requestBids({
  adUnitCodes: [slotCode],
  timeout: 1500,
  bidsBackHandler: function() {
    pbjs.setTargetingForGPTAsync([slotCode]);
    googletag.pubads().refresh([slot]);
  }
});
```

---

### 🔴 `googletag.display()` called more than once on the same slot

```javascript
// BAD: duplicates the slot in the DOM
googletag.display('div-slot-1');
googletag.display('div-slot-1');  // NOT on refresh

// GOOD: display() only on first load; refresh() to reload
googletag.pubads().refresh([slot]);
```

---

### 🔴 Unconditional `setPrivacySettings` / NPA

```javascript
// BAD: forces NPA on all traffic, destroys revenue for users with consent
googletag.pubads().setPrivacySettings({ nonPersonalizedAds: true });

// GOOD: conditional on consent, using addEventListener (robust TCF 2.x pattern)
window.__tcfapi('addEventListener', 2, function(tcData, success) {
  if (!success || !tcData.gdprApplies) {
    googletag.pubads().setPrivacySettings({ nonPersonalizedAds: false });
    return;
  }
  // The exact policy depends on the publisher's legal implementation
  var servePersonalized = tcData.purpose.consents[1] && tcData.purpose.consents[3];
  googletag.pubads().setPrivacySettings({ nonPersonalizedAds: !servePersonalized });
});
```

**Note:** `setRequestNonPersonalizedAds()` is the legacy API. The modern API is `setPrivacySettings({ nonPersonalizedAds })`.

---

### 🟡 SRA with lazy loading

SRA groups all slots into a single request. It is not incompatible with lazy loading, but requires that **all slots are defined and configured before the first `display()` or `refresh()`**. If you cannot guarantee that, individual per-slot requests are safer.

---

### 🟡 `setTargetingForGPTAsync` before slots exist

```javascript
// BAD: silent no-op if slots are not defined
pbjs.setTargetingForGPTAsync();

// GOOD: inside googletag.cmd.push, after defineSlot
googletag.cmd.push(function() {
  pbjs.setTargetingForGPTAsync();
});
```

---

## CONSENT / TCF

### 🔴 Auction launched before the CMP is ready

```javascript
// BAD
window.onload = function() {
  pbjs.requestBids({ ... });  // may execute without consent available
};

// GOOD: let the consentManagement module handle the wait
pbjs.setConfig({
  consentManagement: {
    gdpr: { cmpApi: 'iab', timeout: 10000, allowAuctionWithoutConsent: false }
  }
});
```

---

### 🔴 Custom adapter not propagating consent

```javascript
// BAD: bid request without gdpr/consent_string → TCF non-compliance
function buildRequests(validBidRequests) {
  return validBidRequests.map(function(bid) {
    return { method: 'POST', url: '...', data: JSON.stringify({ placement: bid.params.id }) };
  });
}

// GOOD: use bidderRequest (second argument) to propagate consent
function buildRequests(validBidRequests, bidderRequest) {
  return validBidRequests.map(function(bid) {
    var payload = { placement: bid.params.id };
    if (bidderRequest.gdprConsent) {
      payload.gdpr = bidderRequest.gdprConsent.gdprApplies ? 1 : 0;
      payload.gdpr_consent = bidderRequest.gdprConsent.consentString;
    }
    return { method: 'POST', url: '...', data: JSON.stringify(payload) };
  });
}
```

---

### 🟡 CMP timeout too short

```javascript
// Problematic with slow CMPs (OneTrust under heavy load, Didomi on first visits)
pbjs.setConfig({ consentManagement: { gdpr: { timeout: 500 } } });

// Starting heuristic: 8000–15000ms.
// Measure actual CMP latency in the environment before setting the value.
```

---

## PERFORMANCE

### 🟡 Prebid loaded synchronously

```html
<!-- BAD: blocks render -->
<script src="prebid.js"></script>

<!-- GOOD -->
<script async src="prebid.js"></script>
```

---

### 🟢 adUnits defined inline (not reusable)

```javascript
// Better: separate and reusable definition
const adUnitConfig = require('./adUnits.config');
pbjs.addAdUnits(adUnitConfig.getUnits('homepage'));
```

---

## Symptom → quick diagnosis checklist

| Symptom | Probable cause | Confidence level |
|---|---|---|
| Prebid bids win but ad is not shown | `renderAd` not executed or incorrect creative snippet | Probable |
| All adapters return no bid | Timeout too low, incorrect params, or consent blocking | Requires telemetry |
| Prebid bids never win the GAM line item | Targeting not set before refresh, or misaligned price bucket | Probable |
| CPMs always very low (~$0.01) | Test placementIds in production | Probable |
| Revenue drops after CMP change | Consent not reaching adapters, insufficient timeout, or TCF Control Module blocking | Requires telemetry |
| Prebid key-values not appearing in GAM | `disableInitialLoad()` missing, or `refresh()` before targeting | Probable |
| Double auction in SPA | `addAdUnits` without prior `removeAdUnit` | Verified |
| User ID not reaching adapters | `auctionDelay: 0` with identity modules | Probable |
| S2S bidders never win | `s2sConfig.timeout` >= `bidderTimeout` | Verified |
| CPM drops ~30% when migrating to S2S | Low cookie match rate in PBS | Probable |
