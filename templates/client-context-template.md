# CLIENT CONTEXT — [NOMBRE_CLIENTE]
> Archivo de contexto por cliente · Cargado al inicio de sesión via `client-context-loader`
> Versión: 1.0.0 · Última actualización: YYYY-MM-DD · Responsable: [nombre]

---

## §0. Metadata operativa

| Campo | Valor |
|-------|-------|
| **Cliente** | [nombre] |
| **Tipo** | TV autonómica / broadcaster / publisher web / FAST channel / otro |
| **Network Code GAM** | [código numérico] |
| **Intermediario principal** | Newixmedia / [otro] |
| **Contacto técnico cliente** | [nombre · email · handle Slack] |
| **Contacto comercial Sibbo** | [nombre] |
| **Fecha de onboarding** | YYYY-MM-DD |
| **Estado** | Activo / En onboarding / Piloto / Suspendido |
| **Tier de soporte** | Premium / Estándar |

---

## §1. Stack de entrega de vídeo

### 1.1 Modelo de entrega

- **CSAI (client-side):** Sí / No
- **SSAI / DAI:** Sí / No → proveedor: `Google DAI / Mediakind / AWS Elemental / Harmonic / otro`
- **Modo híbrido:** [descripción si CSAI y SSAI coexisten, e.g., "CSAI en web, SSAI en CTV"]
- **Player principal web:** `Video.js vX.X / Shaka Player vX.X / HLS.js / custom`
- **Player principal CTV:** `ExoPlayer vX.X / AVPlayer / Shaka / custom`
- **Versión VAST máxima soportada:** `2.0 / 3.0 / 4.0 / 4.1 / 4.2`
- **VMAP:** Sí / No
- **Ad pods (VAST 4.x):** Sí / No → configurados en: [GAM ad rules / SSAI / ambos]

### 1.2 IMA SDK — implementación por entorno

| Entorno | SDK | Versión | Integración player | Estado |
|---------|-----|---------|--------------------|--------|
| Web / HLS | IMA HTML5 SDK | vX.X.X | Video.js contrib-ima / Shaka / manual | ✅ Producción |
| Android TV | IMA Android SDK | vX.X.X | ExoPlayer IMA extension vX.X | ✅ Producción |
| Fire TV | IMA Android SDK | vX.X.X | ExoPlayer IMA extension vX.X | ✅ Producción |
| Samsung Tizen | IMA HTML5 SDK | vX.X.X | Custom JS sobre Tizen browser engine | ⚠️ En pruebas |
| LG webOS | IMA HTML5 SDK | vX.X.X | Custom JS sobre webOS browser engine | ✅ Producción |
| tvOS / Apple TV | IMA tvOS SDK | vX.X.X | AVPlayer + IMA Swift/ObjC integration | ❌ No implementado |
| HbbTV | IMA HTML5 SDK | vX.X.X | HbbTV application player | ⚠️ En pruebas |
| Chromecast (receptor) | IMA Cast SDK | vX.X.X | CAF Web Receiver | ❌ No implementado |
| Roku | [no IMA SDK oficial] | — | Direct VAST / SceneGraph | ❌ No implementado |

> **Nota de mantenimiento:** Verificar versiones de IMA SDK contra [releases oficiales](https://developers.google.com/interactive-media-ads/docs/sdks/html5/client-side/history) en cada revisión trimestral. Cambios de MAJOR version requieren regresión.

---

## §2. Dispositivos CTV — particularidades técnicas

> Esta sección documenta las peculiaridades de cada entorno. Es la fuente de verdad para `video-adtech` y `discrepancy-forensics` cuando el comportamiento difiere del estándar IMA.

### 2.1 Android TV

| Campo | Detalle |
|-------|---------|
| **OS version objetivo** | Android TV X (API level YY) |
| **RDID disponible** | TIFA (`Settings.Secure.ANDROID_ID` + opt-out flag) |
| **Disponibilidad TIFA** | Alta (opt-in por defecto en Android TV OS) |
| **Parámetro ad tag** | `rdid=%%TIFA%%&idtype=tifa&is_lat=%%OPT_OUT%%` |
| **Señal de consentimiento** | GPP string + TCF string (si CMP disponible en app) |
| **IMA SDK** | Android IMA SDK vX.X + ExoPlayer IMA extension vX.X |
| **Companion ads** | Soportados / No soportados en este player |
| **Modo DAI (SSAI)** | Implementado / No implementado |

**Particularidades conocidas:**
- [ ] TIFA puede aparecer como todo ceros en apps instaladas via sideload — usar PPID como señal primaria en ese caso
- [ ] ExoPlayer IMA extension requiere que el `ImaAdsLoader` se libere en `onStop()` o hay leak de recursos
- [ ] [Añadir particularidad específica del cliente]

**Anti-patterns documentados:** → knowledge-capture refs: [KC-XXXX-XX-XX]

---

### 2.2 Samsung Tizen (Smart TV)

| Campo | Detalle |
|-------|---------|
| **Tizen OS version objetivo** | X.X (e.g., Tizen 6.5 = modelos 2022+) |
| **RDID disponible** | RIDA (`b.samsungads.com` API) |
| **Disponibilidad RIDA** | Media (requiere llamada a API Samsung Ads, sujeto a opt-out del usuario) |
| **Parámetro ad tag** | `rdid=%%RIDA%%&idtype=rida&is_lat=%%RIDA_OPT_OUT%%` |
| **Señal de consentimiento** | TCF string (limitada — no hay CMP nativa en Tizen OS) |
| **IMA SDK** | HTML5 SDK vX.X sobre Tizen browser engine (Blink/Chromium embebido) |
| **Motor JS** | V8 (Tizen 5.5+) / JavaScriptCore (Tizen <5) |

**Particularidades conocidas:**
- [ ] ES6 modules no soportados en Tizen <5 — transpilación a ES5 requerida (Babel)
- [ ] `localStorage` no persiste entre sesiones en Tizen 4.x — usar `webapis.storage` de Samsung
- [ ] IMA SDK HTML5 requiere `googletag` cargado antes de inicializar — orden de carga crítico
- [ ] CORS en Tizen browser más restrictivo que en desktop Chrome — verificar headers del ad server
- [ ] [Añadir particularidad específica del cliente]

**Anti-patterns documentados:** → knowledge-capture refs: [KC-XXXX-XX-XX]

---

### 2.3 LG webOS (Smart TV)

| Campo | Detalle |
|-------|---------|
| **webOS version objetivo** | X.X (e.g., webOS 6.0 = LG 2021+) |
| **RDID disponible** | LGUDID (via `webOS.service.request('luna://com.webos.service.idgeneration')`) |
| **Disponibilidad LGUDID** | Media-alta (requiere `luna://com.webos.service.idgeneration` permission declarada en `appinfo.json`) |
| **Parámetro ad tag** | `rdid=%%LGUDID%%&idtype=lgudid&is_lat=%%LG_OPT_OUT%%` |
| **Señal de consentimiento** | TCF string (disponibilidad varía) |
| **IMA SDK** | HTML5 SDK vX.X sobre webOS Chromium |

**Particularidades conocidas:**
- [ ] `fetch()` no disponible en webOS <3.x — usar `XMLHttpRequest`
- [ ] LGUDID requiere que la app esté registrada en LG Developer Program con permission `ID_GENERATION`
- [ ] webOS 3.x y anteriores: performance limitada con IMA HTML5 SDK completo — considerar lightweight ad client
- [ ] Memory heap reducido vs desktop — evitar cargar múltiples SDKs simultáneamente
- [ ] [Añadir particularidad específica del cliente]

**Anti-patterns documentados:** → knowledge-capture refs: [KC-XXXX-XX-XX]

---

### 2.4 tvOS / Apple TV

| Campo | Detalle |
|-------|---------|
| **tvOS version objetivo** | X.X |
| **RDID disponible** | IDFA (sujeto a ATT prompt — opt-in requerido desde tvOS 14.5) |
| **Disponibilidad IDFA** | Muy baja en práctica (<20% opt-in típico en CTV) — PPID es la señal principal |
| **Señal alternativa primaria** | PPID (crítico — única señal de identidad viable en tvOS) |
| **Parámetro ad tag** | `rdid=%%IDFA%%&idtype=idfa&is_lat=%%ATT_DENIED%%&ppid=%%PPID%%` |
| **Framework ATT** | `AppTrackingTransparency` — implementado Sí/No |
| **Señal de consentimiento** | ATT consent (IDFA) · TCF string si app incluye CMP |
| **IMA SDK** | IMA tvOS SDK vX.X (Swift / Objective-C) + AVPlayer |

**Particularidades conocidas:**
- [ ] IMA tvOS SDK no soporta companion ads — no esperarlos en ad tags CTV de Apple TV
- [ ] `SKAdNetwork` como mecanismo de medición cuando IDFA no disponible (relevante para paid UA, no para monetización directa)
- [ ] ATT prompt debe mostrarse antes de cualquier solicitud de ad — flujo de onboarding de app crítico
- [ ] PPID debe configurarse como señal obligatoria en GAM antes de activar inventario tvOS
- [ ] [Añadir particularidad específica del cliente]

**Anti-patterns documentados:** → knowledge-capture refs: [KC-XXXX-XX-XX]

---

### 2.5 HbbTV

| Campo | Detalle |
|-------|---------|
| **Versión HbbTV objetivo** | 1.4.7 / 2.0.2 / 2.0.3 (especificar por operador/red) |
| **Operadores/redes activos** | [Movistar+ / Orange / Vodafone / TDT / otro] |
| **RDID disponible** | No (por diseño HbbTV) — señal de identidad alternativa: PPID / IP-based |
| **Señal de consentimiento** | TCF string (disponibilidad varía por operador — Movistar+ sí, TDT no) |
| **IMA SDK** | HTML5 SDK vX.X en modo sin RDID |
| **Motor JS del terminal** | WebKit / Presto (terminales legacy) — rendimiento limitado |

**Particularidades conocidas:**
- [ ] Heap de memoria muy limitado en terminales HbbTV 1.4 legacy — IMA HTML5 SDK puede ser demasiado pesado; considerar `AdsRequest` minimalista
- [ ] Autoplay de vídeo restringido en algunos operadores IPTV — requiere interacción de usuario previa al inicio del stream
- [ ] HbbTV 1.4 no soporta `Promises` ni `async/await` — código síncrono con callbacks
- [ ] `XMLHttpRequest` síncrono bloqueante es más estable que fetch() en terminales legacy
- [ ] En TDT (broadcast HbbTV), la señal retorno no está garantizada — diseñar para modo offline/degradado
- [ ] [Añadir particularidad específica del cliente]

**Anti-patterns documentados:** → knowledge-capture refs: [KC-XXXX-XX-XX]

---

## §3. Integración GAM 360 — señales

> Sección de mayor densidad técnica. Documentar con precisión quirúrgica — es la fuente de verdad para troubleshooting de fill, discrepancias y auditorías de señal.

### 3.1 Estructura de inventario

| Campo | Valor |
|-------|-------|
| **Network Code** | [código] |
| **Ad units CTV pre-roll** | `/[netcode]/[cliente]/ctv/preroll` |
| **Ad units CTV mid-roll** | `/[netcode]/[cliente]/ctv/midroll` |
| **Ad units Web pre-roll** | `/[netcode]/[cliente]/web/preroll` |
| **Ad units display** | `/[netcode]/[cliente]/display/[formato]` |
| **Key-values activos** | `pos`, `env`, `rdid`, `idtype`, `is_lat`, `ppid`, `ctype`, `genre`, `series` |
| **Ad rules CTV** | [descripción: e.g., "pre-roll + 1 mid-roll cada 15 min en live"] |

**Template base de ad tag CTV (VAST URL):**
```
https://pubads.g.doubleclick.net/gampad/ads
  ?iu=/[netcode]/[cliente]/ctv/preroll
  &sz=640x480
  &gdfp_req=1
  &output=vast
  &unviewed_position_start=1
  &env=vp
  &impl=s
  &correlator=%%CACHEBUSTER%%
  &rdid=%%RDID%%
  &idtype=%%IDTYPE%%
  &is_lat=%%IS_LAT%%
  &ppid=%%PPID%%
  &cust_params=env%%3Dctv%%26ctype%%3D%%CONTENT_TYPE%%
  &gdpr=%%GDPR%%
  &gdpr_consent=%%GDPR_CONSENT%%
```

---

### 3.2 Señales compartidas con GAM (signal sharing)

#### 3.2.1 PPID (Publisher Provided ID)

| Campo | Detalle |
|-------|---------|
| **Estado** | Implementado / En pruebas / No implementado |
| **Identity partner(s)** | LiveRamp ATS / ID5 / propio hashed email / otro |
| **Entornos donde se pasa** | Web / Android TV / tvOS / HbbTV / todos |
| **Método de paso al ad tag** | Parámetro `ppid=` en URL · macro GAM `%%PPID%%` |
| **Hashing aplicado** | SHA-256 del email / MD5 / sin hash (raw) |
| **Criticidad** | Alta — es la única señal de identidad viable en tvOS y HbbTV |

**Notas:**
- [ ] [e.g., "PPID no implementado aún en HbbTV — fill programático muy bajo por falta de señal"]
- [ ] [e.g., "PPID en web requiere usuario logado — cobertura ~35% de sesiones"]

---

#### 3.2.2 Secure Signals (Encrypted Signals para Open Bidding)

| Proveedor | Señal encriptada | Entornos activos | Estado | Notas |
|-----------|-----------------|------------------|--------|-------|
| LiveRamp ATS | ATS envelope | Web | ✅ Activo | Solo logged users |
| ID5 | ID5 ID (versión encriptada) | Web / Android TV | ⚠️ En pruebas | |
| [otro] | [señal] | [entornos] | ❌ No activo | |

**Configuración en GAM:**
- Ruta: Admin → Global settings → Publisher provided signals → Encrypted signals
- [ ] Verificar que el script del proveedor se carga antes del GPT tag
- [ ] Verificar que `googletag.encryptedSignalProviders` está configurado en GPT

---

#### 3.2.3 PPS — Publisher Provided Signals (Audience & Content Signals)

| Campo | Detalle |
|-------|---------|
| **Estado** | Implementado / En pruebas / No implementado |
| **Content taxonomy** | IAB Content Taxonomy v2.0 / v3.0 |
| **Audience taxonomy** | IAB Audience Taxonomy v1.1 |
| **Entornos activos** | Web / CTV / ambos |

**Señales de contenido configuradas:**

| Señal | Campo OpenRTB | Activa | Fuente del dato |
|-------|--------------|--------|----------------|
| Género | `content.genre` | Sí / No | CMS del cliente |
| Keywords | `content.keywords` | Sí / No | CMS del cliente |
| Serie | `content.series` | Sí / No | CMS del cliente |
| Episodio | `content.episode` | Sí / No | CMS del cliente |
| Es live | `content.livestream` | Sí / No | Player event |
| Duración | `content.len` | Sí / No | Player metadata |
| Rating de contenido | `content.contentrating` | Sí / No | CMS del cliente |
| Categoría IAB | `content.cat` | Sí / No | CMS del cliente |

**Notas:**
- [ ] [e.g., "PPS activo solo en web — CTV no tiene integración de content taxonomy aún"]
- [ ] [e.g., "Keywords de CMS sin normalización a taxonomía IAB — pendiente mapeo"]

---

#### 3.2.4 RDIDs en ad tags CTV — tabla de estado por dispositivo

| Dispositivo | RDID type | Parámetro ad tag | Opt-out param | Estado | Fallback activo |
|-------------|-----------|-----------------|---------------|--------|----------------|
| Android TV | `tifa` | `rdid=%%TIFA%%&idtype=tifa` | `is_lat=%%OPT_OUT%%` | ✅ Activo | PPID |
| Fire TV | `afai` | `rdid=%%AFAI%%&idtype=afai` | `is_lat=%%OPT_OUT%%` | ✅ Activo | PPID |
| Samsung Tizen | `rida` | `rdid=%%RIDA%%&idtype=rida` | `is_lat=%%RIDA_OPT_OUT%%` | ✅ Activo | PPID |
| LG webOS | `lgudid` | `rdid=%%LGUDID%%&idtype=lgudid` | `is_lat=%%LG_OPT_OUT%%` | ⚠️ En pruebas | PPID |
| tvOS | `idfa` | `rdid=%%IDFA%%&idtype=idfa` | `is_lat=%%ATT_DENIED%%` | ⚠️ Bajo opt-in | PPID (crítico) |
| HbbTV | N/A | *(sin RDID)* | — | — | PPID / IP |
| Chromecast | `tifa` (Android) | `rdid=%%TIFA%%&idtype=tifa` | `is_lat=%%OPT_OUT%%` | ❌ No impl. | — |

---

#### 3.2.5 Señal de consentimiento (TCF / GPP) por entorno

| Entorno | CMP activa | TC string en ad tag | GPP string | Notas |
|---------|-----------|--------------------| -----------|-------|
| Web | Didomi vX.X | Sí — macro `%%GDPR_CONSENT%%` | En pruebas | |
| Android TV app | [CMP en app / sin CMP] | Sí / No | No | |
| Samsung Tizen app | Sin CMP nativa | No | No | TCF limitado |
| LG webOS app | Sin CMP nativa | No | No | |
| tvOS app | Sin CMP (ATT en su lugar) | No | No | ATT es el consent |
| HbbTV | [Movistar+ sí / TDT no] | Sí / No | No | Varía por operador |

---

### 3.3 Open Bidding y Yield Management

| Campo | Detalle |
|-------|---------|
| **SSPs en Open Bidding** | [lista: PubMatic, Magnite, Xandr...] |
| **Yield groups activos** | [descripción: e.g., "YG_CTV_PREROLL con floor €3.00"] |
| **Pricing rules relevantes** | [e.g., "Floor dinámico activado — mín €1.50, máx €8.00"] |
| **Competitive exclusions** | [e.g., "Telecos competidoras excluidas del inventario CTV"] |

---

## §4. SSPs y deals activos

| SSP | Tipo integración | Tipo deal | Deal ID | Floor (CPM) | Buyer | Estado | Notas |
|-----|-----------------|-----------|---------|-------------|-------|--------|-------|
| PubMatic | Open Bidding + Prebid | PMP | [ID] | €X.XX | [Buyer/DSP] | ✅ Activo | |
| Magnite | Open Bidding | PG | [ID] | €X.XX | [Buyer/DSP] | ✅ Activo | |
| Xandr | Prebid header bidding | Open auction | — | €X.XX floor | — | ✅ Activo | |
| FreeWheel | VAST directo (CTV) | Directo | — | — | — | ✅ Activo CTV | |
| SpotX/Magnite CTV | SSAI / OB | PMP CTV | [ID] | €X.XX | [Buyer] | ⚠️ En setup | |

---

## §5. ads.txt / app-ads.txt / sellers.json

| Fichero | URL | Última validación | Estado | Incidencias conocidas |
|---------|-----|------------------|--------|----------------------|
| ads.txt | `https://[dominio]/ads.txt` | YYYY-MM-DD | ✅ OK / ⚠️ Problemas | |
| app-ads.txt (Android) | `https://[dominio]/app-ads.txt` | YYYY-MM-DD | ✅ OK | |
| app-ads.txt (tvOS) | `https://[dominio]/app-ads.txt` | YYYY-MM-DD | ❌ No configurado | |
| sellers.json (Newixmedia) | `https://newixmedia.com/sellers.json` | YYYY-MM-DD | ✅ OK | |

**Bundle IDs de apps:**

| Plataforma | Bundle ID | Estado en app-ads.txt |
|------------|-----------|----------------------|
| Android / Fire TV | `[com.cliente.app]` | ✅ Declarado |
| tvOS | `[com.cliente.app.tv]` | ❌ Pendiente |
| Samsung Tizen | `[XXXXXXXXXXXXXXXX]` | ⚠️ En proceso |

---

## §6. Incidencias abiertas y anti-patterns del cliente

| ID | Fecha | Descripción | Estado | Skill afectado | Ref. KC |
|----|-------|-------------|--------|---------------|---------|
| INC-001 | YYYY-MM-DD | [descripción breve] | Abierta / Resuelta | video-adtech | KC-XXXX |
| INC-002 | YYYY-MM-DD | [descripción breve] | Abierta | programmatic-deals | KC-XXXX |

---

## §7. Changelog del contexto

| Versión | Fecha | Cambio | Autor |
|---------|-------|--------|-------|
| 1.0.0 | YYYY-MM-DD | Creación inicial en onboarding | [nombre] |
| 1.1.0 | YYYY-MM-DD | [e.g., Añadido tvOS — INC-002 resuelta] | [nombre] |
| 1.1.1 | YYYY-MM-DD | [e.g., Actualizado RDID LG webOS a producción] | [nombre] |
