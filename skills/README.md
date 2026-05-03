# Skills

Claude Code skills that activate automatically in the right context. Each skill folder contains:

- **README.md** — description, trigger conditions, and usage
- **\*.skill.zip** — the installable package

---

## How to install a skill

### Option A — Claude Code CLI (recommended)

```bash
claude skills install <path-to-skill.zip>
```

Example:
```bash
claude skills install skills/video-adtech/video-adtech.skill.zip
```

### Option B — Manual installation

1. Unzip the `.skill.zip` file
2. Place the contents in your Claude Code skills directory (`~/.claude/skills/` or the project `.claude/skills/`)
3. Restart Claude Code

---

## Available skills

| Skill | Zip | Triggers |
|-------|-----|---------|
| [broadcaster-onboarding](broadcaster-onboarding/) | broadcaster-onboarding.skill.zip | "new client", "onboarding for [TV]", regional broadcaster name |
| [client-context-loader](client-context-loader/) | client-context-loader.skill.zip | Session start with a known client |
| [client-deliverable](client-deliverable/) | client-deliverable.skill.zip | "prepare deliverable", "client report", "write up for client" |
| [cmp-consent-flows](cmp-consent-flows/) | cmp-consent-flows.skill.zip | TCF, CMP, consent string, GDPR, Didomi, OneTrust |
| [discrepancy-forensics](discrepancy-forensics/) | discrepancy-forensics.skill.zip | "discrepancy", "impression gap", "revenue difference" |
| [fact-checker](fact-checker/) | fact-checker-v1.1.0.skill.zip | "is this correct", "verify", "check against docs" |
| [incident-response](incident-response/) | incident-response.skill.zip | "revenue drop", "ads not serving", "outage", "incident" |
| [knowledge-capture](knowledge-capture/) | knowledge-capture.skill.zip | "capture this", "save to wiki", "log this finding" |
| [programmatic-deals](programmatic-deals/) | programmatic-deals.skill.zip | PMP, PG, deal ID, deal object, supply chain |
| [video-adtech](video-adtech/) | video-adtech.skill.zip | VAST, VMAP, IMA SDK, SSAI, DAI, CTV, ad pod |

---

## Versioning

Skills follow the versioning conventions described in [templates/skill-versioning-system.md](../templates/skill-versioning-system.md).
