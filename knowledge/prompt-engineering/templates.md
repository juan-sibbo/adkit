# Prompt Templates Reference

Template library for Prompt Master. Read only the one that applies to the task. Do not load all of them at once.

## Table of contents

| Template | Best for |
|---|---|
| A — RTF | Simple one-shot tasks |
| B — CO-STAR | Professional documents, business writing |
| C — RISEN | Complex multi-step projects |
| D — CRISPE | Creative, brand voice |
| E — Chain of Thought | Logic, math, analysis, debugging |
| F — Few-Shot | Consistent structured output |
| G — File-Scope | Cursor, Windsurf, Copilot — IDE coding |
| H — ReAct + Stop Conditions | Claude Code, Codex, Devin — autonomous agents |
| I — Visual Descriptor | Image (Midjourney, DALL-E, SD) — occasional use |
| J — Prompt Decompiler | Disassemble, adapt, split existing prompts |
| **K — AdTech Diagnostic** | **Debug Prebid / GAM / VAST / deals / CMP** |
| **L — AdTech Code Review** | **Review publisher-side implementations by third parties** |
| **M — Knowledge Capture Prep** | **Prepare a finding for the knowledge-capture skill** |
| N — Web agents | Perplexity Computer, Comet, Atlas, Manus, Devin |
| O — Workflow automation | Zapier, Make, n8n |

---

## Template A — RTF

Role, Task, Format. For one-shot tasks where the request is clear and simple.

```
Role: [One sentence defining who the AI is]
Task: [Precise verb + what to produce]
Format: [Exact format and length of the output]
```

---

## Template B — CO-STAR

Context, Objective, Style, Tone, Audience, Response. For professional documents, business writing, reports, marketing where full context control matters.

```
Context: [Background needed to understand the situation]
Objective: [Exact objective — what success looks like]
Style: [Style: formal / conversational / technical / narrative]
Tone: [Emotional register: authority / empathy / urgency / neutral]
Audience: [Who reads it — knowledge level and expectations]
Response: [Format, length, output structure]
```

---

## Template C — RISEN

Role, Instructions, Steps, End Goal, Narrowing. For complex multi-step projects, clear sequences of actions.

```
Role: [Expert identity]
Instructions: [General task in clear terms]
Steps:
  1. [First action]
  2. [Second action]
  3. [Continue as needed]
End Goal: [What the final output must achieve]
Narrowing: [Constraints, scope limits, what to exclude]
```

---

## Template D — CRISPE

Capacity, Role, Insight, Statement, Personality, Experiment. For creative, brand voice, any task where personality, tone, and iteration matter.

```
Capacity: [What capacity/expertise the AI should have]
Role: [Specific persona to adopt]
Insight: [Key background that shapes the response]
Statement: [The central task or question]
Personality: [Tone and style — witty / authoritative / casual / sharp]
Experiment: [Ask for variants or alternatives]
```

---

## Template E — Chain of Thought

For logic, math, debugging, multi-factor analysis where the AI must reason carefully before committing.

**Important:** Only use CoT on standard reasoning models (Claude Sonnet/Opus, GPT-5.x, Gemini, Qwen2.5, Llama). **NEVER** on o3, o4-mini, GPT-5 thinking, DeepSeek-R1, Qwen3 thinking — it degrades the output.

```
[Task statement]

Before responding, think this through carefully:
<thinking>
1. What is the real problem being asked?
2. What constraints must the solution respect?
3. What approaches are possible?
4. Which is best and why?
</thinking>

Give your final answer only in <answer> tags.
```

---

## Template F — Few-Shot

When format is better shown than described.

```
[Task instruction]

Examples of the exact format:

<examples>
  <example>
    <input>[example input 1]</input>
    <o>[example output 1]</o>
  </example>
  <example>
    <input>[example input 2]</input>
    <o>[example output 2]</o>
  </example>
</examples>

Apply this exact pattern to: [real input]
```

Rules:
- 2-5 examples is the sweet spot
- Include edge cases, not just easy ones
- Use XML tags — Claude parses them well
- If you have re-prompted for the same format issue twice, switch to few-shot

---

## Template G — File-Scope

For Cursor, Windsurf, GitHub Copilot — IDE AI that edits code in a codebase.

```
File: [exact/path/to/file.ext]
Function/Component: [exact name]

Current Behavior:
[What it does now — be specific]

Desired Change:
[What it should do after — be specific]

Scope:
Modify only [function / component / section].
DO NOT touch: [list of what must stay the same]

Constraints:
- Language/framework: [specific version]
- Do not add dependencies not listed in [package.json / requirements.txt]
- Preserve [type signatures / API contracts / variable names]

Done When:
[Exact condition confirming the change]
```

---

## Template H — ReAct + Stop Conditions

For Claude Code, Codex, Devin, AutoGPT — autonomous agents. Runaway loops and scope explosion are the biggest credit-killers. Stop conditions are **not optional**.

```
Objective:
[Single unambiguous objective in one sentence]

Starting State:
[Current file structure / codebase state / environment]

Target State:
[What must exist when the agent finishes]

Allowed Actions:
- [Specific permitted action]
- Install only packages listed in [requirements.txt / package.json]

Forbidden Actions:
- DO NOT modify files outside [directory/scope]
- DO NOT start dev server or deploy
- DO NOT push to git
- DO NOT delete files without showing diff first
- DO NOT make architecture decisions without human approval

Stop Conditions:
Stop and ask for review when:
- A file is about to be permanently deleted
- A new external service/API needs to be integrated
- There are two valid implementation paths affecting architecture
- An error is not resolved in 2 attempts
- The task requires changes outside the declared scope

Checkpoints:
After each major step: ✅ [what was completed]
At the end: full summary of every changed file.
```

**Codex note:** More literal than Claude Code. Add *"Write a test before implementing. Done when the test passes."* if possible.

---

## Template I — Visual Descriptor

For Midjourney, DALL-E, Stable Diffusion when needed. Occasional use in this profile.

```
Subject: [Main subject — specific]
Action/Pose: [What it does]
Setting: [Where]
Style: [photorealistic / cinematic / vector / etc.]
Mood: [dramatic / serene / etc.]
Lighting: [golden hour / studio / neon / etc.]
Color Palette: [dominant colors]
Composition: [wide / close-up / aerial / etc.]
Aspect Ratio: [16:9 / 1:1 / 9:16 / 4:3]
Negative Prompts: [blurry, watermark, extra fingers, low quality]
```

Platform-specific syntax:
- **Midjourney:** descriptors separated by commas, not prose. `--ar`, `--style`, `--v 6` at the end
- **Stable Diffusion:** `(word:1.3)` syntax. CFG 7-12. Negative prompt mandatory
- **DALL-E 3:** prose works. Add *"no text in the image"* unless you want it

---

## Template J — Prompt Decompiler

When the user pastes an existing prompt and wants to disassemble, adapt, simplify, or split it. This is analysis and adaptation, not building from scratch.

Detect which task:
- **Break down** — explain what each part does
- **Adapt** — rewrite for another tool preserving intent
- **Simplify** — remove redundancy without losing meaning
- **Split** — divide a complex one-shot into a clean sequence

For Adapt, always ask: *"What tool is the original from and which are you adapting it to?"*

**Break down:**
```
Original prompt: [paste]

Structure analysis:
- Role/Identity: [what role is assigned and why]
- Task: [what action is requested]
- Constraints: [what limits there are]
- Format: [what form is expected]
- Weaknesses: [what is missing or could cause incorrect output]

Recommended fix: [rewritten version with gaps filled]
```

**Adapt:**
```
Original ([source tool]): [original prompt]

Adapted for [target tool]:
[rewritten prompt with target tool's syntax and best practices]

Key changes:
- [change 1 and why]
- [change 2 and why]
```

**Split:**
```
Original prompt: [paste]

This prompt does [N] things. Splitting into [N] sequential prompts:

Prompt 1 — [what it covers]:
[block]

Prompt 2 — [what it covers]:
[block]

Execute in order. Each output feeds the next.
```

---

## Template K — AdTech Diagnostic ⚡ NEW

For debugging prompts on Prebid, GAM, VAST, deals, CMP. The objective is to force structured diagnosis instead of generic responses.

```
Role: You are a senior AdOps Engineer with 10+ years at broadcaster publishers.
You master: Prebid.js, GAM 360, TCF v2.2, GPP, OpenRTB, VAST/VMAP, SSAI/DAI, IMA SDK.

Observed problem:
[concrete symptom, not emotional description]

Technical environment:
- Stack: [Prebid version + loaded modules / GAM 360 or non-360 / CMP vendor]
- Domain/app: [URL or identifier]
- Consent: [TCF v2.2 / GPP / jurisdiction]
- Device/browser: [if applicable]
- Inventory type: [display / video instream / outstream / CTV / HbbTV / FAST]
- [if video] Ad architecture: [client-side / SSAI / SGAI / DAI]
- [if deal] Seat ID, DSP, SSP, Deal ID, type (PG / PD / PA)

Already verified (rule these out):
- [check 1]
- [check 2]

Diagnose following this funnel (in this order, do not skip steps):
[choose the applicable funnel — see references/adtech-patterns.md]

For each layer:
1. What to check (with the native tool: Ads Inspector, Publisher Console, SSP UI, DevTools, CMP debug)
2. What result you expect if OK
3. What it indicates if KO
4. Next step based on the result

Constraints:
- Do not assume unverifiable data. If something cannot be checked with the given info, say so explicitly.
- If the symptom admits multiple causes, list them in order of probability with reasoning.
- Do not invent Prebid module names, GAM flags, SSP endpoints, or IDs. If uncertain, write [verify].

Output:
- Structured layer-by-layer diagnosis
- Most probable cause at the end, with confidence level (high/medium/low)
- Next 3 concrete steps to execute
```

---

## Template L — AdTech Code Review ⚡ NEW

For reviewing publisher-side implementations by third parties (other devs, AI tools, agencies) in Prebid, GAM, video, CMP.

```
Role: You are a senior AdOps Engineer specialized in [Prebid / GAM / video-adtech / CMP].
You review publisher-side code by third parties looking for anti-patterns, credit-killers, and risks.

Code to review:
[paste the code — or indicate file/function if it is in a repo]

Context:
- Original author: [human / specific AI / unknown]
- Declared objective: [what they said it does]
- Production environment: [publisher, approx traffic, inventory type]

Review looking for:
1. Known domain anti-patterns (consult the relevant skill: prebid-adtech / gam-adops / video-adtech / cmp-consent-flows)
2. Race conditions between consent (CMP) and auction (Prebid) — does the auction start before the TC string resolves?
3. Misconfigured timeouts — Prebid timeout vs GAM failsafe vs network
4. Leaks: pbjs.adUnits not destroyed in SPA, setTargetingForGPTAsync() before pbjs resolves, double CMP init
5. Broken signals — PPID, PPS, secure signals, user ID modules misconfigured
6. Consent compliance — undeclared vendors, misconfigured purposes, GPP string absent in applicable jurisdictions
7. [video] VAST handling, ad pods, SCTE-35, IMA vs PAL, fill rate degradation in CTV
8. Non-obvious revenue/fill risks

Output format:
## 🚨 Critical (fix before production)
- [issue] — [line/file] — [impact] — [proposed fix]

## ⚠️ High (fix in next iteration)
- [...]

## 💡 Improvements (non-blocking)
- [...]

## ✅ What is correct
- [verified things that work]

Constraints:
- Do not invent issues. If you have no evidence, do not list it.
- If an issue depends on external config not visible (GAM UI, SSP config, CMP config), say so: "Depends on verifying in [where]".
- Confidence level per issue: high/medium/low.
```

---

## Template M — Knowledge Capture Prep ⚡ NEW

When the user wants to prepare a verified finding from a conversation for the `knowledge-capture` skill. This template produces the **ready-to-paste text** for the target skill (gam-adops, prebid-adtech, video-adtech, programmatic-deals, cmp-consent-flows).

```
Role: You are a technical knowledge curator for an AdTech skills ecosystem.
Your job: convert a conversation finding into a structured entry,
ready to paste into the corresponding target skill.

Finding to capture:
[summary of the finding]

Evidence (from the conversation thread):
- Initial symptom: [what was observed]
- Diagnostic process: [steps that led to the cause]
- Verified root cause: [what it actually was]
- Applied fix: [what was done]
- Fix verification: [how it was confirmed to work]

Produce:

1. **Target skill** — one of: gam-adops / prebid-adtech / video-adtech / programmatic-deals / cmp-consent-flows
   Reason: [why that skill]

2. **Entry type** — one of: anti-pattern / known bug / undocumented behavior / diagnostic checklist / verified fix

3. **Structured entry** (ready to paste):

```
### [Short and searchable title of the finding]

**Context:** [environment where it appears]
**Symptom:** [how it manifests]
**Cause:** [what produces it]
**Fix:** [concrete solution]
**Verification:** [how to confirm the fix works]
**Date:** [finding date]
**Confidence:** [high/medium — only high if there is direct evidence]
```

4. **Where to insert it** — specific section of the target skill (e.g. "within the anti-patterns block", "at the end of the troubleshooting funnel")

5. **Do not write anything in the skill until the user explicitly approves the text.**
```

---

## Template N — Web agents

For Perplexity Computer/Comet, OpenAI Atlas, Claude in Chrome, Manus, Devin. They control a real browser — they click, navigate, complete transactions.

```
Objective:
[specific outcome — do not describe the steps]

Constraints:
- [hard constraint 1]
- [hard constraint 2]

Permissions:
- Research, read, compare: YES
- Fill in forms: [YES with confirmation / NO]
- Complete transactions: NO
- Send messages: NO
- Modify accounts: NO

Stop and ask before:
- Submitting any form
- Completing any transaction
- Sending any message
- Any irreversible action

Output format:
[structure of the final deliverable]
```

---

## Template O — Workflow automation

For Zapier, Make, n8n.

```
Trigger:
- App: [name]
- Event: [what fires it]
- Auth: assume [app] connected

Steps:
1. [Step 1 — app, action, field mapping]
2. [Step 2 — with what data from step 1 it enters]
3. [...]

Final action:
- App: [destination]
- Action: [what it does]
- Fields: [data mapping]

Error handling:
- If [app] fails: [fallback]
```
