# GAM / GPT — Integration with Prebid

---

## Base pattern (banner, manual sequencing)

```html
<script async src="https://securepubads.g.doubleclick.net/tag/js/gpt.js" crossorigin="anonymous"></script>
<script async src="prebid.js"></script>

<script>
window.googletag = window.googletag || { cmd: [] };
window.pbjs     = window.pbjs     || { que: [] };

var slot;

googletag.cmd.push(function() {
  slot = googletag
    .defineSlot('/1234567/banner-300x250', [300, 250], 'div-gpt-ad-123456789-0')
    .addService(googletag.pubads());

  googletag.pubads().disableInitialLoad();
  googletag.pubads().enableSingleRequest();
  googletag.enableServices();

  // display() registers the slot. With disableInitialLoad() active,
  // it does not fire the ad server request.
  googletag.display('div-gpt-ad-123456789-0');
});

pbjs.que.push(function() {
  pbjs.addAdUnits(adUnits);
  pbjs.requestBids({
    timeout: 1500,
    bidsBackHandler: function() {
      pbjs.setTargetingForGPTAsync(['div-gpt-ad-123456789-0']);
      googletag.cmd.push(function() {
        googletag.pubads().refresh([slot]);
      });
    }
  });
});
</script>
```

`disableInitialLoad()` is required in this manual sequencing pattern: without it, GPT makes the ad server call as soon as `display()` executes, before Prebid has bids. It is not a universal GPT requirement — it only applies when manually controlling the first request. [Verified]

---

## Key-values and targeting

### Standard Prebid keys

| Key | Example value | Description |
|---|---|---|
| `hb_pb` | `2.40` | Winning CPM (price bucket) |
| `hb_bidder` | `appnexus` | Winning bidder |
| `hb_adid` | `abc123def` | Creative ID in cache |
| `hb_size` | `300x250` | Bid size |
| `hb_format` | `banner` | Media type |
| `hb_source` | `client` | `client` or `s2s` |
| `hb_cache_id` | `uuid` | Prebid Cache ID (for video/native) |

With `enableSendAllBids: true`: bidder suffix per key (`hb_pb_appnexus`, etc.). See anti-patterns.md for the key explosion risk with many bidders.

### Page-level targeting

```javascript
// Modern API (preferred for global targeting)
googletag.cmd.push(function() {
  googletag.setConfig({
    targeting: { content_category: 'sports', user_segment: ['seg1', 'seg2'] }
  });
  slot.setConfig({ targeting: { pos: 'right-rail' } });
});
```

`pubads().setTargeting()` is still valid for dynamic user-level targeting, but `googletag.setConfig({ targeting })` is the preferred modern API for page-level targeting.

---

## GAM Line Items checklist for Prebid

The mismatch between Prebid's `priceGranularity` and GAM price buckets is a frequent cause of silent revenue loss. [Verified]

- Price buckets aligned with Prebid's `priceGranularity`
- Key `hb_pb` used correctly in each line item's targeting
- Price ranges with no overlap between line items
- Correct creative macros (`%%PATTERN:hb_adid%%`)
- Catch-all line item ($0.01) for bids below the minimum bucket
- Correct priority (Price Priority for Prebid)

### Creative snippet (banner)

```html
<script>
  var w = window;
  for (var i = 0; i < 10; i++) {
    w = w.parent;
    if (w.pbjs) {
      try {
        w.pbjs.renderAd(document, '%%PATTERN:hb_adid%%');
        break;
      } catch(e) { continue; }
    }
  }
</script>
```

---

## Ad refresh

```javascript
function refreshAds(slots) {
  pbjs.requestBids({
    adUnitCodes: slots.map(function(s) { return s.getSlotElementId(); }),
    timeout: 1500,
    bidsBackHandler: function() {
      pbjs.setTargetingForGPTAsync();
      googletag.pubads().refresh(slots);
    }
  });
}
```

Refresh policy: minimum 30-second interval between refreshes (Google Ad Exchange). Declare refresh usage in the account configuration. Non-compliance can lead to penalties or Ad Exchange exclusion. [Verified]

---

## Lazy loading

```javascript
const observer = new IntersectionObserver(function(entries) {
  entries.forEach(function(entry) {
    if (entry.isIntersecting) {
      pbjs.requestBids({
        adUnitCodes: [entry.target.id],
        timeout: 1500,
        bidsBackHandler: function() {
          pbjs.setTargetingForGPTAsync([entry.target.id]);
          googletag.pubads().refresh([slotMap[entry.target.id]]);
        }
      });
      observer.unobserve(entry.target);
    }
  });
}, { rootMargin: '500px' }); // [Heuristic] — prefetch before viewport
```

Launching `requestBids` at the exact moment of viewport entry is too late. [Verified]

---

## Single Request Architecture (SRA)

SRA is appropriate when roadblocks or competitive exclusions are needed and all slots can be defined before the first `display()`.

With lazy loading: SRA is not incompatible, but requires all slots to be defined before the first `display()`. If slots are defined dynamically on viewport entry, SRA does not apply to those slots. [Verified]

---

## SPA — Slot management between navigations

```javascript
// On unmount:
googletag.cmd.push(function() {
  googletag.destroySlots([slot1, slot2]);
});
pbjs.removeAdUnit('div-slot-1');
pbjs.removeAdUnit('div-slot-2');

// In the new view, define new slots:
googletag.cmd.push(function() {
  slot1 = googletag.defineSlot('/path/to/unit', [300, 250], 'div-slot-1')
    .addService(googletag.pubads());
  googletag.display('div-slot-1');
});
pbjs.addAdUnits(newAdUnits);
pbjs.requestBids({ ... });
```

Do not reuse the same slot object after `destroySlots()`. [Verified]

---

## GPT events for debugging

```javascript
googletag.pubads().addEventListener('slotRequested', function(event) {
  console.log('GAM request sent:', event.slot.getSlotElementId());
});
googletag.pubads().addEventListener('slotResponseReceived', function(event) {
  console.log('GAM responded:', event.slot.getSlotElementId());
});
googletag.pubads().addEventListener('slotRenderEnded', function(event) {
  console.log('Render:', event.slot.getSlotElementId());
  console.log('isEmpty:', event.isEmpty);
  console.log('lineItemId:', event.lineItemId);  // verify if it is the Prebid line item
  console.log('advertiserId:', event.advertiserId);
});
```

`slotRenderEnded` with `lineItemId` allows verifying whether the winning line item is the Prebid one.

---

## Fallback and alternative demand

`googletag.definePassback()` is deprecated in modern GPT. For passback to external networks, use ad server fallback mechanisms or safeframes with custom logic. [Verified]
