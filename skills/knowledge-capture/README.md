---
name: knowledge-capture
description: >
  Cross-cutting meta-learning skill. Converts verified technical findings from
  conversations into structured entries for the AdTech ecosystem skills
  (gam-adops, prebid-adtech, video-adtech, programmatic-deals, cmp-consent-flows).
  Explicit activation with /learn, /capture or /knowledge, or autonomous activation
  upon detecting a verified technical resolution in the thread. Use it when a bug is
  resolved, an undocumented behavior is confirmed, a new anti-pattern is identified,
  or when the user says "save this", "we need to remember this", "add this to the
  skill", "new anti-pattern", "lesson learned", or any variant indicating intent to
  preserve knowledge. Never writes anything without explicit user approval.
---

# Knowledge Capture — Meta-learning skill

Selective consolidation layer for verified technical knowledge. Its purpose is to avoid rediscovering the same findings in future conversations — not to accumulate memory, but to distill what deserves to be reusable.

---

## Activation triggers

### Explicit
The user writes `/learn`, `/capture`, `/knowledge`, "save this", "add this to the skill", "new anti-pattern", "lesson learned", or similar variants.

### Autonomous
Activate when **all** of the following indicators are detected in the same thread:
1. A concrete technical symptom or problem was described
2. A diagnosis with an identified root cause was reached (not just a fix)
3. The finding is validated — at least one of these three cases:
   - **Verified fix:** applied and the user confirmed it worked
   - **Verified behavior:** demonstrated by official documentation or a reproducible test
   - **Probable anti-pattern:** solid pattern but without production validation — capturable, but only in a differentiated section
4. The finding adds material novelty: new pattern, relevant refinement, or correction of an existing entry

When in doubt about autonomous activation, **do not activate**. A false positive disrupts the flow; a false negative can be recovered with the explicit trigger.

---

## Capture process

### Step 1 — Thread scan

Extract from the conversation thread:

| Field | What to look for |
|---|---|
| **Symptom** | The problem as the user described it (unexpected behavior, error, discrepancy) |
| **Context** | Stack, platform, version, environment (GAM 360/non-360, Prebid version, SSP, player) |
| **Diagnosis** | Identified root cause, technical mechanism explained |
| **Resolution** | What change or action resolved it |
| **Evidence level** | Verified / Probable / Requires data (see table below) |
| **Target skill** | Which skill the finding belongs to (see routing table) |

Only ask if the absence of data prevents correct classification or drafting. Otherwise, mark the field as *unconfirmed* and continue.

### Step 1.5 — Eligibility test

Before continuing, verify that the finding meets **at least one** of these criteria:

- Prevents a recurring error or a future re-diagnosis
- Saves troubleshooting time on a non-obvious pattern
- Corrects a frequent mistaken belief
- Documents an undocumented limitation in official sources
- Affects architecture or configuration decisions

If none are met, do not propose capture. This is not reusable knowledge, it is noise.

Also verify that it **generalizes**:
- **Generalizes:** reproducible pattern in similar stacks → capture as a general entry
- **Generalizes with conditions:** valid only in a specific combination (e.g. Prebid 10.x + video + cache) → capture with explicit operating conditions
- **Does not generalize:** too local or client-specific → do not capture, or capture as a *case note* not as an anti-pattern

### Step 2 — Routing to the correct skill and file

Load `references/routing-table.md` to determine destination. Quick summary:

| Finding domain | Target skill | Preferred file |
|---|---|---|
| Line items, trafficking, GAM delivery, yield management | `gam-adops` | `references/anti-patterns.md` |
| Prebid.js, bid adapters, GPT/JS, TCF from Prebid | `prebid-adtech` | `references/anti-patterns.md` |
| VAST, IMA SDK, SSAI/DAI, CTV, players | `video-adtech` | `references/anti-patterns.md` |
| Deal setup SSP-side, OpenRTB, sellers.json | `programmatic-deals` | `references/anti-patterns.md` |
| CMP, TCF string upstream, consent latency | `cmp-consent-flows` | `references/anti-patterns.md` |
| Finding that crosses two skills | Primary skill + mention in the other | Both `anti-patterns.md` |

When routing is unclear, propose to the user and wait for confirmation.

### Step 2.5 — Deduplication

Before drafting, check whether the finding already exists in the target file. Classify:

| Classification | Action |
|---|---|
| **New** | Proceed to draft full entry |
| **Refinement** | Propose expanding or correcting the existing entry, not creating a new one |
| **Duplicate** | Do not propose capture — indicate it is already documented |

If the content of the target file cannot be verified, state this before continuing.

### Step 3 — Draft the entry

Use the **canonical format** defined in `references/entry-format.md`. Never invent a custom format.

Drafting principles:
- Symptom in the user's terms, not rewritten in abstract language
- Root cause with the technical mechanism explained, not just the fix
- Actionable resolution: what to check, what to change, in what order
- Evidence level explicitly labeled
- Minimum context needed for reproducibility

**No-overgeneralization policy:** do not elevate a local case to a general rule without sufficient evidence. If the finding depends on specific conditions (version, player, SSP, wrapper), draft it with those conditions explicit. An anti-pattern with clear conditions is more useful than a false universal rule.

### Step 4 — Approval before writing

**Never write to skill files without explicit approval.**

Present to the user:
1. The drafted entry in canonical format
2. The exact destination: skill + file
3. A direct confirmation question

Only after "yes", "approved", "go ahead", or an unambiguous equivalent, proceed to write.

### Step 5 — Write and confirm

1. Locate the target file in the installed skill
2. Add the entry in the correct section (see `references/entry-format.md` for sections)
3. Confirm to the user: what was written, in which file, in which section

---

## Evidence levels

| Level | Criterion to apply it |
|---|---|
| **Verified** | The user confirmed the solution worked in production, or the behavior is documented in an official source (GAM Help Center, IAB specs, Prebid docs) |
| **Probable** | The diagnosis is consistent with the system's technical logic, but was not validated in the specific case |
| **Requires data** | The finding is plausible but depends on specific configuration that was not shared |

Do not capture entries with level "Insufficient evidence" — if there is no technical basis, it does not belong in an anti-patterns.md.

---

## What NOT to capture

- Standard behaviors already documented in existing skills
- Fixes that are workarounds with no identified root cause
- Findings marked as "Requires data" without the confirming data
- Hypotheses that were not validated
- Client-specific configurations that do not generalize

---

## References — when to load each one

| Reference | When to load it |
|---|---|
| `references/routing-table.md` | Always when determining the destination of a finding — especially for cases that cross skills |
| `references/entry-format.md` | Always before drafting an entry — contains the canonical format and examples by type |

---

## Output format for the user

### Autonomous activation detected

Briefly indicate that a capturable finding was detected and propose the capture without unnecessarily interrupting:

> **[knowledge-capture]** I detected a verified technical finding in this thread. Should I capture it for the `[destination]` skill? I can draft the entry for your review.

### Presentation for approval

```
FINDING DETECTED
─────────────────
Target skill:     [skill name] → [file]
Entry type:       [anti-pattern / verified-fix / behavioral-note / debug-note]
Classification:   [New / Refinement of existing entry]

Capture reason:      [one line — what future error it prevents or what diagnosis it accelerates]
Novelty:             [New entry / Refinement of "[existing entry title]"]
Generalization:      [General / With conditions: specify / Does not generalize: case note]

PROPOSED ENTRY:
[entry in canonical format]

─────────────────
Do you approve this entry as-is, would you like to modify it, or should we discard it?
```

### Confirmation after writing

> ✓ Entry added to `[skill]/references/anti-patterns.md` section `[section name]`.
