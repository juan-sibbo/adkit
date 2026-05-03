# Token Estimation Guide

## Empirical rules by content type

| Content type | Approximate ratio |
|---|---|
| Spanish prose text | ~1 token / 3.5 characters |
| English prose text | ~1 token / 4 characters |
| Source code | ~1 token / 3 characters |
| Structured JSON / XML | ~1 token / 2.5 characters |
| Markdown with tables | ~1 token / 3 characters |
| Long URLs | ~1 token / 2 characters |
| Base64 / binary data | ~1 token / 1.5 characters |

## Estimation by file type

| Format | Typical size | Estimated tokens |
|---|---|---|
| Short email | ~300 words | ~500 tokens |
| 1-page PDF | ~500 words | ~800 tokens |
| 10-page PDF | ~5,000 words | ~8,000 tokens |
| CSV 100 rows × 10 cols | variable | ~2,000-5,000 tokens |
| Python code ~200 lines | ~6,000 chars | ~2,000 tokens |
| Conversation 10 turns | ~1,500 words | ~2,500 tokens |

## System overhead

| Component | Estimated fixed tokens |
|---|---|
| Base system prompt | ~500-2,000 tokens |
| Skill instructions | ~800 tokens |
| Response format | ~50-200 tokens |

## Decision thresholds

```
< 4,000 input tokens   → Green  🟢 Process directly
4,000 - 8,000 tokens   → Yellow 🟡 Notify estimation
8,000 - 20,000 tokens  → Orange 🟠 Request confirmation + suggest chunking
> 20,000 tokens        → Red    🔴 Mandatory split or reject
```

## Chunking strategy for large documents

When the document exceeds the orange threshold:

1. **Semantic chunking**: Split by logical sections (chapters, main paragraphs).
2. **Fixed chunking**: Blocks of ~3,000 tokens with ~200 token overlap.
3. **Map-Reduce**: Summarize each chunk → combine summaries → answer on the aggregate.
4. **Selective extraction**: If the question is specific, extract only the relevant section.

## Relative cost calculation

To help the user decide whether to optimize:

```
Relative cost = (input_tokens × 1) + (output_tokens × 3) + (reasoning_tokens × 2)
```
*(Multipliers reflect the typical relative cost between input, output, and reasoning in modern models)*

If relative cost > 30,000 units → strongly recommend optimization.
