# adkit

**AdKit** is a curated toolkit for programmatic advertising professionals — Claude Code skills, structured knowledge bases, and reusable templates covering the full AdTech stack.

Built for ad operations, header bidding engineers, and AdTech consultants working with Prebid.js, Google Ad Manager, video ad serving, CMP/consent frameworks, and programmatic deals.

---

## Repository structure

```
adkit/
├── skills/          # Installable Claude Code skills
├── knowledge/       # Domain knowledge bases by topic
├── templates/       # Reusable templates and frameworks
└── references/      # Reference documents and specs
```

---

## Skills

Installable skills for [Claude Code](https://claude.ai/code). Each skill is a specialised agent mode that activates automatically in the right context.

→ See [skills/README.md](skills/README.md) for installation instructions.

| Skill | Description |
|-------|-------------|
| [broadcaster-onboarding](skills/broadcaster-onboarding/) | Guided onboarding flow for new broadcaster clients on a header bidding stack (GAM setup, CMP, video, deals) |
| [client-context-loader](skills/client-context-loader/) | Loads client-specific context files at session start to avoid repeating briefings |
| [client-deliverable](skills/client-deliverable/) | Standards and checklist for producing client-facing deliverables with professional quality |
| [cmp-consent-flows](skills/cmp-consent-flows/) | TCF/CMP consent flow analysis, debugging, and configuration review |
| [discrepancy-forensics](skills/discrepancy-forensics/) | Systematic root-cause analysis for impression and revenue discrepancies between ad servers and SSPs |
| [fact-checker](skills/fact-checker/) | Verifies AdTech claims against primary sources (Prebid docs, IAB specs, GAM documentation) |
| [incident-response](skills/incident-response/) | Structured incident response protocol for ad serving outages and revenue drops |
| [knowledge-capture](skills/knowledge-capture/) | Captures session learnings into structured knowledge entries for long-term retention |
| [programmatic-deals](skills/programmatic-deals/) | PMP/PG deal setup, troubleshooting, OpenRTB deal objects, and supply chain analysis |
| [video-adtech](skills/video-adtech/) | Video ad serving: VAST/VMAP, IMA SDK, SSAI/DAI, CTV, OpenRTB video, GAM video line items |

---

## Knowledge base

Structured reference documentation by topic. Each folder has an `index.md` overview and individual articles per subtopic.

| Topic | Articles |
|-------|----------|
| [prebid/](knowledge/prebid/) | Prebid core, Amazon APS, AMP, GAM/GPT integration, privacy signals, anti-patterns |
| [video-adtech/](knowledge/video-adtech/) | VAST/VMAP, IMA SDK, SSAI/SGAI/DAI, HLS/DASH, CTV environments, OpenRTB video, GAM video, measurement, QA |
| [gam-adops/](knowledge/gam-adops/) | Line items, inventory, programmatic yield, discrepancy, troubleshooting tools, anti-patterns |
| [programmatic-deals/](knowledge/programmatic-deals/) | OpenRTB deal object, supply chain, anti-patterns |
| [cmp-consent/](knowledge/cmp-consent/) | TCF signal flow, CMP config, consent latency, A/B testing, anti-patterns |
| [context-guardian/](knowledge/context-guardian/) | PII pattern detection, token estimation for AI context management |
| [knowledge-capture/](knowledge/knowledge-capture/) | Entry format and routing table for structured knowledge management |
| [prompt-engineering/](knowledge/prompt-engineering/) | AdTech-specific prompt patterns, killers, and reusable templates |

---

## Templates

| Template | Description |
|----------|-------------|
| [client-context-template.md](templates/client-context-template.md) | Standard template for creating client context files loaded by `client-context-loader` |
| [skill-versioning-system.md](templates/skill-versioning-system.md) | Versioning conventions and update protocol for Claude Code skills |

---

## References

| File | Description |
|------|-------------|
| [VAST_OpenRTB_Signals_CTV_CSAI.xlsx](references/VAST_OpenRTB_Signals_CTV_CSAI.xlsx) | Signal mapping reference: VAST, OpenRTB, CTV, and CSAI ad request parameters |

---

## License

MIT
