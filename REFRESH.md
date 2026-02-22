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
| New free model with >1400 Arena Elo | DeepSeek V3.2 (primary) | The new model | Must be free and available on OpenRouter or direct API |
| Qwen 3.5 confirmed Arena Elo >1420 | DeepSeek V3.2 (primary) | Qwen 3.5 | When Arena data is available |
| DeepSeek V3.2 removed from free tier | DeepSeek V3.2 (primary) | Qwen3-Coder 480B or MiMo-V2-Flash | If paid cost exceeds $1/M output |
| Groq discontinues free tier | Groq (heartbeat + fallback) | OpenRouter llama-3.1-8b or Step 3.5 Flash | If Groq requires payment |
| Mistral removes Experiment plan | Mistral (fallback anchor) | Cerebras as last fallback | If 1B tokens/month goes away |
| New provider with better free tier | Current chain | Add or replace weakest link | If daily limits exceed current providers |
| Step 3.5 Flash proves unreliable | Step 3.5 Flash (fallback) | GLM-4.5 Air or Trinity Large Preview | After observed failures |
| GLM-4.7 Flash Z.AI tier proves stable | Groq (heartbeat) | Z.AI GLM-4.7 Flash | If latency is consistently <1s |

### Signals That the Current Recommendation is Still Optimal

The recommendation is stable as long as:
- DeepSeek V3.2 remains top-5 on Chatbot Arena and available free/cheap on OpenRouter
- Groq, Mistral, and Cerebras all maintain their free tiers
- No free model surpasses Arena Elo ~1420 (which would become the new primary)
- OpenClaw's config schema hasn't changed significantly

### Models to Watch

| Model | Why | When to Re-evaluate |
|---|---|---|
| **Qwen 3.5 Plus** | Claims to beat GPT-5.2. Awaiting Arena confirmation. | When Elo data is published |
| **GLM-4.5 Air** | Free on OpenRouter, agentic-focused. Unproven. | After community reports on quality |
| **Trinity Large Preview** | Free, 512K context. Preview quality. | When it exits preview |
| **Nemotron 3 Super/Ultra** | NVIDIA's upcoming larger models | On release (H1 2026) |
| **Doubao Seed 2.0** (ByteDance) | Strong benchmarks, cheap. Not yet on OpenRouter. | When available on OpenRouter |

### Version History

| Date | Change | Reason |
|---|---|---|
| 2026-02-21 | Initial analysis | Full evaluation from scratch |
