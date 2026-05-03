---
name: incident-response
description: War-room response flow for production incidents affecting ad serving, GAM, Prebid, CTV/video, programmatic deals, SSP/DSP integrations, consent, reporting, or revenue. Activate for: "incident", "outage", "ads not serving", "fill at zero", "black screen", "VAST error", "urgent rollback", "client reports no ads", "sudden fill drop", "revenue collapse", "deal stopped delivering", "wrapper issue in production", "CMP broke ads", or any high-pressure AdTech situation requiring fast triage, controlled mitigation, escalation, communication, and post-mortem capture.
---

# Incident Response

**Version:** 1.1.1  
**Status:** Stable — War-Room Lean  
**Last updated:** 2026-04-27  
**Domain:** AdTech / GAM / Prebid / CTV / Video / Programmatic Deals / Consent / SSP-DSP / Reporting  
**Depends on:** gam-adops, prebid-adtech, video-adtech, programmatic-deals, cmp-consent-flows, discrepancy-forensics, client-deliverable  
**Used by:** Standalone incident coordinator

This skill manages the incident response flow. It does not replace specialised technical diagnosis. Its job is to keep the response controlled under pressure: triage, severity, roles, evidence, mitigation, escalation, communication, verification, and post-mortem capture.

---

# Core Rules

1. Stabilise before optimising.
2. Do not change multiple variables at once unless containment requires it.
3. Every mitigation needs owner, timestamp, expected effect, success criterion, and rollback path.
4. Separate symptom, hypothesis, root cause, mitigation, and fix.
5. Communicate impact honestly; do not speculate as fact.
6. Do not close until recovery is verified with data.
7. Capture timeline while the incident is happening, not after memory degrades.

---

# Incident States

| State | Meaning | Exit condition |
|---|---|---|
| DETECTED | Alert/client/internal team reports anomaly | First triage started |
| TRIAGED | What/where/impact/start time roughly known | Severity assigned + owner named |
| CONTAINING | Active steps to reduce damage | Containment applied or rejected with reason |
| DIAGNOSING | Technical root cause investigation ongoing | Root cause found or working hypothesis strong enough for action |
| MITIGATED | Business impact reduced but root cause may remain | Serving/revenue partially restored or risk reduced |
| FIXING | Permanent fix or rollback being applied | Fix completed |
| VERIFYING | Checking recovery with data | Metrics normalised for defined window |
| RESOLVED | Incident no longer active | Client/internal resolution sent |
| POST-MORTEM | Final cause, impact, prevention captured | Post-mortem delivered / knowledge captured |

Never jump from DETECTED to RESOLVED without evidence of VERIFYING.

---

# 0. Assign Roles

For HIGH or CRITICAL incidents, assign roles immediately.

| Role | Responsibility |
|---|---|
| Incident Commander | Owns severity, priorities, decisions, escalation, and prevents chaotic changes |
| Technical Lead | Runs diagnosis using relevant technical skills/tools |
| Comms Owner | Sends client/internal updates and avoids unsupported claims |
| Scribe | Maintains timeline, actions, evidence, and decisions |
| Approver | Authorises risky mitigations: rollback, disabling SSP/deal, PSA, major GAM changes |

For smaller incidents, one person may hold multiple roles, but Incident Commander and Technical Lead must still be explicit.

---

# 1. Immediate Triage — First 5 Minutes

Answer before changing anything unless there is obvious active harm requiring emergency containment.

```markdown
## Triage Snapshot

1. WHAT is failing?
- Fill at zero / low fill
- VAST error / black screen / ad not playing
- Revenue drop without fill drop
- Deal/PMP not delivering
- Reporting discrepancy
- Consent/privacy signal issue
- Wrapper / Prebid / targeting issue
- GAM trafficking / pricing / policy issue
- Other: [describe]

2. WHERE does it affect?
- Client(s): [name]
- Inventory: [ad unit/domain/app/platform/content]
- Environment: web / mobile web / Android TV / tvOS / Samsung / LG / HbbTV / app
- Demand: direct / open auction / PMP / PG / SSP / DSP / buyer
- Geography/device/browser/app version if relevant

3. HOW BIG is the blast radius?
- % affected inventory: [estimate / unknown]
- Revenue at risk per hour: [estimate / unknown]
- Requests/impressions affected: [estimate / unknown]
- Users/viewers affected: [estimate / unknown]

4. WHEN did it start?
- Exact or approximate start time:
- Detection time:
- Coincides with deploy/configuration/vendor change? yes/no/unknown

5. CURRENT STATE
- State: DETECTED / TRIAGED / CONTAINING / DIAGNOSING / MITIGATED / FIXING / VERIFYING / RESOLVED
- Severity: CRITICAL / HIGH / MEDIUM / LOW
- Incident Commander:
- Technical Lead:
```

---

# 2. Severity Classification

| Severity | Criteria | Response target | Communication |
|---|---|---|---|
| CRITICAL | Fill at zero across portfolio; major ad serving outage; black screen at scale; estimated loss >€500/h; reputationally sensitive live event | Immediate, under 5 min | Internal + client immediately |
| HIGH | Major client affected; widespread VAST/player error; important deal/CTV inventory stopped; loss likely material | Under 15 min | Client informed within 30 min |
| MEDIUM | Specific environment, low but non-zero fill, moderate revenue drop, contained deal issue | Under 1h | Client update within 1–2h if material |
| LOW | Minor discrepancy, reporting anomaly, non-urgent issue, no active revenue/user impact | Under 4h | Only if client asks or issue persists |

Escalate severity if impact grows, uncertainty remains high, or client pressure rises.

---

# 3. Change Freeze

For HIGH or CRITICAL incidents:

```markdown
## Change Freeze

- Freeze all non-essential changes.
- Only incident-related mitigations are allowed.
- Each change must be logged before execution.
- Do not change more than one major variable at once unless approved as emergency containment.
- Every action must have rollback path and success criterion.
```

Use this log:

```markdown
| Time CET | Actor | Action | Reason | Expected effect | Success criterion | Result | Evidence | Rollback path |
|---|---|---|---|---|---|---|---|---|
| HH:MM | [name] | [change/check] | [why] | [expected] | [metric/window] | [result] | [link/screenshot/log] | [how to revert] |
```

---

# 4. Branch Selection

Pick the dominant symptom. If several apply, start with the branch most directly affecting serving/revenue.

| Branch | Use when |
|---|---|
| A. GAM / Ad server | Ad server delivery, line items, pricing rules, ad units, creatives, policies, GAM status |
| B. Prebid / Wrapper | Header bidding, targeting, bidder calls, timeouts, auction flow, setTargeting, refresh |
| C. CTV / VAST / Player | VAST errors, black screen, video not playing, IMA, SDK/device-specific failures |
| D. Programmatic Deals | PMP/PG not delivering, buyer not bidding, DSP filtering, floors, allowlists |
| E. Consent / Privacy | CMP failures, missing TC string, GDPR/TCF/GPP/USP issues, consent mode changes |
| F. SSP / DSP / Demand Partner | SSP down, no-bids at scale, bidder endpoint errors, partner-specific collapse |
| G. Reporting / Discrepancy | Revenue/reporting drop without serving symptoms, mismatched systems |
| H. Revenue Drop Without Fill Drop | CPM collapse, floor/pricing issue, demand mix shift, deal displacement |

---

# Branch A — GAM / Ad Server Incident

Immediate checks:

```markdown
- GAM status page checked: yes/no
- Affected ad unit(s): [path]
- Recent changes in last 24h: UPR, line items, creatives, targeting, protections, competitive exclusions
- Delivery tools / preview tested: yes/no
- Inventory availability checked: yes/no
- Ad request reaching GAM: yes/no/unknown
- Unfilled reason available: yes/no/unknown
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| Rollback GAM change | Issue coincides with known config change | Export/screenshot before rollback |
| Disable problematic rule/protection | Rule blocks serving unexpectedly | Confirm fallback before action |
| Activate safe fallback line item | Paid ads unavailable but blank inventory unacceptable | Client approval if brand-sensitive |
| Narrow affected targeting | Bad targeting blocks delivery | Validate with preview/report |

Escalate to `gam-adops` for diagnosis.

---

# Branch B — Prebid / Wrapper Incident

Immediate checks:

```markdown
- Was wrapper deployed recently? yes/no
- Previous script available for rollback? yes/no
- pbjs object present? yes/no
- Ad units built correctly? yes/no/unknown
- Bid requests firing? yes/no/unknown
- Bid responses received? yes/no/unknown
- setTargetingForGPTAsync executed before refresh/display? yes/no/unknown
- Consent/CMP gate blocking auction? yes/no/unknown
- Console/network errors captured? yes/no
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| Roll back wrapper | Incident starts after deploy | Verify hb_* targeting and fill after rollback |
| Disable problematic adapter | One bidder breaks auction or causes timeouts | Confirm auction still runs |
| Increase failsafe temporarily | Auctions timing out before ad server call | Avoid masking root cause permanently |
| Bypass experimental module | New module breaks flow | Document module/version |

Escalate to `prebid-adtech` for diagnosis.

---

# Branch C — CTV / VAST / Player Incident

Immediate checks:

```markdown
- VAST URL responds: yes/no
- VAST XML valid: yes/no/unknown
- Error code(s): [VAST/IMA/player]
- Widespread or device-specific: [scope]
- Recent player/app/SDK/config change: yes/no
- Creative type involved: direct / programmatic / wrapper / VPAID / mezzanine / unknown
- Wrapper depth: [value/unknown]
- Timeout observed: yes/no
- Consent/device identifiers present: yes/no/unknown
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| Disable problematic creative/deal | Specific creative causes playback failure | Confirm alternate demand fills |
| Reduce wrapper complexity | Wrapper depth/timeouts causing errors | Track fill/revenue impact |
| Roll back player/app config | Issue coincides with release | Verify on affected devices |
| Disable affected environment temporarily | One platform causes user-facing failure | Requires approval if revenue impact large |
| Switch to safe VAST output/format | Format incompatibility suspected | Confirm with player test |

Escalate to `video-adtech` for diagnosis.

---

# Branch D — Programmatic Deals Incident

Immediate checks:

```markdown
- Deal ID:
- Buyer / DSP / seat:
- Deal type: PMP / PG / Preferred Deal / Programmatic Guaranteed
- Dates/timezone valid: yes/no/unknown
- Deal active in GAM: yes/no/unknown
- Eligible impressions available: yes/no/unknown
- Floor / UPR conflict checked: yes/no/unknown
- Domain/app/bundle allowlist checked: yes/no/unknown
- Buyer-side active status confirmed: yes/no/pending
- Bid requests visible to buyer: yes/no/pending
- Bid responses / filter reason from DSP: yes/no/pending
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| Confirm buyer activation before changes | Publisher side looks eligible | Avoid unnecessary GAM edits |
| Temporarily lower conflicting floor | Strong evidence floor blocks buyer | Track CPM/revenue impact |
| Open fallback demand | Deal not buying and inventory going unfilled | Confirm fallback does not violate commercial terms |
| Pause broken deal route | Deal causes errors or blocks fallback | Requires commercial awareness |
| Escalate to buyer/DSP | Need bid visibility/filtering reason | Use buyer-facing template |

Escalate to `programmatic-deals` for diagnosis and `client-deliverable` for buyer email if needed.

---

# Branch E — Consent / Privacy Incident

Immediate checks:

```markdown
- CMP loaded: yes/no
- Consent event fired: yes/no/unknown
- TC string present: yes/no/unknown
- gdpr/gdpr_consent or GPP fields present where required: yes/no/unknown
- Applies to all users or specific geography: [scope]
- Recent CMP/vendor list change: yes/no
- Prebid consentManagement config changed: yes/no
- GAM privacy settings changed: yes/no
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| Roll back CMP change | Incident follows CMP release | Legal/privacy owner awareness required |
| Restore previous vendor list/config | Vendors unexpectedly blocked | Document exact version |
| Adjust auction delay/failsafe | Consent callback delayed | Do not bypass legal requirements |
| Disable affected experimental consent path | New integration unstable | Preserve compliant baseline |

Escalate to `cmp-consent-flows` and legal/privacy owner if compliance-sensitive.

---

# Branch F — SSP / DSP / Demand Partner Incident

Immediate checks:

```markdown
- Partner affected:
- Status page checked: yes/no
- Endpoint/network errors: yes/no/unknown
- Timeout rate changed: yes/no/unknown
- Other demand sources normal: yes/no/unknown
- Recent adapter/config change: yes/no
- Partner contacted: yes/no
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| Disable or throttle affected partner | Partner causes timeouts or bad responses | Verify other demand absorbs traffic |
| Reduce timeout impact | Partner slows auction | Monitor bid rate and latency |
| Shift demand to alternatives | Partner down but inventory valid | Track CPM/fill tradeoff |
| Wait and communicate | Confirmed external outage with no local workaround | Avoid random local changes |

Escalate to partner support and `prebid-adtech` if adapter-related.

---

# Branch G — Reporting / Discrepancy Incident

Immediate checks:

```markdown
- Which systems disagree:
- Metric affected: requests / impressions / fill / revenue / CPM / clicks
- Real-time or delayed report:
- Serving symptoms present: yes/no
- Data freshness checked: yes/no
- Timezone alignment checked: yes/no
- Currency/net-gross alignment checked: yes/no
- Sampling/filtering differences checked: yes/no
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| No serving change | Reporting-only issue likely | Avoid creating real outage |
| Validate with independent source | Dashboard suspect | Use GAM + logs + SSP if possible |
| Communicate as reporting anomaly | Serving normal but numbers delayed | State uncertainty clearly |

Escalate to `discrepancy-forensics`.

---

# Branch H — Revenue Drop Without Fill Drop

Immediate checks:

```markdown
- Fill stable: yes/no
- CPM changed: yes/no
- Demand mix changed: yes/no
- Floors/UPR changed in last 24–48h: yes/no
- Major buyer/SSP spend changed: yes/no/unknown
- Deal delivery changed: yes/no/unknown
- Direct campaigns displacing programmatic: yes/no/unknown
- Reporting net/gross/currency issue ruled out: yes/no
```

Containment options:

| Mitigation | Use when | Control |
|---|---|---|
| Roll back floor/pricing rule | CPM drop follows pricing change | Monitor fill and CPM together |
| Restore previous demand priority | Misprioritisation suspected | Check direct/programmatic impact |
| Escalate to buyer/SSP | Demand spend changed externally | Request spend/bid visibility |
| No immediate change | Reporting delay or market fluctuation possible | Validate before touching setup |

Escalate to `gam-adops`, `programmatic-deals`, or `discrepancy-forensics` depending on evidence.

---

# 5. Mitigation Discipline

Before any mitigation:

```markdown
- Current state documented: yes/no
- Evidence captured: screenshot/report/log/export
- Owner assigned:
- Expected effect:
- Success metric:
- Verification window:
- Rollback path:
- Approval needed: yes/no
```

Preferred order:

1. Roll back known recent change.
2. Disable only the narrow broken component.
3. Route to safe fallback demand.
4. Throttle problematic partner/component.
5. Activate emergency inventory only if user experience or contractual requirement demands it.

High-risk actions requiring approval:

- Disabling a major SSP
- Disabling a major deal/PG
- Changing global floors/UPR
- Activating PSA/house ads on premium inventory
- Changing consent behaviour
- Rolling back app/player release
- Large-scale wrapper rollback if multiple clients share it

---

# 6. Verification

A fix is not verified because someone says “looks better”. Use metrics.

```markdown
## Verification Checklist

- [ ] Affected ad requests returning expected responses
- [ ] Fill rate recovered or trending normally
- [ ] Revenue/CPM normalising if relevant
- [ ] VAST/player errors reduced to baseline if relevant
- [ ] hb_* targeting or bidder responses restored if Prebid-related
- [ ] Deal delivery or bid visibility confirmed if deal-related
- [ ] Consent signals present and valid if privacy-related
- [ ] No new regression in fallback/demand mix
- [ ] Verification window completed: [N minutes]
```

Suggested minimum verification windows:

| Incident type | Minimum window |
|---|---|
| CRITICAL portfolio outage | 15–30 min after fix |
| HIGH major client issue | 15 min after fix |
| CTV/device-specific playback | Tested on affected device + 15 min monitoring |
| Deal delivery issue | Until buyer/DSP confirms bid visibility or delivery resumes |
| Reporting discrepancy | Until data freshness/timezone issue confirmed |

---

# 7. Communication

## Rules

- Communicate state, impact, action, and next update.
- Do not claim root cause until confirmed.
- Do not blame buyer, SSP, GAM, or internal team without evidence.
- If impact is unknown, say it is being quantified.
- For HIGH/CRITICAL, update cadence should be explicit.
- Remove “next update in X minutes” only if you cannot actually sustain that cadence.

## Initial client communication

```text
Subject: [INCIDENT] Advertising anomaly detected — [CLIENT] — [HH:MM]

We have detected an advertising anomaly affecting [CLIENT] from approximately [HH:MM] Madrid time.

Current status: [brief description]
Known scope: [inventory/platform/client affected, or "still being quantified"]
Action underway: [triage / rollback / partner escalation / containment]

We will provide the next update at [HH:MM] or sooner if the status changes materially.

[signature]
```

## Progress update

```text
Subject: [UPDATE] Advertising incident — [CLIENT] — [HH:MM]

Update at [HH:MM]:

Current state: [triaging / containing / diagnosing / mitigated / verifying]
Impact: [known impact or "still being quantified"]
Cause: [confirmed / working hypothesis / still under investigation]
Action taken: [what was done]
Next step: [what happens next]

Next update: [HH:MM]
```

## Resolution communication

```text
Subject: [RESOLVED] Advertising incident — [CLIENT] — Resolved [HH:MM]

The incident detected at approximately [HH:MM] has been resolved at [HH:MM].

Total duration: [N hours N minutes]
Confirmed root cause: [clear explanation, or "root cause still under final validation" if not fully confirmed]
Action applied: [what was done]
Current status: ad serving has been verified as normal for [verification window].
Estimated impact: [N impressions / €X / affected inventory] [or "being finalised"]

We will prepare a post-mortem with final impact and preventive actions.

[signature]
```

## Internal executive update

```text
Status: [CRITICAL/HIGH/MEDIUM] incident affecting [client/inventory]
Start: [HH:MM]
Current state: [state]
Impact: [known/estimated/unknown]
Owner: [Incident Commander]
Main risk: [revenue/client/user experience]
Next decision needed: [approval/escalation/none]
```

---

# 8. Escalation

Escalate immediately when:

```markdown
- Severity is CRITICAL
- HIGH incident unresolved after 60 minutes
- Root cause unknown after 45 minutes of active diagnosis
- Client is escalating internally
- Issue affects live event, premium campaign, PG/SLA, or strategic client
- Mitigation requires risky commercial/technical decision
- Suspected GAM/SSP/DSP/vendor outage
- Consent/privacy compliance risk exists
```

Provider escalation data to include:

```markdown
- Client/publisher:
- Network/account ID:
- Deal ID / seat ID / ad unit path / app bundle / domain:
- Start time and timezone:
- Affected platforms/devices/geos:
- Request IDs / auction IDs / VAST URLs / HAR / logs:
- Expected vs observed behaviour:
- Changes already tested:
- Business impact estimate:
```

---

# 9. Post-Mortem Capture

Trigger `client-deliverable` post-mortem once RESOLVED.

Minimum data before closure:

```markdown
- Final timeline: start, detection, triage, mitigation, fix, verification, resolution
- Confirmed root cause, or explicit unresolved root cause status
- Final blast radius: inventory, platforms, impressions, revenue, duration
- Customer/user impact
- Actions taken with timestamps
- What worked / what failed
- Preventive actions with owner/date
- Monitoring or alerting gaps
- Whether knowledge-capture should record a new anti-pattern
```

Do not close as “resolved” if only the symptom disappeared and no verification window was completed. Use “mitigated” or “monitoring” instead.

---

# Anti-Patterns

## AP-IR-001 — Changing too many variables

Bad:

```text
We changed floors, disabled two SSPs, reverted the wrapper, and adjusted GAM priorities at the same time.
```

Good:

```text
One mitigation was applied at 12:10, with expected effect and rollback path. Metrics were checked before the next change.
```

---

## AP-IR-002 — Treating symptom as cause

Bad:

```text
Root cause: fill dropped to zero.
```

Good:

```text
Symptom: fill dropped to zero. Root cause: [confirmed cause], or working hypothesis pending validation.
```

---

## AP-IR-003 — Client communication before scope is known

Bad:

```text
Everything is down.
```

Good:

```text
We are seeing an anomaly affecting [known scope]. The full impact is still being quantified.
```

---

## AP-IR-004 — No rollback path

Bad:

```text
We changed the global pricing rule to see if it helps.
```

Good:

```text
We are reverting pricing rule [X] to its previous value. Rollback path: restore exported config [Y]. Success criterion: fill returns to baseline within 15 minutes.
```

---

## AP-IR-005 — Waiting too long to escalate

Bad:

```text
We spent three hours investigating internally before contacting the SSP.
```

Good:

```text
After 45 minutes without confirmed root cause, the partner was escalated with logs, timestamps, IDs, and observed behaviour.
```

---

## AP-IR-006 — Declaring resolved without verification

Bad:

```text
It seems fine now, closing.
```

Good:

```text
The incident is resolved after 20 minutes of normal fill and no recurrence of VAST errors on the affected platform.
```

---

## AP-IR-007 — Overconfident root cause in final email

Bad:

```text
The DSP caused the outage.
```

Good:

```text
The current evidence indicates the issue originated in the DSP path. We are waiting for final partner confirmation for the post-mortem.
```

---

## AP-IR-008 — No incident log

Bad:

```text
We reconstructed the timeline from memory the next day.
```

Good:

```text
Every action, owner, evidence point, and result was logged during the incident.
```

---

# Quality Rubric

| Score | Meaning |
|---:|---|
| 6 | Basic checklist; useful but weak under pressure |
| 7 | Good triage and templates, limited operational control |
| 8 | Solid incident flow with severity, mitigation, and comms |
| 9 | War-room ready: roles, states, evidence, controlled changes, escalation, verification |
| 9.5 | Excellent: protects revenue, prevents chaotic changes, supports client trust, and produces clean post-mortem data |

A 9+ incident response requires:

- Incident Commander assigned
- State tracked
- Triage snapshot completed
- Severity assigned
- Change freeze applied when needed
- Incident log maintained
- Mitigations controlled with rollback and success criteria
- Technical branch selected
- Client/internal comms calibrated
- Recovery verified with data
- Post-mortem data captured

---

# Changelog

## v1.1.1 — 2026-04-27

**Type:** Runtime lean revision

### Removed

- Redundant `Typical symptoms` blocks inside each branch; branch selection table already covers symptom routing.

### Kept

- Incident states
- Roles
- Change freeze
- Incident log
- AdTech branches
- Mitigation discipline
- Verification windows
- Escalation package
- Anti-patterns
- Quality rubric

---

## v1.1.0 — 2026-04-27

**Type:** Major operational upgrade

### Added

- Incident state machine
- Explicit incident roles
- Change freeze
- Incident action log
- Branches for GAM, Prebid/wrapper, CTV/VAST/player, deals, consent/privacy, SSP/DSP, discrepancy, and revenue drop without fill drop
- Mitigation discipline with owner, evidence, success criterion, and rollback path
- Verification windows
- Stronger escalation package
- Anti-patterns with bad/good examples
- Quality rubric

### Changed

- Reframed skill from linear checklist to war-room operating model
- Strengthened separation between containment, diagnosis, mitigation, fix, verification, and post-mortem
- Reduced risk of uncontrolled changes during high-pressure incidents

### Integrations

- Use `gam-adops` for GAM-specific diagnosis
- Use `prebid-adtech` for wrapper/header bidding diagnosis
- Use `video-adtech` for VAST/CTV/player diagnosis
- Use `programmatic-deals` for PMP/PG/buyer/DSP diagnosis
- Use `cmp-consent-flows` for privacy/CMP diagnosis
- Use `discrepancy-forensics` for reporting mismatch
- Use `client-deliverable` for post-mortem and client-ready reports
