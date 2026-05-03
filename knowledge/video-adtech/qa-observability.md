# QA and Observability — Technical reference

## 1. Test matrix prioritized by European market share

**Testing prioritization (Europe):**

| Priority | Platforms | % effort | Approx. market share |
|---|---|---|---|
| High | Samsung Tizen (latest + legacy 2.4), LG webOS (latest) | 50% | ~50% |
| Medium | Android TV, Fire TV | 30% | ~25% |
| Low | Apple TV, Roku, consoles | 20% | ~15% |

**Test matrix dimensions:**

| Dimension | Values to test |
|---|---|
| OS/device | Tizen 5.5+, Tizen 2.4 (legacy), webOS 5.0+, Android TV 10+, Fire TV Gen 3 |
| Resolution | 1080p, 720p (legacy), 4K (if applicable) |
| Network | WiFi fiber (50+ Mbps), standard WiFi (20 Mbps), mobile hotspot (5 Mbps), throttled 3G (simulated) |
| Content | VOD, Live, FAST |
| Position | Pre-roll, mid-roll, post-roll |

**Minimum viable:** 3 OS × 2 resolutions × 2 networks × 2 content types = 24 combinations.

---

## 2. Pre-launch checklist by block

### GAM configuration
- [ ] App registered and claimed in GAM
- [ ] app-ads.txt published and validated
- [ ] Ad units created with consistent naming
- [ ] Key-values defined
- [ ] Line items configured (direct + house ads)
- [ ] Creatives uploaded and approved (multiple resolutions)
- [ ] Ad Rules configured (correct timing)
- [ ] Frequency caps set
- [ ] Unified Pricing Rules (floors by geo/content)
- [ ] Open Bidding yield groups active
- [ ] Third-party verification configured (OMID vendors)

### Technical integration
- [ ] IMA SDK (or PAL/PAI) correctly integrated
- [ ] Device ID obtained and passed (rdid, idtype, is_lat)
- [ ] cust_params built dynamically and URL-encoded
- [ ] GDPR consent string passed (gdpr, gdpr_consent)
- [ ] Content metadata included (vid or cust_params)
- [ ] Session ID implemented (is_lat=1 fallback)
- [ ] Robust error handling (timeouts, no-fill, VAST errors)
- [ ] Fallback strategy implemented (ad → house → content)
- [ ] Functional event tracking (impression, quartiles, complete)

### Metadata and content
- [ ] MRSS feed configured and GAM syncing (if applicable)
- [ ] Content bundles designed
- [ ] IAB taxonomy mapped
- [ ] Content ratings assigned
- [ ] Cue points defined

### Testing
- [ ] Test matrix executed (minimum 5 devices)
- [ ] Slow network tested (timeout works)
- [ ] No-fill tested (app does not freeze)
- [ ] VAST error tested (403, 404, timeout)
- [ ] Device ID validated (format, type, LAT handling)
- [ ] Consent string validated

### Performance
- [ ] Ad request latency <5s (median)
- [ ] Timeout handling functional
- [ ] No memory leaks after 10+ ad breaks
- [ ] Reasonable CPU during ad playback

---

## 3. Debugging tools

| Tool | Purpose | Platforms |
|---|---|---|
| **GAM Publisher Console** | See eligible line items, no-match reasons | All (web-based) |
| **Charles Proxy / Proxyman** | Intercept HTTP/HTTPS traffic from device | All (requires proxy on same WiFi) |
| **Google Video Suite Inspector** | Validate VAST XML | All (web-based) |
| **adb logcat** | SDK logs on Android TV/Fire TV | Android TV, Fire TV |
| **Xcode Console** | SDK logs on tvOS | Apple TV |
| **Tizen Web Inspector** | JavaScript console on Samsung | Samsung Tizen |
| **Network Link Conditioner** | Simulate slow network | Mac (proxy to device) |
| **curl** | Manual fetch of VAST URL and MediaFile URL | Manual diagnosis |

---

## 4. Internal SLAs (between AdOps and Dev/CMS)

| SLA | Description | Target |
|---|---|---|
| Data freshness | MRSS metadata update frequency | Every 6-12h |
| Device ID availability | % of ad requests with valid rdid | 100% (when is_lat=0) |
| CMS timeout | Maximum time the app waits for metadata before ad request without it | 2s |
| PPS audit | Verify that sent cust_params match CMS | Monthly |

---

## 5. Reference KPIs

| KPI | What to measure | Alert if |
|---|---|---|
| Fill rate | Served impressions / ad requests | <40% (general) or >10% drop in 24h |
| Error rate | VAST errors / total ad requests | >5% |
| Latency (p50) | Ad request → first frame of ad | >3s (CTV), >2s (live) |
| VCR | Completions / starts | <80% in CTV |
| Discrepancy | GAM vs SSP/DSP | >12% |
| House ad rate | House ads / total served impressions | >5% (indicates lack of demand) |
