# Prebid.js Core — Technical reference

Always validate the exact deployed version and module compatibility before applying any recommendation from this reference.

---

## `pbjs.setConfig` — critical options

### `priceGranularity`

```javascript
pbjs.setConfig({ priceGranularity: 'dense' });

// Custom (when the real CPM range justifies it)
pbjs.setConfig({
  priceGranularity: {
    buckets: [
      { max: 5,  increment: 0.05 },
      { max: 10, increment: 0.10 },
      { max: 20, increment: 0.50 }
    ]
  }
});
```

[Heuristic]: `dense` offers more granularity but is not necessarily the right balance for all publishers. It depends on the real CPM range and the line item design in GAM.

Critical rule: `priceGranularity` buckets must be aligned with GAM line items. A mismatch causes silent revenue loss. [Verified]

### `bidderTimeout`

[Heuristic]: <500ms → slow adapters never win; >3000ms → impact on UX and Core Web Vitals. The optimal value depends on the real latency of the environment.

### `enableSendAllBids`

With many bidders, always pair with `targetingControls`:

```javascript
pbjs.setConfig({
  enableSendAllBids: true,
  targetingControls: {
    allowTargetingKeys: ['BIDDER', 'AD_ID', 'PRICE_BUCKET', 'SIZE', 'FORMAT']
  }
});
```

### `userSync`

```javascript
pbjs.setConfig({
  userSync: {
    filterSettings: {
      iframe: { bidders: ['appnexus', 'rubicon'], filter: 'include' },
      image:  { bidders: '*', filter: 'include' }
    },
    syncsPerBidder: 5,
    syncDelay: 3000,
    auctionDelay: 0 // increase if Identity modules are active
  }
});
```

---

## Modules — verification in the bundle

Critical rule: if the module is not compiled into the bundle, the configuration is silently ignored with no warning. [Verified]

| Config | Module required in bundle |
|---|---|
| `floors` | Price Floors Module (`priceFloors`) |
| `userIds` | User ID Module + specific submodules (`id5Id`, `sharedId`, etc.) |
| `consentManagement.gdpr` | GDPR Consent Management Module |
| `consentManagement.usp` | USP Consent Management Module |
| `consentManagement.gpp` | GPP Consent Management Module |
| `s2sConfig` | Prebid Server S2S Module (`prebidServerBidAdapter`) |
| `currency` | Currency Module |

Verify with `pbjs.installedModules` or by reviewing the compiled bundle.

Build checklist:
- Included modules match those used in config
- Adapters are actually present in the bundle
- Reasonable size [Heuristic: <500KB gzipped]
- Prebid loaded with `async`

---

## Bid Lifecycle

```
1. pbjs.addAdUnits(units)
2. pbjs.requestBids({ bidsBackHandler, timeout })
   ├─ Prebid sends bid requests to each adapter
   ├─ Adapters respond with bids (or timeout)
   └─ On completion (or timeout):
       3. bidsBackHandler executes
          ├─ pbjs.setTargetingForGPTAsync()  ← MUST be here
          └─ googletag.pubads().refresh(slots) ← AFTER targeting
```

Bid response structure:

```javascript
{
  bidderCode: 'appnexus',
  adId: 'abc123',
  cpm: 2.45,
  currency: 'USD',
  width: 300, height: 250,
  ad: '<div>...</div>',
  ttl: 300,           // seconds before the bid expires
  creativeId: '...',
  netRevenue: true,
  mediaType: 'banner'
}
```

---

## AdUnit — structure

### Banner

```javascript
const adUnits = [{
  code: 'div-gpt-ad-123456789-0',
  mediaTypes: {
    banner: { sizes: [[300, 250], [300, 600]] }
  },
  bids: [{
    bidder: 'appnexus',
    params: { placementId: 12345678 }
  }]
}];
```

### Video / Instream

```javascript
{
  code: 'video-unit',
  mediaTypes: {
    video: {
      context: 'instream',        // instream | outstream | adpod
      playerSize: [[640, 480]],   // playerSize, never sizes [Verified]
      mimes: ['video/mp4', 'video/webm'],
      protocols: [1, 2, 3, 4, 5, 6, 7, 8],
      playbackmethod: [2],
      skip: 1,
      minduration: 0,
      maxduration: 30,
      startdelay: 0
    }
  },
  bids: [...]
}
```

---

## Web video — auction↔player timing

The most frequent operational gap in video implementations:

Problem: the player makes the VAST request when it is ready to play. If the Prebid auction has not completed at that point, the player does not wait and serves without a Prebid bid.

Rule: the auction must complete BEFORE the player requests the VAST URL. [Verified]

```javascript
// Correct flow: auction first, player second
pbjs.requestBids({
  adUnitCodes: ['video-unit'],
  timeout: 2000,  // [Heuristic: video needs more time than banner]
  bidsBackHandler: function(bids) {
    var videoUrl = pbjs.adServers.dfp.buildVideoUrl({
      adUnit: videoAdUnit,
      params: { iu: '/1234/video-unit', sz: '640x480', output: 'vast' }
    });
    // Pass videoUrl to the player at this point, not before
    player.src({ type: 'video/mp4', src: videoUrl });
  }
});
```

`vastUrl` vs `vastXml`:
- `vastUrl`: the player makes an HTTP request to the Prebid Server cache. Requires a cache endpoint.
- `vastXml`: the VAST is inlined in the bid response. No cache required, but some players do not accept `vastXml` directly.
- Verify what the adapter delivers and what the player accepts before assuming. [Requires telemetry if there are discrepancies]

Cache endpoint:
```javascript
pbjs.setConfig({
  cache: { url: 'https://prebid.adnxs.com/pbc/v1/cache' }
});
```
Verify the active endpoint with the PBS provider — it may have been deprecated.

Ad pods (`adpod`): require `durationRangeSec` and `requireExactDuration`. Advanced case — consult docs.prebid.org for the deployed version.

Video checklist:
- `playerSize` defined (not `sizes`) [Verified]
- Correct `context`: `instream`, `outstream`, or `adpod` [Verified]
- `mimes` covers the player's formats
- `protocols` covers VAST 2.0+ and VPAID if applicable
- `renderer` if outstream (or the adapter provides one)
- Cache endpoint configured if using Prebid Server
- `vastUrl` vs `vastXml` verified against the player
- Auction completed before the player requests the VAST

---

## Identity Modules (User ID)

```javascript
pbjs.setConfig({
  userSync: {
    auctionDelay: 300  // [Heuristic: 200–500ms] — adjust to actual latency
  },
  userIds: [{
    name: 'id5Id',
    params: { partner: 173 },
    storage: { type: 'cookie', name: 'id5id', expires: 90 }
  }, {
    name: 'sharedId',
    storage: { type: 'cookie', name: '_pubcid', expires: 365 }
  }, {
    name: 'uid2',
    params: { uid2ApiBase: 'https://prod.uidapi.com' }
  }]
});
```

Without `auctionDelay`, adapters bid without User ID → lower resulting CPM. [Verified]
IDs using cookies/localStorage require TCF Purpose 1 on GDPR traffic. [Verified]

---

## Price Floors Module

```javascript
pbjs.setConfig({
  floors: {
    enforcement: { enforceJS: true, enforcePBS: true },
    data: {
      currency: 'USD',
      skipRate: 0,
      schema: { fields: ['mediaType', 'size'] },
      values: {
        'banner|300x250': 0.50,
        'banner|300x600': 1.00,
        'video|640x480': 2.00,
        '*|*': 0.25  // mandatory fallback
      }
    }
  }
});
```

Remote floors:

```javascript
pbjs.setConfig({
  floors: {
    auctionDelay: 500,  // ms to wait for the remote JSON
    endpoint: { url: 'https://floors.example.com/floors.json' }
  },
  bidderTimeout: 2000   // must be greater than floors.auctionDelay + adapter latency
});
```

`floors.auctionDelay` is different from `userSync.auctionDelay` — independent configurations. [Verified]
If the remote JSON does not arrive before the delay, Prebid uses the local fallback. A very low `*|*` fallback can nullify the effect of dynamic floors.

---

## Server-Side Bidding (S2S)

```javascript
pbjs.setConfig({
  s2sConfig: [{
    accountId: 'my-account',
    bidders: ['appnexus', 'rubicon'],
    defaultVendor: 'appnexus',
    timeout: 1000,   // MUST be less than bidderTimeout [Verified]
    endpoint: { p1Consent: 'https://prebid.adnxs.com/pbs/v1/openrtb2/auction' }
  }]
});
```

Operational rules:
- `s2sConfig.timeout` < `bidderTimeout` always. [Verified]
- When migrating to S2S: sync cookies are not automatically shared with PBS. This can reduce CPM on identity cookie-dependent bidders. [Verified in practice]

---

## Diagnostic APIs

```javascript
pbjs.getBidResponses()   // all bids received per adUnit
pbjs.getNoBids()         // adUnits with no bids
pbjs.getEvents()         // full timeline

pbjs.setConfig({ debug: true });
// or: ?pbjs_debug=true
```

For winning bid APIs, verify they exist in the deployed version — naming has varied across versions. [Requires version-specific verification]

Network tab: always combine with the Prebid console.
- HTTP 200 does not guarantee a valid bid — review the response body
- Some adapters return 200 with an empty body on no-bid

Key events:

```javascript
pbjs.onEvent('auctionEnd',  function(data) { console.log('Auction ended:', data); });
pbjs.onEvent('bidWon',      function(bid)  { console.log('Bid won:', bid.bidderCode, bid.cpm); });
pbjs.onEvent('noBid',       function(bid)  { console.log('No bid from:', bid.bidder); });
pbjs.onEvent('bidTimeout',  function(bids) { console.log('Timed out:', bids.map(b => b.bidder)); });
```

---

## SPA — adUnit management between navigations

```javascript
// When unmounting the current view:
pbjs.removeAdUnit('div-slot-1');
pbjs.removeAdUnit('div-slot-2');

// GPT: destroy slots (see gam-gpt.md, SPA section)
// In the new view: addAdUnits + requestBids again
```

Without `removeAdUnit`, a second call to `addAdUnits` with the same codes duplicates adUnits → duplicate bid requests and race conditions. [Verified]

---

## Performance

[Heuristics] — measure in the real environment:
- More than 10–12 adapters rarely improves revenue and does increase latency [Heuristic]
- `syncDelay: 3000ms` so syncs do not compete with page resources during load [Heuristic]
- Identify slow adapters via `pbjs.getEvents()` before adjusting `bidderTimeout`
