# OpenRTB Video — Technical reference

## 1. imp.video in CTV bid request

Key fields of the `imp.video` object in OpenRTB 2.5/2.6:

| Field | Type | Description | Typical CTV value |
|---|---|---|---|
| `mimes` | array | Supported MIME types | `["video/mp4"]` |
| `minduration` | int | Minimum duration in seconds | 15 |
| `maxduration` | int | Maximum duration | 30 (or 60 for long-form) |
| `protocols` | array | Supported protocols | `[2,3,5,6]` (VAST 2.0-4.0) |
| `w` | int | Player width | 1920 |
| `h` | int | Player height | 1080 |
| `startdelay` | int | Position: 0=pre-roll, >0=mid-roll (offset in s), -1=generic mid, -2=generic post | 0 |
| `placement` | int | 1=in-stream, 2=in-banner, 3=in-article, 4=floating, 5=interstitial | 1 |
| `linearity` | int | 1=linear (in-stream), 2=non-linear (overlay) | 1 |
| `skip` | int | 0=not skippable, 1=skippable | 0 (CTV typically non-skip) |
| `pos` | int | Position in pod (if applicable) | 1 (1st position) |
| `api` | array | Supported APIs: 1=VPAID 1.0, 2=VPAID 2.0, 3=MRAID-1, 5=MRAID-2, 6=MRAID-3, 7=OMID-1 | `[7]` (OMID) |
| `companiontype` | array | Companion types: 1=static, 2=HTML, 3=iframe | Rarely used in CTV |

---

## 2. device object in CTV

| Field | Description | CTV notes |
|---|---|---|
| `devicetype` | 3=Connected TV (OpenRTB 2.5+), 7=Set-Top Box | Distinguish Smart TV vs STB |
| `make` | Manufacturer | "Samsung", "LG", "Amazon" |
| `model` | Model | "SmartTV", "BRAVIA", etc. |
| `os` | Operating system | "Tizen", "webOS", "Android", "tvOS", "Fire OS" |
| `osv` | OS version | "4.0", "5.0", "11.0" |
| `ifa` | Identifier for Advertisers (device ID) | Mapped from rdid |
| `lmt` | Limit Ad Tracking | 0=tracking OK, 1=opt-out |
| `ua` | User Agent | Device identification signal |
| `ip` | IP address | Geo-targeting, fraud detection |
| `connectiontype` | Connection type | 2=wifi, 7=cellular |

---

## 3. CTV PMP — deals in bid request

```json
{
  "pmp": {
    "private_auction": 1,
    "deals": [
      {
        "id": "PG_12345",
        "bidfloor": 12.0,
        "bidfloorcur": "EUR",
        "at": 3
      }
    ]
  }
}
```

**Auction types (`at`):**
- 1 = First Price Auction
- 2 = Second Price Auction
- 3 = Fixed Price (Programmatic Guaranteed)

---

## 4. Bid response for CTV

DSP responds with:

```json
{
  "seatbid": [{
    "bid": [{
      "id": "bid_001",
      "impid": "1",
      "price": 15.50,
      "adm": "<VAST version='4.0'>...</VAST>",
      "crid": "creative_12345",
      "dealid": "PG_12345",
      "w": 1920,
      "h": 1080,
      "cat": ["IAB1-6"],
      "dur": 30
    }]
  }]
}
```

**`adm`:** contains inline VAST XML (the creative). In some cases it is a URL to a VAST wrapper.

---

## 5. Content signals in OpenRTB

| OpenRTB field | Publisher source | Description |
|---|---|---|
| `site.content.id` | vid / Content ID | Content ID |
| `site.content.title` | MRSS / cust_params | Title |
| `site.content.series` | MRSS | Series name |
| `site.content.season` | MRSS | Season |
| `site.content.episode` | MRSS | Episode |
| `site.content.cat` | cc / IAB taxonomy | Content category |
| `site.content.contentrating` | MRSS / cust_params | Rating (TV-MA, etc.) |
| `site.content.language` | cust_params | Language |
| `site.content.len` | MRSS / duration | Duration in seconds |
| `site.content.livestream` | Content type | 0=VOD, 1=live |

**More signals → DSPs target better → bid higher → higher CPMs.**
