# Supply Chain — Deal health prerequisites

This file covers the foundations that must be in order **before** entering the deal funnel. If any of these prerequisites fail, diagnosing the deal itself is pointless.

---

## Check 0 — Privacy hygiene / TCF v2.2

The most invisible prerequisite. If the inventory does not emit valid consent signals, programmatic inventory is invisible to Tier-1 buyers — especially DV360 — regardless of how the deal is configured.

**What to verify:**
- The bid request includes `regs.ext.gdpr=1` and `user.ext.consent` with a valid, non-empty TC string
- The TC string includes consent for **Purpose 1** (storage and access to information on the device) — without it, most DSPs discard the bid request before evaluating the deal
- The CMP is firing correctly and the TC string reaches Prebid / the ad server before the auction launches

**Failure symptom:** the buyer does not bid on any deal from the publisher, or the bid rate drops drastically on European traffic (GDPR scope) with no configuration changes. The problem is cross-cutting — not specific to one deal.

**Where to diagnose in depth:** → `cmp-consent-flows`

**Practical rule:** if the symptom affects all deals and all programmatic traffic, check TCF before entering the deal funnel. If the symptom is specific to one deal or one buyer, TCF is probably not the cause.

---

## Check 1 — ads.txt / sellers.json / supply chain

A deal misconfigured in ads.txt or sellers.json produces no explicit error — the buyer simply does not bid or the SSP silently rejects bid responses. It is the most frequently ignored deal health prerequisite.

**Practical rule:** before diagnosing the deal object or the DSP, verify the supply chain. An incomplete ads.txt completely blocks deals with buyers that have strict validation (most tier-1 DSPs).

---

## ads.txt — Publisher Authorized Digital Sellers

### ads.txt line format

```
[SSP domain], [publisher account ID / seller ID], [relationship type], [TAGs Certification ID optional]
```

Example:
```
google.com, pub-1234567890123456, DIRECT, f08c47fec0942fa0
rubiconproject.com, 12345, DIRECT, 0bfd66d529a55807
triplelift.com, 10000-EB, DIRECT
appnexus.com, 9876, RESELLER
```

### Mandatory fields

| Field | Description | Valid values |
|---|---|---|
| SSP domain | Canonical SSP domain exactly as it appears in sellers.json | Verify in the SSP's sellers.json |
| Account ID | Publisher ID in the SSP — what the SSP uses to identify you | Get from SSP dashboard |
| Relationship type | `DIRECT` if the publisher has a direct agreement with the SSP; `RESELLER` if there is an intermediary | Almost always `DIRECT` for publisher |
| TAGs ID | Optional IAB certification ID — improves trust but not mandatory | Get from ads.txt TAGs lookup |

### Frequent ads.txt errors for deals

| Error | Symptom | Diagnosis |
|---|---|---|
| Incorrect account ID | Deal not delivering with buyers that validate ads.txt | Compare with the ID in the SSP dashboard |
| Misspelled SSP domain | Validators do not find the line | `rubiconproject.com` ≠ `rubicon.com` |
| Relationship as `RESELLER` when it should be `DIRECT` | Some buyers reject RESELLER inventory for direct deals | Verify with the SSP |
| Missing line for an SSP with active deal | Deal completely blocked with buyers with strict validation | Add correct line |
| ads.txt empty or inaccessible (404/500 error) | All buyers with strict validation reject the inventory | Verify accessibility from the publisher's URL |

### Verification tools
- [IAB ads.txt validator](https://iabtechlab.com/ads-txt/) — official verification
- [Google ads.txt crawler](https://adstxt.guru/) — third-party tool
- Directly: `https://[publisher-domain]/ads.txt`

---

## sellers.json — SSP Authorized Sellers

sellers.json is maintained by the **SSP**, not the publisher. The publisher cannot edit it directly — they must coordinate with the SSP if there are errors.

### Relevant structure

```json
{
  "sellers": [
    {
      "seller_id": "12345",
      "name": "Publisher Name",
      "domain": "publisher.com",
      "seller_type": "PUBLISHER",
      "is_confidential": 0
    }
  ]
}
```

### Critical fields for deals

| Field | Description | Impact on deals |
|---|---|---|
| `seller_id` | Publisher ID in the SSP — must match the account ID in ads.txt | Discrepancy blocks supply chain validation |
| `domain` | Canonical publisher domain | Buyers validate it matches the site in the bid request |
| `seller_type` | `PUBLISHER`, `INTERMEDIARY`, or `BOTH` | Must be `PUBLISHER` for a direct relationship |
| `is_confidential` | `1` = SSP does not publish publisher name/domain | Some buyers reject confidential sellers for deals |

### sellers.json diagnosis

1. Locate the SSP's sellers.json URL (normally `https://[ssp-domain]/sellers.json`)
2. Find the `seller_id` that appears in the publisher's ads.txt
3. Verify `domain` matches the publisher's domain
4. Verify `seller_type: PUBLISHER`
5. If `is_confidential: 1`, coordinate with the SSP to make it public — blocks deals with some buyers

**sellers.json URLs for common SSPs:**
- Google AdX: `https://storage.googleapis.com/adx-rtb-dictionaries/sellers.json`
- Magnite: `https://rubiconproject.com/sellers.json`
- Triplelift: `https://triplelift.com/sellers.json`

---

## app-ads.txt — For app inventory

Same format as ads.txt but hosted on the app developer's domain (not the app store). The buyer validates that the app's bundle ID corresponds to the developer declared in the store and that the developer has the correct ads.txt.

**Path:** `https://[developer-domain]/app-ads.txt`

**Additional requirement:** the developer domain must be declared in the app's listing on Google Play / App Store so buyers can find it.

**app-ads.txt specific diagnosis:**
1. Verify the bundle ID in the bid request matches the store app
2. Verify the store has the correct developer domain
3. Verify the developer domain has accessible app-ads.txt
4. Verify the SSP line in app-ads.txt has the correct account ID

---

## SupplyChain object (OpenRTB schain)

The SupplyChain object travels in the bid request so the buyer can verify the resale chain. For publishers with a direct relationship with the SSP, the chain must have exactly one node.

```json
"schain": {
  "complete": 1,
  "ver": "1.0",
  "nodes": [
    {
      "asi": "rubiconproject.com",
      "sid": "12345",
      "hp": 1
    }
  ]
}
```

| Field | Description |
|---|---|
| `complete` | `1` = the chain is complete from publisher to SSP |
| `asi` | SSP domain — must match ads.txt and sellers.json |
| `sid` | Seller ID in the SSP — must match ads.txt |
| `hp` | `1` = this node is paying the publisher |

**schain diagnosis for deals:**
- If `complete=0`, some buyers reject the bid request
- If `sid` does not match ads.txt, validation fails
- If there is more than one node for a direct publisher → SSP relationship, review the resale chain

---

## Supply chain checklist before diagnosing the deal object

- [ ] ads.txt accessible at `https://[publisher-domain]/ads.txt` (HTTP 200)
- [ ] SSP line present with correct domain, correct account ID, `DIRECT`
- [ ] SSP sellers.json includes the publisher with `seller_type: PUBLISHER`
- [ ] `seller_id` in sellers.json matches account ID in ads.txt
- [ ] `domain` in sellers.json matches the publisher's domain
- [ ] `is_confidential: 0` in sellers.json (or coordinate with SSP if `1`)
- [ ] For apps: app-ads.txt accessible, correct developer domain in the store
- [ ] schain in bid requests with `complete=1` and a single node for direct relationship
