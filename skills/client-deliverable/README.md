---
name: client-deliverable
description: Converts verified AdTech findings into client-ready, buyer-ready, executive, or internal deliverables for GAM, Prebid, CTV/video, programmatic deals, SSP/DSP troubleshooting, incidents, audits, and optimisation reports. Activate on: "client report", "deliverable", "executive summary", "post-mortem", "send to client", "buyer request", "deal diagnostic", "email with results", "client-facing", "C-level", "/close [client]", "/deliverable [topic]".
---

# Client Deliverable

**Version:** 1.2.0  
**Status:** Stable — Runtime Optimised  
**Last updated:** 2026-04-27  
**Domain:** AdTech / GAM / Prebid / CTV / Programmatic Deals / Video / SSP-DSP troubleshooting  
**Depends on:** fact-checker (`/verify-deliverable` mode), context-guardian, domain skills  
**Used by:** Agent γ — Deliverable Composer

Transforms technical conclusions into sendable artefacts with evidence control, audience calibration, commercial safety, and explicit next steps.

---

# Execution Flow

1. Intake
2. Evidence Map
3. Audience & Risk Mode
4. Deliverable Builder
5. Sendability Gate

---

# 1. Intake

Extract from session:

- Client / publisher / partner
- Topic, incident, audit, deal, or optimisation
- Findings, decisions, recommendations
- Metrics, IDs, dates, affected inventory
- Recipient and desired format
- Open questions / pending validations

If source material is unclear, ask once:

```text
What session or analysis should this deliverable close? Give me the key points in 3–5 lines and I’ll generate the sendable version.
```

If audience/format is unclear, ask once:

```text
Who is it for and in what format?
A) Client technical team — Markdown / DOCX
B) C-level — Executive PDF / PPTX
C) Mixed — DOCX with executive summary + appendix
D) Buyer / agency / DSP — diagnostic email
E) Internal team — technical post-mortem
F) Slack/email — short message
```

If intent is reasonably clear, choose the best-fit mode and proceed.

---

# 2. Evidence Map

Mandatory for client-facing, buyer-facing, executive-facing, or commercially sensitive deliverables.

```markdown
## Evidence Map

### Confirmed facts
- [Fact] — evidence: [log / report / screenshot / config / session finding / official documentation]

### Reasonable inferences
- [Inference] — basis: [why it follows from evidence]

### Pending validation
- [Unknown] — needed from: [publisher / buyer / DSP / SSP / GAM admin / engineering / vendor]

### Claims not allowed in final version
- [Claim] — reason: unsupported / speculative / exaggerated / commercially unsafe / missing data
```

## Classification rules

| Class | Requirement | Final wording |
|---|---|---|
| Confirmed | Supported by logs, reports, screenshots, config, vendor/buyer confirmation, official docs, reproducible test, or explicit session data | Direct claim allowed |
| Inferred | Plausible and evidence-led, but not directly proven | “The evidence suggests…”, “likely cause…”, “working hypothesis…” |
| Pending | Missing third-party/client/vendor confirmation | Convert into question, dependency, or next step |
| Not allowed | Unsupported, overconfident, blaming, risky, or outside evidence | Remove or mark `[REVIEW]` |

Do not present hypotheses as root causes. Do not assign blame without confirmation.

---

# 3. Audience & Risk Mode

## Mode T — Technical Client Team

Use for AdOps, engineering, GAM admins, wrapper/prebid implementers, video/CTV teams.

```markdown
# [Technical title]

## Context
## Findings
## Root cause / Working hypothesis
## Evidence
## Recommendation
## Implementation notes
## Risks / dependencies
## Next steps
## Technical appendix
```

Tone: precise, technical, evidence-led, direct.

---

## Mode E — Executive / C-Level

Use for CEO, COO, CRO, commercial director, publisher leadership, non-technical management.

```markdown
# [Business-oriented title]

## Situation
## What we found
## Business impact
## Recommended action
## Risk of inaction
## Expected result
## Next milestone
```

Rules:

- Translate technical metrics into business meaning.
- Avoid unexplained acronyms.
- Lead with decision, impact, and risk.

---

## Mode M — Mixed Audience

Use when technical and business stakeholders share the same document.

Structure:

1. Executive summary
2. Business impact
3. Recommended action
4. Technical findings
5. Evidence appendix
6. Open questions

Rule: first page understandable without technical background; detail goes to appendix.

---

## Mode B — Buyer / Agency / DSP / External Programmatic Partner

Use for deal delivery, PMP/PG troubleshooting, DSP filtering, allowlists, bid visibility, seat/campaign mapping, floor mismatch, or partner-side validation.

```markdown
# [Issue] — [Deal ID / Campaign / Inventory] — [Date]

## What we are observing
## Publisher-side checks completed
## Information needed from your side
## Working hypotheses
## Proposed next step
```

Tone: collaborative, evidence-led, non-accusatory, specific.

Prefer:

- “Could you please confirm whether the deal is active on your side?”
- “Publisher-side reporting currently shows no delivery.”
- “To isolate eligibility, filtering, or auction competitiveness, could you confirm…?”

Avoid unsupported blame such as “your DSP is blocking the deal”.

---

## Mode I — Internal Team

Use for internal-only reviews.

```markdown
# Internal diagnostic — [Topic]

## Current state
## What we know
## What we suspect
## What is still missing
## Risks
## Recommended internal action
## External communication line
```

Can be blunter and include internal mistakes, ownership, and stronger hypotheses. If external sharing is needed, create a sanitised client-facing version.

---

## Mode S — Slack / Email

```markdown
Subject/Title: [Topic] — [Client / Deal / Incident] — [Date]

[2–3 lines of context]

✓ [Main result or finding]
• [Detail 1]
• [Detail 2]
• [Detail 3]

Next step: [Concrete action]
```

No long background. Link or attach full report if needed.

---

# 4. Deliverable Builder

| User need | Best output |
|---|---|
| Send to technical team | Markdown or DOCX |
| C-level / steering committee | PDF or PPTX |
| Mixed client audience | DOCX with executive summary + appendix |
| Fast buyer clarification | Email draft |
| Internal incident review | Markdown / DOCX |
| Commercial meeting | PPTX |
| Evidence record | Markdown appendix |

Before finalising client-facing or executive-facing work, activate `/verify-deliverable` from `fact-checker`.

Verify:

- Percentages, revenue, CPM, fill rate, bid rate, win rate, latency, impression loss
- Date ranges, deal IDs, seat IDs, GAM network IDs, ad unit paths
- Version claims, “official recommendation”, “industry standard”, “root cause” statements

Unsupported quantitative claims must be removed, labelled as estimates, or marked `[REVIEW]`.

---

# 5. Sendability Gate

```markdown
## Sendability Gate

- [ ] Quantitative claims supported or marked [REVIEW]
- [ ] Facts separated from hypotheses
- [ ] Pending validations explicit
- [ ] No internal-only assumptions exposed as facts
- [ ] No unsupported blame
- [ ] Acronyms adapted to audience
- [ ] Impact stated without exaggeration
- [ ] Next step explicit
- [ ] Owner/date included where relevant
- [ ] Tone appropriate for recipient
- [ ] Forwardable without extra explanation
```

If the gate fails, label the artefact:

- `DRAFT — NEEDS DATA`
- `CLIENT-READY — WITH REVIEW FLAGS`
- `INTERNAL ONLY`
- `EXECUTIVE SUMMARY — ESTIMATES MARKED`
- `BUYER REQUEST — PENDING CONFIRMATION`

---

# Templates

## 1. Incident Post-Mortem

```markdown
# Post-mortem: [Brief description] — [Client] — [Date]

**Severity:** High / Medium / Low  
**Incident duration:** [HH:MM] ([start] → [resolution])  
**Estimated impact:** [N impressions / €X / affected inventory] [REVIEW if not confirmed]  
**Status:** Resolved / Monitoring / Under investigation

## Executive summary
[2–4 lines: what happened, impact, current status, next step]

## Timeline

| Time (CET) | Event | Evidence |
|---|---|---|
| [HH:MM] | Detection | [source] |
| [HH:MM] | Diagnosis started | [source] |
| [HH:MM] | Cause identified / hypothesis formed | [source] |
| [HH:MM] | Fix or mitigation applied | [source] |
| [HH:MM] | Recovery verified | [source] |

## Root cause / Working hypothesis
[Confirmed root cause only if proven. Otherwise label as working hypothesis.]

## Evidence
- [Evidence 1]
- [Evidence 2]
- [Evidence 3]

## Resolution / Mitigation
[Exactly what was changed or recommended]

## Preventive actions

| Action | Owner | Deadline | Status |
|---|---|---|---|
| [Action 1] | [Owner] | [Date] | Pending |
| [Action 2] | [Owner] | [Date] | Pending |

## Lessons learned
[1–2 concrete operational learnings]

--- DRAFT EMAIL ---
Subject: Post-mortem — [Incident] — [Client] — [Date]

Hi [Name],

Please find attached the post-mortem for [incident]. The issue is currently [resolved / under monitoring], and the next preventive actions are listed in the final section.

Best,
[Signature]
```

---

## 2. Programmatic Optimisation Report

```markdown
# Programmatic optimisation report — [Client] — [Period]

## Executive summary
[2–3 lines: what was optimised, main result, next step]

## Baseline vs result

| Metric | Before | After | Δ | Evidence |
|---|---:|---:|---:|---|
| Fill rate | X% | Y% | +Z% | [source] |
| Average CPM | €X | €Y | +Z% | [source] |
| Revenue | €X | €Y | +Z% | [source] |
| Bid rate | X% | Y% | +Z% | [source] |

## Changes made

| Change | Date | Expected / observed impact | Evidence |
|---|---|---|---|
| [Change 1] | [Date] | [Impact] | [source] |
| [Change 2] | [Date] | [Impact] | [source] |

## Interpretation
[Separate confirmed results from hypotheses.]

## Recommendations

| Priority | Recommendation | Expected impact | Dependency |
|---|---|---|---|
| High | [Action] | [Impact or REVIEW] | [Dependency] |
| Medium | [Action] | [Impact or REVIEW] | [Dependency] |

## Next milestone
[Concrete next action, owner, date]

--- DRAFT EMAIL ---
Subject: Optimisation report — [Client] — [Period]

Hi [Name],

Attached is the optimisation report for [period]. The main finding is [one-line result]. Recommended next steps are summarised on page [X].

Best,
[Signature]
```

---

## 3. Technical Recommendation for C-Level

```markdown
# Recommendation: [Business-oriented title] — [Date]

## Current situation
[One paragraph, low jargon]

## Opportunity identified
[What can be improved and why it matters commercially]

## Recommended action
[What to do, timeframe, required resources]

## Expected impact
[Quantified only if supported. Otherwise label as estimate.]

## Risk of inaction
[Consequence of not acting, expressed in business terms]

## Decision required
[Clear decision needed from leadership]

## Next step
[Concrete owner/date]

--- DRAFT EMAIL ---
Subject: Recommendation — [Topic] — [Date]

Hi [Name],

Attached is a short recommendation note on [topic]. The proposed decision is [decision], with the main expected benefit being [benefit].

Best,
[Signature]
```

---

## 4. Deal Not Delivering Diagnostic

```markdown
# Deal delivery diagnostic — [Client] — [Deal ID] — [Date]

## Executive summary
[1 short paragraph: what is happening, business impact, current working hypothesis]

## Current observation
- Deal ID: [Deal ID]
- Buyer / DSP: [Buyer / DSP]
- Inventory: [Ad units / domains / apps]
- Expected delivery: [Expectation]
- Observed delivery: [Observed status]
- Period checked: [Date range]

## Publisher-side checks

| Area | Status | Evidence | Owner |
|---|---|---|---|
| Inventory eligibility | Confirmed / Review / Pending | [source] | Publisher |
| Deal active in GAM | Confirmed / Review / Pending | [source] | Publisher |
| Line item / yield group active | Confirmed / Review / Pending | [source] | Publisher |
| Dates and timezone | Confirmed / Review / Pending | [source] | Publisher / Buyer |
| Floors | Confirmed / Review / Pending | [source] | Publisher / Buyer |
| Domain/app/bundle signals | Confirmed / Review / Pending | [source] | Publisher / Buyer |
| Consent / privacy signals | Confirmed / Review / Pending | [source] | Publisher / Buyer |
| Bid request visibility | Pending | DSP confirmation needed | Buyer / DSP |
| Bid response / filtering reason | Pending | DSP logs needed | Buyer / DSP |

## Most likely causes

| Hypothesis | Confidence | Evidence | Validation needed |
|---|---|---|---|
| [Cause 1] | High / Medium / Low | [basis] | [needed] |
| [Cause 2] | High / Medium / Low | [basis] | [needed] |

## Information requested from buyer / DSP
Could you please confirm:

1. Whether the deal is active in the DSP and linked to the correct campaign / advertiser / seat.
2. Whether your platform is receiving bid requests for Deal ID [X].
3. Whether domain, app bundle, device, geo, or content filters are blocking eligibility.
4. Whether the floor seen from your side matches the agreed deal economics.
5. Whether there are bid responses, no-bids, or filtering reasons available in DSP logs.
6. Whether there are any pending changes requiring buyer-side acceptance.

## Recommended next step
Once buyer-side checks are confirmed, isolate whether the issue is publisher eligibility, buyer/DSP filtering, floor competitiveness, deal mismatch, or demand-side activation.

--- DRAFT EMAIL ---
Subject: Deal delivery check — [Client] — Deal ID [X]

Hi [Name],

We are currently seeing [low/no] delivery from publisher-side reporting for Deal ID [X]. We have completed the main publisher-side checks and would need your confirmation on buyer/DSP-side visibility and filtering to isolate the cause.

Could you please confirm the items listed below?

[Short bullet list]

Best,
[Signature]
```

---

## 5. Prebid / Wrapper Technical Audit Summary

```markdown
# Prebid implementation audit — [Client] — [Date]

## Executive summary
[2–4 lines: current state, severity, main risk, recommended action]

## Scope reviewed
- Prebid version: [version]
- GAM integration: [yes/no/details]
- APS / Amazon integration: [yes/no/details]
- CMP / consent flow: [yes/no/details]
- Formats: Display / Video / CTV / Native
- Pages / environments tested: [list]

## Key findings

| Finding | Severity | Evidence | Impact | Recommendation |
|---|---|---|---|---|
| [Finding 1] | High / Medium / Low | [source] | [impact] | [action] |
| [Finding 2] | High / Medium / Low | [source] | [impact] | [action] |

## Critical issues
[Detailed explanation of high-severity issues]

## Medium / low priority improvements
[Operational or structural improvements]

## Recommended implementation plan

| Priority | Action | Owner | Estimated effort | Dependency |
|---|---|---|---|---|
| P0 | [Action] | [Owner] | [Effort] | [Dependency] |
| P1 | [Action] | [Owner] | [Effort] | [Dependency] |

## Technical appendix
[Logs, configs, screenshots, code references, tested URLs]

--- DRAFT EMAIL ---
Subject: Prebid audit summary — [Client] — [Date]

Hi [Name],

Attached is the Prebid implementation audit. The main priority is [priority], as it may affect [impact]. Recommended next steps are listed in the implementation plan.

Best,
[Signature]
```

---

## 6. CTV / Video Monetisation Recommendation

```markdown
# CTV / Video monetisation recommendation — [Client] — [Date]

## Executive summary
[2–4 lines: current issue, opportunity, recommended direction]

## Current setup
- Platform(s): Android TV / tvOS / Tizen / webOS / Web / Mobile web
- Ad server: GAM / GAM 360
- Integration: CSAI / SSAI / IMA SDK / custom player
- Formats: preroll / midroll / postroll / VMAP / VAST
- Demand: Direct / Open Auction / PMP / PG / Prebid / APS

## Key findings

| Area | Current status | Risk / opportunity | Evidence |
|---|---|---|---|
| VAST signal quality | [status] | [risk] | [source] |
| Device identifiers | [status] | [risk] | [source] |
| Consent signals | [status] | [risk] | [source] |
| Content signals | [status] | [risk] | [source] |
| Player compatibility | [status] | [risk] | [source] |
| Measurement / OMID / Active View | [status] | [risk] | [source] |

## Recommendation
[Clear recommendation by priority]

## Expected impact
[Only quantified if supported; otherwise describe qualitatively]

## Implementation plan

| Phase | Action | Owner | Deadline |
|---|---|---|---|
| 1 | [Action] | [Owner] | [Date] |
| 2 | [Action] | [Owner] | [Date] |

## Open validations
[Questions for Google, player team, app team, buyer, DSP, or vendor]

--- DRAFT EMAIL ---
Subject: CTV monetisation recommendation — [Client] — [Date]

Hi [Name],

Attached is our recommendation for the CTV/video monetisation setup. The main priority is [priority], because it affects [business/technical impact].

Best,
[Signature]
```

---

## 7. Client-Facing Technical Explanation Email

```markdown
Subject: [Topic] — [Client] — [Date]

Hi [Name],

Following our review of [topic], we have identified [main finding in one sentence].

At this stage, the confirmed points are:
- [Confirmed point 1]
- [Confirmed point 2]
- [Confirmed point 3]

The current working hypothesis is [hypothesis], pending confirmation from [party/source].

Recommended next step: [specific action].

Best,
[Signature]
```

---

# Anti-Patterns

## AP-CD-001 — Unsupported quantitative claims

Bad:

```text
This change will increase revenue by 30%.
```

Good:

```text
Based on the current test window, revenue increased by 30%. This should be validated over a longer period before treating it as stable.
```

Or:

```text
We expect a positive revenue impact, but there is not enough data yet to quantify it.
```

---

## AP-CD-002 — Hypothesis presented as root cause

Bad:

```text
The deal is not delivering because the buyer has an allowlist issue.
```

Good:

```text
A buyer-side allowlist restriction is one of the most likely explanations, but this requires DSP confirmation.
```

---

## AP-CD-003 — Over-technical executive summary

Bad:

```text
The hb_pb targeting did not propagate due to a race condition between setTargetingForGPTAsync and refresh.
```

Good:

```text
Some eligible impressions may not have been exposed to programmatic demand because the auction data was not consistently available before the ad request.
```

---

## AP-CD-004 — Blame-first buyer communication

Bad:

```text
Your DSP is not bidding even though the deal is active.
```

Good:

```text
Publisher-side reporting currently shows no delivery. Could you confirm whether the DSP is receiving bid requests and whether any filters are being applied?
```

---

## AP-CD-005 — No next step

Bad:

```text
We found several possible causes and will keep checking.
```

Good:

```text
Next step: buyer to confirm bid request visibility and filtering reason for Deal ID [X] by [date].
```

---

## AP-CD-006 — Hiding uncertainty

Bad:

```text
Everything is configured correctly.
```

Good:

```text
The publisher-side configuration checks completed so far do not show an obvious blocker. Buyer-side activation and filtering still need confirmation.
```

---

## AP-CD-007 — Sending internal diagnosis externally

Bad:

```text
Our setup may have been wrong and the buyer probably configured the wrong seat too.
```

Good:

```text
We are reviewing both publisher-side configuration and buyer-side activation to isolate the delivery blocker.
```

---

# Quality Rubric

| Score | Meaning |
|---:|---|
| 6 | Readable but generic; weak evidence control |
| 7 | Useful, but not fully audience-calibrated |
| 8 | Solid client deliverable with clear recommendations |
| 9 | Evidence-controlled, audience-calibrated, actionable, safe to forward |
| 9.5 | Precise, commercially safe, concise, technically defensible, decision-ready |

A 9+ deliverable requires:

- Evidence Map completed
- Claims verified
- Audience mode selected correctly
- Clear next step
- Unknowns handled honestly
- No unsupported blame
- No hidden assumptions
- Format appropriate to channel

---

# Output Rules

| Format | Use when | Requirement |
|---|---|---|
| DOCX | Formal client / mixed technical report | Executive summary + technical appendix |
| PPTX | C-level / commercial meeting | One idea per slide + decision slide |
| PDF | Sealed final deliverable | No editable assumptions left unresolved |
| Markdown | Iteration / internal / technical evidence | Clear headings + evidence markers |
| Email / Slack | Short update / buyer request / escalation | 3–5 lines plus concrete next step |

Every full deliverable includes this closing block unless the user says otherwise:

```markdown
--- DRAFT [EMAIL / SLACK / MESSAGE] ---
To: [recipient]
Subject: [topic] — [date]

[Message body in 3–5 lines maximum]

[Attachment/reference if applicable]

[signature]
```

---

# Changelog

## v1.2.0 — 2026-04-27

**Type:** Runtime optimisation

### Kept from v1.1

- Evidence Map
- Buyer / Agency / DSP mode
- Internal mode
- Sendability Gate
- Deal not delivering template
- Prebid / wrapper audit template
- CTV / video recommendation template
- Anti-patterns with bad/good examples
- Claim verification rules

### Reduced / removed

- Long Purpose section
- Long Activation section
- Redundant stage goal paragraphs
- Terminology lists
- Verbose file generation rules
- Separate tone calibration section
- Final design intent prose
- Repetitive explanatory framing

### Result

Captures most of v1.1’s behavioural value with substantially lower runtime overhead.

