# Amazon Publisher Services (APS / TAM) — Coexistence with Prebid

Load only if mentioned: `apstag`, `APS`, `TAM`, `Amazon Publisher Services`.

---

## Integration model

Amazon APS/TAM operates in parallel with Prebid — not as another bidder inside the Prebid auction, but as an independent simultaneous auction. Coordination happens in GAM: both Prebid and APS set key-values on GPT slots, and GAM decides the winner between both demand sources.

```
Browser
├── Prebid auction (all configured adapters)
└── APS auction (apstag.fetchBids)
         ↓ both respond
GAM reads key-values from Prebid (hb_pb, hb_bidder...) and from APS (amznbid, amzniid...)
         ↓
GAM selects the winner across all demand sources
```

---

## Initialization

```javascript
apstag.init({
  pubID: 'YOUR_PUBLISHER_ID',
  adServer: 'googletag',
  gdpr: { cmpTimeout: 3000 }
});
```

---

## Prebid + APS coordination pattern

Launch both auctions in parallel and wait for both to complete before refreshing GAM:

```javascript
var prebidComplete = false;
var apsComplete = false;

function sendAdserverRequest() {
  if (prebidComplete && apsComplete) {
    googletag.cmd.push(function() {
      apstag.setDisplayBids();           // APS before Prebid targeting
      pbjs.setTargetingForGPTAsync();
      googletag.pubads().refresh();
    });
  }
}

// APS auction
apstag.fetchBids({ slots: apstagSlots, timeout: 2000 }, function(bids) {
  apsComplete = true;
  sendAdserverRequest();
});

// Prebid auction
pbjs.que.push(function() {
  pbjs.addAdUnits(adUnits);
  pbjs.requestBids({
    timeout: 2000,
    bidsBackHandler: function() {
      prebidComplete = true;
      sendAdserverRequest();
    }
  });
});
```

**Critical:** `apstag.setDisplayBids()` must be called **before** `pbjs.setTargetingForGPTAsync()` and before `refresh()`. If omitted or called after, APS key-values do not reach GAM.

---

## APS key-values in GAM

| Key | Description |
|---|---|
| `amznbid` | Amazon bid (encoded, not a direct CPM) |
| `amzniid` | Amazon creative ID |
| `amznp` | Position/placement |

Line items must exist in GAM configured for these key-values, with the same priority logic as Prebid line items.

---

## Timeout — coordination

The `apstag.fetchBids` timeout and Prebid's `bidderTimeout` must be consistent. If one is significantly lower than the other, `sendAdserverRequest` will be called with one completed and the other in flight. Heuristic: use the same value or close to it. Adjust based on actual measured latency.

---

## Consent / GDPR with APS

```javascript
apstag.init({
  pubID: 'YOUR_PUBLISHER_ID',
  adServer: 'googletag',
  gdpr: { cmpTimeout: 3000 }  // coordinate with Prebid's consentManagement module timeout
});
```

---

## APS-specific anti-patterns

🔴 **`apstag.setDisplayBids()` omitted or called after `refresh()`:** APS key-values do not reach GAM.

🔴 **Sequential auctions instead of parallel:** unnecessarily increases total latency.

🟡 **APS timeout much lower than Prebid's:** the refresh may occur without Prebid targeting.

🟡 **APS line items not configured in GAM:** APS responds with bids but there are no line items to capture them.
