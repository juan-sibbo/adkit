# Ad Request Signals — Technical reference

## Table of contents
1. Complete VAST URL parameters
2. Device IDs by platform (verified idtypes)
3. PPID vs rdid
4. Session ID (sid) and is_lat=1 fallback
5. Consent signals (GDPR, COPPA)
6. Content signals and metadata
7. PPS and cust_params
8. Secure Signals
9. Buyer expectations in OpenRTB

---

## 1. Complete VAST URL parameters

| Parameter | Mandatory | Description |
|---|---|---|
| `iu` | ✅ | Ad unit path in GAM |
| `sz` | ✅ | Size (e.g. 640x480) |
| `env=vp` | ✅ | Environment: video player |
| `gdfp_req=1` | ✅ | GAM request |
| `output` | ✅ | Format: `xml_vast4`, `xml_vast3` |
| `correlator` | ✅ | Timestamp for cache-busting and competitive exclusion |
| `url` | Recommended | App URL or bundle |
| `description_url` | Recommended | Content URL (metadata) |
| `rdid` | Critical CTV | Resettable Device ID |
| `idtype` | Critical CTV | ID type (see table §2) |
| `is_lat` | Critical CTV | Limit Ad Tracking: 0=no, 1=yes |
| `msid` | Critical CTV | App store ID / bundle ID |
| `an` | Recommended | App name (human-readable) |
| `sid` | Recommended | Session ID (ephemeral) |
| `dth` | Recommended | Device type hint: 5=CTV |
| `plcmt` | Recommended | Placement: 1=in-stream |
| `vpa` | Recommended | Video playback: auto/click |
| `vpmute` | Recommended | Muted: 0=unmuted, 1=muted |
| `cust_params` | Recommended | Custom parameters URL-encoded |
| `cc` | Recommended | Content category (IAB taxonomy) |
| `vid` | Recommended | Video content ID (if using MRSS/Content Sources) |
| `cmsid` | Recommended | CMS ID (if using MRSS) |
| `ad_rule` | Conditional | 1 if using Ad Rules |
| `gdpr` | Conditional | 0/1 if audience in EEA |
| `gdpr_consent` | Conditional | Full TC String |
| `ppid` | Optional | Publisher-provided ID (hash of login) |
| `givn` | Conditional | PAL nonce (if using PAL SDK) |

---

## 2. Device IDs by platform

| Platform | ID name | idtype in VAST | API to get ID | Evidence level |
|---|---|---|---|---|
| Samsung Tizen | TIFA | `tifa` | `webapis.adinfo.getTIFA()` | Probable (source material indicates `ssai` but inconsistent with Samsung docs) |
| LG webOS | LGUDID / LGADI | `lgudid` | Proprietary LG API | Probable |
| Android TV | GAID | `adid` | `AdvertisingIdClient.getAdvertisingIdInfo()` | Verified |
| Fire TV | AFAI | `afai` | `AdRegistration.getAdvertisingId()` | Probable |
| Apple TV | IDFA | `idfa` | `ASIdentifierManager.shared().advertisingIdentifier` | Verified |
| Roku | RIDA | `rida` | Roku API | Verified |

**Format:** all UUID v4 (36 characters with hyphens).

**⚠️ Note on Samsung:** the value `ssai` as idtype appears frequently in industry material but is not confirmed in official Google or Samsung documentation. Using `tifa` is more consistent with Samsung's own nomenclature. Verify with the SSAI provider or GAM support if in doubt.

---

## 3. PPID vs rdid

| Field | rdid | ppid |
|---|---|---|
| **What it is** | Advertising device identifier (hardware/OS) | Publisher-provided ID (login hash, account ID) |
| **Who generates it** | Device OS | Publisher |
| **Resettable** | Yes (by the user in settings) | No (controlled by publisher) |
| **Scope** | Device-level | User-level (cross-device if there is a login) |
| **Use** | Frequency capping, attribution, audience segments | Cross-session identity, first-party segments |
| **is_lat** | Yes (if user opts out, do not use for targeting) | N/A (it is the publisher's ID) |

**Anti-pattern:** passing rdid as the value of ppid → semantically different fields [Verified]. Keep them separate.

---

## 4. Session ID (sid) and is_lat=1 fallback

**Session ID:** ephemeral UUID generated at app session start. Regenerated each time the app closes and reopens.

**When it is critical:** when `is_lat=1` (user opted out of tracking):
- `rdid` present but MUST NOT be used for targeting
- No cross-session frequency capping
- **Fallback:** GAM uses `sid` for frequency cap within the active session
- Limitation: if user closes app and reopens → new sid → cap resets

**Household problem:**
- 1 TV = 1 device ID = multiple users in the household
- Device ID is NOT user-level, it is device-level
- No native cross-device tracking in CTV
- Mitigation: viewing patterns as proxy, or login-based identification (ppid)

---

## 5. Consent signals (GDPR, COPPA)

**GDPR:**
- `gdpr=1` if user is in EEA
- `gdpr_consent=<TC_STRING>` — full TC String (TCF v2.2)
- TC String: starts with CO/CP, ~100-200 chars, Base64 URL-safe
- Decode: https://iabtcf.com/#/decode
- If TC String is invalid or absent → DSPs do not bid or only non-personalized ads → CPM drops significantly

**COPPA:**
- `coppa=1` if content is directed at children under 13
- Restrictions: no behavioral targeting, no frequency cap with device ID, no third-party ads

---

## 6. Content signals and metadata

**Via Content ID (MRSS):** `vid=CONTENT_ID&cmsid=CMS_ID`
- GAM enriches with pre-registered metadata

**Via cust_params:** `cust_params=genre%3Ddrama%26rating%3Dtv_ma`
- Limit: ~2000 chars in total query string
- Encoding: `=` → `%3D`, `&` → `%26`
- Anti-pattern: double encoding (`%253D` instead of `%3D`)

**Via content category:** `cc=IAB1-6` (IAB taxonomy)
- Multiple: `cc=IAB1-6,IAB17`

---

## 7. PPS and cust_params

**cust_params** are visible in query strings (logs, proxies). For sensitive data → Secure Signals.

**cust_params best practices:**
- Normalize values: lowercase, no spaces, underscores
- 10-15 key-values maximum
- Cache calculations per session (daypart, etc.)
- Verify encoding with a URL decode tool

---

## 8. Secure Signals

**What they are:** encrypted/obfuscated signals that are NOT visible in URLs. Only revealed in the bid stream to authorized buyers.

**Configuration in GAM:** Admin > Secure Signals
- Define signals
- Assign authorized buyers
- Configure consent requirements

**Implementation:** there is NO public `setSecureSignals()` method in the IMA SDK as a standard documented method. Integration happens:
- Via signal providers (third-party adapters integrated in the SDK)
- Via server-side configuration in GAM
- The exact implementation details depend on the GAM and SDK version

**Testing:** signals are opaque (encrypted). Validate via:
1. GAM admin: verify active signal definition
2. Test deal with buyer: buyer confirms visibility in bid request
3. Indirect reporting: create line item with signal targeting

---

## 9. Buyer expectations in OpenRTB

What DSPs expect to see in a CTV bid request:

| OpenRTB field | Mapping from VAST tag | Importance |
|---|---|---|
| `device.ifa` | rdid | Critical: frequency cap, attribution |
| `device.lmt` | is_lat | Critical: privacy compliance |
| `device.ua` | User Agent | High: device identification, fraud detection |
| `device.ip` | Device IP | High: geo-targeting, fraud detection |
| `device.geo` | Derived from IP | High: geographic targeting |
| `site.content.cat` | cc (IAB taxonomy) | High: categorical targeting |
| `site.content.language` | From content | Medium: language targeting |
| `imp.video.w/h` | sz | Medium: creative selection |
| `imp.video.startdelay` | Position (pre/mid/post) | Medium: positioning |
| `user.consent` | gdpr_consent | Critical: GDPR compliance |
| `app.bundle` | msid | High: app verification |

**Missing signals → lower CPM:** DSPs bid less (or not at all) if key signals such as device ID, consent, or app bundle are absent.
