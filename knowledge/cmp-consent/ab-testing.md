# Consent A/B Testing — Methodology and metrics

Consent banner A/B testing is an area where the revenue impact is significant but experiment design risks are high. A 10% improvement in consent rate can translate into a substantial increase in programmatic fill rate.

---

## Target metrics

### Primary metric (choose one)

| Metric | When to use it |
|---|---|
| **Global consent rate** (% users accepting at least P1-P4) | When the objective is to maximize programmatic fill |
| **Consent rate per purpose** (% users accepting each purpose) | When there are specific purposes blocking high-value vendors |
| **Revenue per session for new users** | When the objective is direct business impact, not consent rate |

### Secondary metrics (always measure)

- **Bounce rate after banner** — a more aggressive banner may improve consent rate but increase abandonment
- **Time to banner interaction** — indicates perceived friction
- **% users choosing granular configuration** vs accept/reject all — indicates sophistication level
- **Consent rate by device** — mobile and desktop can react very differently to the same design

---

## Variables to test — by typical impact

### High impact
- **Main button copy** — "Accept all" vs "Got it" vs "Continue" can move consent rate by 5-15%
- **"Reject" button prominence** — reducing its visibility improves consent rate but is a dark pattern in GDPR (regulatory risk)
- **Structure: accept-all vs granular as first option** — showing granular first reduces overall consent but improves consent on specific purposes

### Medium impact
- **Number of declared purposes** — fewer purposes = less friction = higher consent rate
- **Banner body copy** — technical language vs plain language
- **Banner position** — bottom bar vs modal vs top bar

### Low impact (but easy to test)
- **Colors and design** — rarely moves more than 2-3%
- **Automatic vs forced language** — impact on multi-language traffic

---

## Experiment design

### Minimum validity criteria

1. **Correct randomization** — A/B assignment must be per user (persistent), not per session or per pageview. A user must always see the same variant.

2. **Clean segmentation** — separate analysis by:
   - New vs returning (returning users have prior consent — they will not see the banner)
   - Jurisdiction (GDPR vs non-GDPR)
   - Device (mobile vs desktop)

3. **Sufficient sample size** — to detect differences of 5% in consent rate with 80% statistical power, ~3,000–5,000 new users per variant are needed. Calculate before launching.

4. **Minimum duration** — at least 2 full weeks to capture day/week variation. Mondays and Sundays behave differently.

5. **One variable per experiment** — do not change copy + design + structure simultaneously. If the experiment wins, you will not know what worked.

### Experiment design anti-patterns

**Split by pageview instead of by user**
The same user can see variant A on one visit and variant B on another. This contaminates the data and can generate users who interact with both variants.

**Not excluding returning users with prior consent**
These users do not see the banner — including them in the analysis dilutes the real effect of the experiment.

**Only measuring consent rate, not revenue impact**
A banner that improves consent rate but increases bounce rate can be negative in terms of revenue. Always correlate with fill rate and RPM.

**Stopping the experiment too early (peeking)**
If the experiment shows positive results after 3 days, the temptation is to stop it. Doing so before the calculated sample size produces false positives. Use Bonferroni correction or Bayesian methodology if continuous monitoring is needed.

---

## Expected impact on the programmatic ecosystem

The relationship between consent rate and revenue is not linear. Purposes have different weights:

| Consent scenario | Estimated impact on programmatic CPM |
|---|---|
| P1+P2+P3+P4 granted | Baseline — 100% |
| P1+P2 only | -30% to -50% (premium DSPs do not bid without P3/P4) |
| P1 only | -70% to -80% (non-personalized advertising only) |
| No consent (NPA) | -85% to -95% (non-personalized demand only) |

These figures are indicative and vary significantly by vertical, audience, and demand mix. Measure the real impact in your own ecosystem before making configuration decisions based on third-party benchmarks.
