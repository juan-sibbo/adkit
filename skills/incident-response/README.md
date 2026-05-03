---
name: incident-response
description: Structured response flow for production incidents affecting ad serving. Activate for "incident", "outage", "ads not serving", "fill at zero", "mass black screen", "VAST error in production", "urgent rollback", "client reports no advertising", "sudden fill drop", or any high-pressure situation requiring fast action without making mistakes. Provides a decision tree, communication templates, and post-mortem capture.
---

# Incident Response

**Version:** 1.0.0
**Status:** Stable
**Last updated:** 2026-04-27
**Depends on:** gam-adops, video-adtech, programmatic-deals (for technical diagnosis)
**Used by:** Standalone — activated in high-pressure situations

> This skill does NOT perform technical diagnosis — that is handled by the specialised skills.
> This skill manages the response flow: triage, communication, escalation, and post-mortem.
> Quality under pressure depends on having the process loaded before you need it.

---

## Step 0 — Immediate triage (first 5 minutes)

As soon as an incident is detected, answer these 4 questions before doing anything else:

```
1. WHAT is failing?
   □ Fill at zero or abnormally low fill
   □ VAST error / black screen / ad not playing
   □ Revenue drop without fill drop
   □ Massive discrepancy between systems
   □ Other: [describe]

2. WHERE does it affect?
   □ A specific client: [which one]
   □ All clients in the portfolio
   □ A specific device/environment: [web / Android TV / Samsung / LG / tvOS / HbbTV]
   □ A specific SSP: [which one]
   □ A time window: [when it started, exact time]

3. HOW BIG is the blast radius?
   □ % of affected inventory (estimated)
   □ Revenue at risk per hour (estimated)
   □ Number of affected users/viewers (if estimable)

4. WHEN did it start?
   □ Exact start time (or approximate)
   □ Does it coincide with any deploy, configuration change, or external event?
```

With these 4 answers, activate the decision tree in Step 1.

---

## Step 1 — Decision tree: incident type

### Branch A — Fill at zero or abnormally low fill

```
Does it affect a single client or all clients?
├── Single client:
│   → Check client's ads.txt/sellers.json (supply chain)
│   → Check for recent changes in GAM (pricing rule, competitive exclusion)
│   → Activate discrepancy-forensics for rapid analysis
│   → Severity: MEDIUM if <30 min, HIGH if >30 min
│
└── All clients:
    → Check GAM status (https://www.google.com/appsstatus — Ad Manager)
    → Check primary SSP status (provider status page)
    → If external issue (GAM/SSP) → communicate and wait, no publisher-side action
    → If internal issue → escalate immediately
    → Severity: CRITICAL
```

### Branch B — VAST error / black screen / ad not playing

```
Is it widespread or in a specific environment?
├── Widespread (all devices):
│   → Check base ad tag: does the VAST URL respond?
│   → Check for broken wrapper (depth >5 levels)
│   → Activate video-adtech for diagnosis
│   → Severity: HIGH
│
└── Specific environment (e.g. Samsung only, tvOS only):
    → Check IMA SDK in that environment (version, compatibility)
    → Check RDID/signal on that device
    → Activate video-adtech for device-specific diagnosis
    → Severity: MEDIUM
```

### Branch C — Revenue drop without fill drop

```
→ Likely a pricing problem or reporting discrepancy
→ Activate discrepancy-forensics
→ Check for GAM floor or pricing rule changes in the last 24h
→ Check if SSPs changed floors or deals
→ Severity: MEDIUM-HIGH (may be normal — verify before escalating)
```

---

## Step 2 — Severity classification and response SLA

| Severity | Criterion | Response time | Communication |
|----------|-----------|---------------|---------------|
| **CRITICAL** | Fill at zero across the portfolio, or estimated revenue loss >€500/h | Immediate (<5 min) | Client + internal |
| **HIGH** | Fill at zero for an important client, or widespread VAST error | <15 min | Client informed within <30 min |
| **MEDIUM** | Low but non-zero fill, specific environment, moderate revenue drop | <1 hour | Client update within 1–2h |
| **LOW** | Minor discrepancy, reporting anomaly, issue not directly affecting revenue | <4 hours | Only if client asks |

---

## Step 3 — Mitigation paths (in order of preference)

**Before applying any mitigation, document the current state** (screenshot, GAM export, SSP log).

### Mitigation 1 — Rollback to previous configuration
If the problem coincides with a deploy or configuration change:
```
□ Identify the exact change (commit, GAM config, SDK version)
□ Revert the change
□ Verify within 5 min that fill normalises
□ If not normalising → Mitigation 2
```

### Mitigation 2 — Inventory fallback
If the problem is in a specific SSP or deal type:
```
□ Temporarily disable the affected deal or SSP in GAM
□ Verify that the fallback inventory (open auction or another SSP) is active
□ Confirm fill rises with the fallback active
□ Document the applied fallback for reversal once root cause is resolved
```

### Mitigation 3 — Throttling
If the problem is capacity-related or an SSP is sending invalid traffic:
```
□ Reduce QPS to the problematic SSP in GAM
□ Activate a temporary higher floor to filter low-quality traffic
□ Communicate the problem to the SSP and request diagnosis from their side
```

### Mitigation 4 — Activate emergency inventory
If fill drops with no clear solution:
```
□ Check whether house ads are configured as fallback in GAM
□ Activate PSA (Public Service Announcements) as a last resort
□ Confirm with the client before activating (may affect brand image)
```

---

## Step 4 — Communication templates

### Initial client communication (within the first 30 minutes)

```
Subject: [INCIDENT] Advertising anomaly detected — [CLIENT] — [HH:MM]

We have detected an anomaly in the ad serving for [CLIENT]
starting at [HH:MM] (Madrid time).

Current status: [brief description — e.g. "abnormally low fill rate on Android TV devices"]
Estimated impact: [brief — e.g. "approximately 40% of CTV inventory affected"]
Action underway: [what we are doing — e.g. "active diagnosis, identifying root cause"]

Next update in 30 minutes.

[signature]
```

### Progress update (every 30 minutes until resolution)

```
Subject: [UPDATE] Advertising incident — [CLIENT] — [HH:MM]

Update at [HH:MM]:

Cause identified: [yes / in progress / not yet identified]
[If identified]: [description in non-technical terms]

Action applied: [what was done — or "diagnosis ongoing"]
Current status: [improving / unchanged / worsening]

Next update in [N] minutes.
```

### Resolution communication

```
Subject: [RESOLVED] Advertising incident — [CLIENT] — Resolved [HH:MM]

The incident detected at [HH:MM] has been resolved at [HH:MM].

Total duration: [N hours N minutes]
Root cause: [clear, non-technical description]
Solution applied: [what was done]

Ad serving is operating normally.
Estimated impact: [N impressions affected / €X in revenue]

We will prepare a detailed post-mortem report within the next 24 hours.

[signature]
```

---

## Step 5 — Internal escalation

### When to escalate

```
□ CRITICAL severity: escalate immediately to Sibbo management
□ HIGH severity > 1 hour without resolution: escalate
□ Root cause not identified after 45 minutes of diagnosis: escalate to SSP/GAM support
□ Client is escalating internally within their organisation: get ahead of it and escalate too
```

### Provider support contacts

```
GAM/Google:
  - Support portal: https://support.google.com/admanager
  - For GAM 360: priority support via account manager
  - Priority: P0 for outages, P1 for revenue incidents

PubMatic:
  - Publisher support: [fill in with account manager contact]
  - Open ticket: [PubMatic support portal URL]

Magnite:
  - Publisher support: [fill in]

Xandr:
  - Publisher support: [fill in]
```

---

## Step 6 — Post-mortem (within 24 hours)

Use the post-mortem template from `client-deliverable` to generate the signable document.

Minimum data to capture before closing the incident:

```
□ Exact timeline (start, detection, diagnosis, resolution)
□ Confirmed root cause (not the initial hypothesis — the actual cause)
□ Final blast radius (impressions, revenue, duration)
□ Solution applied
□ Preventive actions (with owner and date)
□ Lessons learned
```

Trigger `knowledge-capture` with the anti-pattern learned if the root cause was not previously documented.

---

## Incident response anti-patterns

- **Applying changes without documenting the previous state:** clean rollback becomes impossible
- **Communicating to the client before understanding the actual impact:** generates unnecessary alarm
- **Not escalating in time:** the pride of solving it alone can cost hours of revenue
- **Confusing symptom with cause:** "fill at zero" is a symptom, not a cause — always find the root
- **Not doing a post-mortem:** the same incident will happen again

---

## Changelog

### v1.0.0 — 2026-04-27
**Type:** MAJOR (creation)
**Origin:** Ecosystem audit — latent pain point P_incident
- Create: initial skill with triage, decision tree, mitigations, and communication templates
- Create: severity table and response SLAs
- Create: post-mortem template integrated with client-deliverable
