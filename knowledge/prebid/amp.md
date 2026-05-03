# Prebid on AMP — Technical reference

Load only if mentioned: `AMP`, `<amp-ad>`, `amp-consent`, `Accelerated Mobile Pages`.

---

## Fundamental differences: AMP vs web

| Aspect | Standard web | AMP |
|---|---|---|
| Client-side Prebid | Yes | **No** — AMP does not execute arbitrary JS |
| Prebid on AMP | — | **Prebid Server mandatory** (S2S) |
| Consent management | `__tcfapi` / CMP JS | `<amp-consent>` |
| Ad slots | GPT / `googletag` | `<amp-ad>` with `type="doubleclick"` |
| Targeting | `setTargetingForGPTAsync` | RTC (Real-Time Config) |

**Practical consequence:** any performance or bid problem in AMP diagnosed with client-side Prebid logic is an incorrect diagnosis. The flow is completely different.

---

## Prebid architecture on AMP

```
AMP Runtime
└── <amp-ad type="doubleclick">
    └── RTC (Real-Time Config)
        └── Call to Prebid Server endpoint
            └── PBS runs the auction server-side
                └── Returns targeting params to amp-ad
                    └── amp-ad makes the request to GAM with those params
```

There is no `pbjs` in the browser. There is no `bidsBackHandler`. There is no `setTargetingForGPTAsync`.

---

## RTC configuration on AMP

```html
<amp-ad
  type="doubleclick"
  data-slot="/1234567/amp-banner"
  data-multi-size="300x250,320x50"
  rtc-config='{
    "vendors": {
      "prebidrubicon": {
        "REQUEST_ID": "1234",
        "ACCOUNT_ID": "5678"
      }
    },
    "timeoutMillis": 1000
  }'
>
</amp-ad>
```

`rtc-config` defines the RTC vendors (which point to PBS endpoints) and the timeout. The RTC timeout is independent of Prebid web's `bidderTimeout`.

---

## Consent on AMP

AMP uses `<amp-consent>` instead of a standard JavaScript CMP:

```html
<amp-consent id="consent" layout="nodisplay" type="YOUR_CMP_TYPE">
  <script type="application/json">
  {
    "consentInstanceId": "consent",
    "consentRequired": true,
    "checkConsentHref": "https://your-cmp.example.com/amp-consent",
    "promptUISrc": "https://your-cmp.example.com/amp-ui.html"
  }
  </script>
</amp-consent>
```

`<amp-ad>` automatically respects the `<amp-consent>` state. There is no `__tcfapi` available in AMP — consent is managed through the `<amp-consent>` system.

---

## Diagnosis on AMP vs web

### Symptoms that behave differently on AMP

| Symptom on web | On AMP may mean something else |
|---|---|
| No bids from an adapter | On AMP: the adapter may not be supported in PBS, or the RTC vendor is not configured |
| Low revenue | On AMP: lower match rate than web due to different cookie sync in PBS |
| Consent not reaching adapters | On AMP: verify `<amp-consent>` and the PBS-CMP integration, not `__tcfapi` |
| Bidder timeout | On AMP: the RTC timeout may be shorter than the `bidderTimeout` configured in PBS |

### Diagnostic tools on AMP

- **AMP Validator:** verify the AMP markup is valid
- **AMP Debug:** add `#development=1` to the URL to see logs in the console
- **Network tab:** RTC calls appear as requests from the AMP runtime to the PBS endpoint
- There is no `pbjs.getEvents()` — the equivalent is the RTC log in the AMP runtime

---

## AMP-specific anti-patterns

🔴 **Attempting to use client-side Prebid on AMP:** does not work. AMP does not execute arbitrary scripts.

🔴 **Applying web Prebid diagnostics to AMP problems:** the flow is completely different; the correct diagnosis starts from RTC and PBS, not from `pbjs`.

🟡 **RTC timeout not coordinated with PBS:** if the RTC timeout is lower than the PBS endpoint latency, all auctions expire before receiving bids.

🟡 **RTC vendor not configured in AMP's approved vendor list:** RTC vendors must be registered in the official AMP list to be usable.
