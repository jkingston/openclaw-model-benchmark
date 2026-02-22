# When and How to Refresh This Analysis

## Triggers That Warrant a Refresh

### Immediate Triggers

- **New frontier model release**: A major new model from DeepSeek, Google, Meta, Qwen, Mistral, OpenAI, Anthropic, or Zhipu
- **Pricing change on recommendations**: If any recommended model changes from free to paid (or vice versa)
- **Free tier policy change**: If any provider changes their free tier limits, adds a credit card requirement, or discontinues their free tier
- **Model deprecation**: If a recommended model is deprecated or removed from its provider
- **OpenClaw config schema change**: If OpenClaw adds new model roles or changes its config format

### Periodic Triggers

- **Monthly**: Quick check of [OpenRouter free models](https://openrouter.ai/collections/free-models) and [Chatbot Arena](https://lmarena.ai/) for significant shifts
- **Quarterly**: Full re-analysis following [METHODOLOGY.md](METHODOLOGY.md)

## How to Refresh

### Quick Check (~10 minutes)

1. Visit [OpenRouter free models](https://openrouter.ai/collections/free-models) -- any new free models?
2. Visit [Chatbot Arena](https://lmarena.ai/) -- have rankings shifted significantly?
3. Check each provider's pricing page (links in [ANALYSIS.md](ANALYSIS.md#references)) -- any limit changes?
4. If nothing significant changed, no update needed.

### Full Re-Analysis (~2 hours)

Follow [METHODOLOGY.md](METHODOLOGY.md) from Step 2 onwards:

1. Query `GET https://openrouter.ai/api/v1/models` for the current model catalog
2. Check all provider free tier pages for current limits
3. Gather latest benchmark scores from Chatbot Arena, SWE-Bench, MMLU-Pro
4. Rebuild the comparison matrix
5. Re-run the Pareto analysis
6. Update model assignments if the frontier has shifted
7. Update config snippets
8. Update [ANALYSIS.md](ANALYSIS.md) with new data, date, and references

## What Would Change the Recommendation

| Signal | Current Pick | Would Change To | Threshold |
|---|---|---|---|
| New free model with SWE-Bench >75% | Qwen3 Coder (primary) | The new model | Must be free (`:free` variant) on OpenRouter or direct API |
| Step 3.5 Flash gets Arena Elo >1460 | Qwen3 Coder (primary) | Step 3.5 Flash | Already has SWE-Bench 74.4%; Arena confirmation would make it primary |
| Qwen 3.5 gets free variant | Qwen3 Coder (primary) | Qwen 3.5 | If `:free` variant appears and quality is confirmed |
| Qwen3 Coder removed from free tier | Qwen3 Coder (primary) | Step 3.5 Flash or DeepSeek R1 | Check `:free` variant still exists |
| Groq discontinues free tier | Groq (heartbeat + fallback) | OpenRouter llama-3.1-8b or Step 3.5 Flash | If Groq requires payment |
| Mistral removes Experiment plan | Mistral (fallback anchor) | Cerebras as last fallback | If 1B tokens/month goes away |
| New provider with better free tier | Current chain | Add or replace weakest link | If daily limits exceed current providers |
| Step 3.5 Flash proves unreliable | Step 3.5 Flash (fallback 1) | GPT-OSS 120B or GLM-4.5 Air | After observed failures |
| GLM-4.7 Flash Z.AI tier proves stable | Groq (heartbeat) | Z.AI GLM-4.7 Flash | If latency is consistently <1s |
| DeepSeek V3.2 gets `:free` variant | Qwen3 Coder (primary) | DeepSeek V3.2 | Arena ~1421 overall would make it the top free model |

### Signals That the Current Recommendation is Still Optimal

The recommendation is stable as long as:
- Qwen3 Coder remains available as `qwen/qwen3-coder:free` on OpenRouter
- Step 3.5 Flash remains available as `stepfun/step-3.5-flash:free` on OpenRouter
- Groq, Mistral, and Cerebras all maintain their free tiers
- No free model surpasses SWE-Bench ~75% (which would become the new primary)
- OpenClaw's config schema hasn't changed significantly

### Models to Watch

| Model | Why | When to Re-evaluate |
|---|---|---|
| **Qwen 3.5 Plus** | Claims to beat GPT-5.2. Awaiting Arena confirmation. | When Elo data is published; especially if `:free` variant appears |
| **DeepSeek V3.2** | Arena ~1421 overall but not free. Would be top primary if it gets a `:free` variant. | If OpenRouter adds `:free` variant |
| **Trinity Large Preview** | Free, 131K context. Preview quality. | When it exits preview |
| **Nemotron 3 Super/Ultra** | NVIDIA's upcoming larger models | On release (H1 2026) |
| **Doubao Seed 2.0** (ByteDance) | Strong benchmarks, cheap. Not yet on OpenRouter. | When available on OpenRouter |
| **Devstral 2** (Mistral) | SWE-Bench 72.2%, agentic coding. Free variant may exist but unconfirmed. | When confirmed `:free` on OpenRouter |

### Version History

| Date | Change | Reason |
|---|---|---|
| 2026-02-22 | Switch primary to Qwen3 Coder; correct free model status | DeepSeek V3.2 is NOT free on OpenRouter. Verified all `:free` model IDs via API. |
| 2026-02-21 | Initial analysis | Full evaluation from scratch |
