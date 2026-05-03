# Credit-Killing Patterns Reference

Patterns that waste tokens and cause re-prompts. Read this file when the user pastes a bad prompt and asks for a fix, or when diagnosing why a prompt is underperforming.

## Task Patterns

| # | Pattern | Bad | Fixed |
|---|---|---|---|
| 1 | Vague verb | "help me with my code" | "Refactor `getUserData()` to use async/await and handle null returns" |
| 2 | Two tasks in one prompt | "explain AND rewrite this function" | Two prompts: first explain, then rewrite |
| 3 | No success criteria | "make it better" | "Done when the function passes existing unit tests and handles null without throwing" |
| 4 | Over-permissive agent | "do whatever it takes" | Explicit allowed actions + explicit forbidden actions |
| 5 | Emotional description | "it's totally broken, fix everything" | "Throws uncaught TypeError at line 43 when user is null" |
| 6 | "Build everything" | "build my entire app" | Split: Prompt 1 (scaffold), Prompt 2 (core feature), Prompt 3 (polish) |
| 7 | Implicit reference | "now add the other thing we talked about" | Always restate the full task |

## Context Patterns

| # | Pattern | Bad | Fixed |
|---|---|---|---|
| 8 | Assumed prior knowledge | "continue where we left off" | Include Memory Block with prior decisions |
| 9 | No project context | "write a cover letter" | "PM role at B2B fintech, 2yr SWE transitioning to product, 3 features as tech lead" |
| 10 | Stack forgotten | New prompt contradicts previous tech choice | Memory Block with established stack always |
| 11 | Invites hallucination | "what do experts say about X?" | "Only cite sources you are confident about. If uncertain, say so explicitly." |
| 12 | Undefined audience | "write something for users" | "Non-technical B2B buyers, no coding knowledge, decision-maker level" |
| 13 | No mention of prior attempts | (empty) | "I already tried X and it failed because of Y. Do not suggest X." |

## Format Patterns

| # | Pattern | Bad | Fixed |
|---|---|---|---|
| 14 | No output format | "explain this concept" | "3 bullets, each under 20 words, with a summary sentence at the start" |
| 15 | Implicit length | "write a summary" | "Summary in exactly 3 sentences" |
| 16 | No role assigned | (empty) | "You are a senior AdOps Engineer specialized in Prebid and GAM 360" |
| 17 | Vague aesthetic adjective | "make it look professional" | "Monochrome palette, 16px base font, 24px line-height, no decorative elements" |
| 18 | No negative prompts in image | "a portrait of a woman" | Add: "no watermark, no blur, no extra fingers, no distortion, no text overlay" |
| 19 | Prose for Midjourney | Full descriptive sentence | "subject, style, mood, lighting, composition, --ar 16:9 --v 6" |

## Scope Patterns

| # | Pattern | Bad | Fixed |
|---|---|---|---|
| 20 | No scope boundary | "fix my app" | "Fix only the login form validation in `src/auth.js`. Do not touch anything else." |
| 21 | No stack constraints | "build a React component" | "React 18, TypeScript strict, no external libraries, Tailwind only" |
| 22 | No stop condition for agents | "build the whole feature" | Explicit stop conditions + ✅ checkpoint after each step |
| 23 | No file path for IDE AI | "update the login function" | "Update `handleLogin()` in `src/pages/Login.tsx` only" |
| 24 | Wrong template for the tool | GPT-style prompt in Cursor | Adapt to Template G (File-Scope) |
| 25 | Paste entire codebase | Full repo context in every prompt | Scope to the relevant function/file |

## Reasoning Patterns

| # | Pattern | Bad | Fixed |
|---|---|---|---|
| 26 | No CoT on logic task | "which approach is better?" | "Think through both approaches step by step before recommending" |
| 27 | CoT on reasoning models | "think step by step" to o3/o4-mini/GPT-5 thinking/R1/Qwen3-thinking | Remove it — they reason internally, CoT degrades output |
| 28 | Expectation of inter-session memory | "you already know my project" | Re-provide Memory Block in every new session |
| 29 | Contradicts prior work | New prompt ignores prior architecture | Memory Block with all established decisions |
| 30 | No grounding on factual | "summarize what experts say about X" | "Use only information you are confident about. Mark [uncertain] if not." |

## Agentic Patterns

| # | Pattern | Bad | Fixed |
|---|---|---|---|
| 31 | No starting state | "build me a REST API" | "Empty Node.js project, Express installed, `src/app.js` exists" |
| 32 | No target state | "add authentication" | "`src/middleware/auth.js` with JWT verify. POST `/login` and POST `/register` in `src/routes/auth.js`" |
| 33 | Silent agent | No progress output | "After each step: ✅ [what was completed]" |
| 34 | Filesystem without lock | No file restrictions | "Only edit inside `src/`. Do not touch `package.json`, `.env`, or config." |
| 35 | No human review trigger | Agent decides everything autonomously | "Stop and ask before: deleting files, adding a dependency, touching DB schema, git push" |

## AdTech-specific Killers ⚡ NEW

| # | Pattern | Bad | Fixed |
|---|---|---|---|
| 36 | Prebid without version or modules | "why isn't X bidder bidding?" | "Prebid 9.x, modules [consentManagement, gdprEnforcement, currency, priceFloors], bidder [X] in version [Y]" |
| 37 | VAST without distinguishing architecture | "VAST error in video" | "VAST error, SSAI architecture with Google DAI, HLS manifest, IMA SDK absent (SDK-less with PAL nonce)" |
| 38 | Deal without identifiers | "the deal isn't scaling" | "Deal ID [X], SSP [Y], PG type, buyer seat [Z] in DSP [W]" |
| 39 | CMP without framework/jurisdiction | "consent isn't working" | "Didomi + TCF v2.2, EEA jurisdiction, GPP not active" |
| 40 | GAM without distinguishing 360 vs non-360 | "GAM not delivering" | "GAM 360, line item [type], issue with [trafficking/forecasting/yield]" |
| 41 | Symptom as cause | "no revenue" treated as root cause | "No revenue" is a symptom. Real layer: avails / listening / bidding / delivery |
| 42 | Mixed publisher/buyer POV | Diagnosis mixing SSP and DSP data without distinguishing | Explicitly declare which side you are analyzing from — SSP wins ≠ GAM impressions ≠ DSP billed |
| 43 | "Expert in everything" without domain | "You are an adtech expert" | "You are a senior AdOps Engineer specialized in [Prebid.js / GAM 360 / video-adtech / CMP]" |
| 44 | Code pasted without context | Prebid config snippet without env | "Production / staging / local, approx [X] QPS traffic, inventory type [Y]" |
| 45 | CoT on AdTech reasoning-native diagnosis | "think step by step" to GPT-5 thinking for diagnosis | Remove it. Structure the funnel directly (Funnel 1-6 in adtech-patterns.md) |

## Meta-patterns of prompt engineering

| # | Pattern | Signal | Fix |
|---|---|---|---|
| 46 | Framework-stuffing | User packs CO-STAR + RISEN + CoT + Few-Shot into one prompt | Pick ONE template. Frameworks do not stack. |
| 47 | Over-roleplay | "You are Steve Jobs + Einstein + a Michelin-star chef" | One specific role coherent with the domain |
| 48 | Constraint inflation | 50 constraints, many redundant | Top 5-7 critical ones, the rest are inferred from the role |
| 49 | Infinite prompt-within-prompt | "Write a prompt that writes a prompt that..." | Collapse to one level. If two levels are needed, split into a sequence. |
| 50 | Ceremonial preamble | "Please, if you don't mind, would you be so kind as to..." | Direct. Models do not respond better to politeness, only to clarity. |
