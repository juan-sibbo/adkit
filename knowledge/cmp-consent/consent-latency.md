# Consent Latency — Impact on auctions and the programmatic ecosystem

## Why CMP latency matters in auctions

The Prebid auction has a total timeout (`bidderTimeout`) typically of 1000–3000ms. Within that timeout, the `consentManagement` module adds an additional wait (`auctionDelay`) for the CMP to resolve before launching bid requests.

If the CMP takes longer than `auctionDelay`:
- With `allowAuctionWithoutConsent: true` → auction launches without full TC string → TCF-compliant bidders receive empty or incomplete signal → CPM degrades or they do not bid
- With `allowAuctionWithoutConsent: false` → auction blocked until CMP resolves → total page latency increases → risk of total timeout

**CMP latency is not uniform** — it depends on user type and device:

| User type | Typical latency | Source of variability |
|---|---|---|
| Returning (valid cookie) | 20–80ms | localStorage/cookie read |
| Returning (expired cookie) | 100–300ms | Re-fetch of prior consent |
| New user (GDPR) | 500ms–5s+ | Depends on when they interact |
| New user (non-GDPR) | 30–100ms | CMP resolves without showing banner |
| Safari/ITP | 200–500ms extra | ITP deletes cookies → more "new" users |
| Slow mobile | 2x–3x desktop | CMP script load time |

---

## How to measure CMP latency in production

### Method 1 — Manual timeline in DevTools

```javascript
// In console, before the page loads
const t0 = performance.now();
__tcfapi('addEventListener', 2, (tcData) => {
  if (tcData.eventStatus === 'tcloaded' || tcData.eventStatus === 'useractioncomplete') {
    console.log('CMP resolved in:', performance.now() - t0, 'ms');
    console.log('eventStatus:', tcData.eventStatus);
  }
});
```

### Method 2 — Correlation with pbjs.getEvents()

```javascript
// After the auction
const events = pbjs.getEvents();
const auctionInit = events.find(e => e.eventType === 'auctionInit');
const firstBidRequested = events.find(e => e.eventType === 'bidRequested');

console.log('Auction init:', auctionInit?.ts);
console.log('First bid requested:', firstBidRequested?.ts);
// If gap > configured auctionDelay → CMP took longer than expected
```

### Method 3 — Didomi SDK events `[Didomi]`

```javascript
window.didomiOnReady = window.didomiOnReady || [];
window.didomiOnReady.push(function(Didomi) {
  console.log('Didomi ready at:', performance.now());
  Didomi.on('consent.changed', function(e) {
    console.log('Consent changed at:', performance.now());
  });
});
```

### Method 4 — RUM / Analytics

Instrument in production with custom events:
- `cmp_resolved` with dimensions: `eventStatus`, `gdprApplies`, `time_ms`, `user_type` (returning/new)
- Correlate with `auction_started` and fill rate metrics

---

## Mitigation strategies by scenario

### Scenario A — Majority of returning traffic (>70%)

**Symptom:** low median latency but high 95th percentile (new users raise the tail).

**Strategy:** `auctionDelay` calibrated for returning users (200–400ms). For new users, accept that the first auction may launch without full consent if `allowAuctionWithoutConsent: true`. Revenue on the first session is lower anyway.

**Trade-off:** compliance risk if `allowAuctionWithoutConsent: true` is activated in a strict GDPR jurisdiction. Legal decision, not technical.

### Scenario B — High % of new traffic (viral news publishers, social media)

**Symptom:** high median latency, low fill rate on first visit.

**Strategies by order of impact:**

1. **Optimize CMP script load time** — load CMP as early as possible in the `<head>`, before Prebid. Never defer/async if avoidable for the CMP.

2. **Pre-compute the consent decision** — if the publisher can assume users outside GDPR do not need a banner, configure jurisdiction detection not to show a banner on non-GDPR traffic. Drastically reduces latency for those users.

3. **Soft opt-in / implied consent** — in jurisdictions where permitted, the CMP can resolve with "implied" consent before user interaction. Reduces latency to <100ms. Requires legal validation.

4. **Scroll/click as banner trigger** — delaying the banner until the user interacts reduces perceived friction but does not reduce auction latency (the auction still waits).

5. **Adaptive `auctionDelay`** — not natively available in Prebid, but can be implemented with custom logic: if the consent cookie already exists, reduce `auctionDelay`; if not, use the longer value. Requires custom code before `pbjs.setConfig`.

### Scenario C — Safari / ITP dominant (Apple-heavy publishers, iOS)

**Symptom:** apparently normal consent rate but high latency and low fill on Safari.

**Cause:** ITP deletes third-party cookies and may delete localStorage before TTL → more users treated as new → more frequent banner → higher latency.

**Strategy:** store consent in a first-party cookie with the publisher's explicit domain. `[Didomi]` Didomi supports storage in first-party cookies — verify it is configured and that the domain matches the root domain (not subdomain).

### Scenario D — CMP script as single point of failure

**Symptom:** when the CMP's CDN has an incident, the auction is blocked or launches without consent.

**Strategy:** configure a maximum timeout in Prebid's `consentManagement` (separate from `auctionDelay` — it is the absolute timeout). If exceeded, Prebid acts according to `allowAuctionWithoutConsent`. Have a clear policy defined for this case.

---

## Consent latency diagnostic metrics

| Metric | What it indicates | Alert threshold |
|---|---|---|
| P50 CMP latency | Median user experience | >300ms warrants investigation |
| P95 CMP latency | Experience of the problematic percentile | >2000ms → fill rate at risk |
| % auctions without TC string | Auctions that started before CMP | >5% → auctionDelay insufficient |
| Fill rate returning vs new | Consent impact on monetization | >20% difference → optimizable |
| Consent rate per purpose | Which purposes users block | P2 < 60% → CPM degrades significantly |

---

## Interaction with User ID modules in Prebid

User ID modules (ID5, LiveRamp, UID2, SharedID) require Purpose 1 to access cookies/localStorage. If the TC string does not have P1 granted:

- Modules cannot read or write identifiers → bid requests without user ID → reduced CPM
- `auctionDelay` must be sufficient for both the CMP and the User ID modules to resolve

**Critical timing:**
```
CMP resolves (TC string with P1)
        → User ID modules read/write identifiers
                → Prebid launches bid requests with user IDs
```

If `auctionDelay` is correct for the CMP but insufficient for User ID modules to resolve afterwards, bids launch without identifiers. Verify with `pbjs.getUserIds()` that IDs are present before the first `bidRequested`.
