# Methodology: How to Build This Analysis From Scratch

This document describes the repeatable process for evaluating LLM models and producing an optimal tiering recommendation for OpenClaw.

## Prerequisites

- Familiarity with the [OpenClaw configuration reference](https://github.com/openclaw/openclaw/blob/main/docs/gateway/configuration-reference.md)
- Access to the internet for benchmark data and pricing lookups

## Step 1: Understand OpenClaw's Model Roles

Read the OpenClaw docs to identify every config key that accepts a model:

| Config Key | Purpose | Optimise For |
|---|---|---|
| `agents.defaults.model.primary` | Main interactive model | Quality |
| `agents.defaults.model.fallbacks` | Ordered failover chain | Breadth, reliability |
| `agents.defaults.heartbeat.model` | Periodic background checks | Cost, speed |
| `agents.defaults.subagents.model` | Background sub-tasks | Quality/cost balance |
| `agents.defaults.imageModel.primary` | Vision/image input | Multimodal capability |

Key docs:
- [Model concepts](https://github.com/openclaw/openclaw/blob/main/docs/concepts/models.md) -- selection hierarchy, allowlist, aliases
- [Model failover](https://github.com/openclaw/openclaw/blob/main/docs/concepts/model-failover.md) -- auth profile rotation, backoff, how fallbacks are triggered
- [Configuration reference](https://github.com/openclaw/openclaw/blob/main/docs/gateway/configuration-reference.md) -- full schema

Note: OpenClaw does **not** have a formal `model_tiers` config key. Tiering is achieved by assigning different models to `primary`, `heartbeat.model`, `subagents.model`, and `fallbacks`.

## Step 2: Discover Available Models on OpenRouter

OpenRouter is the primary provider. Query their model catalog:

```bash
curl -s https://openrouter.ai/api/v1/models | jq '.data | length'
```

For each model, note:
- `id` (e.g., `deepseek/deepseek-v3.2`)
- `pricing.prompt` and `pricing.completion` (cost per token)
- `context_length`
- `architecture.modality` (text, multimodal)
- Whether it appears in the [free models collection](https://openrouter.ai/collections/free-models)

Filter candidates:
- **Free models**: Include all (the primary value proposition)
- **Ultra-cheap** (<$0.30/M output): Include all
- **Cheap** ($0.30-$2/M): Include top 10 by recency/popularity
- **Mid-range** ($2-$10/M): Include top 3 as quality baselines
- **Premium** (>$10/M): Exclude unless explicitly needed

## Step 3: Research Direct-Provider Free Tiers

Many providers offer their own free API tiers with independent rate limits. This is critical for the multi-provider failover strategy.

Check each provider's pricing/docs page:

| Provider | Where to Check | Key Detail |
|---|---|---|
| Google Gemini | [ai.google.dev/gemini-api/docs/pricing](https://ai.google.dev/gemini-api/docs/pricing) | Per-project quotas; multiple projects = multiplied limits |
| Groq | [console.groq.com/docs/rate-limits](https://console.groq.com/docs/rate-limits) | Very fast inference, generous RPD |
| Mistral | [docs.mistral.ai/deployment/ai-studio/tier](https://docs.mistral.ai/deployment/ai-studio/tier) | 1B tokens/month on Experiment plan |
| Cerebras | [cerebras.ai/pricing](https://www.cerebras.ai/pricing) | 1M tokens/day, fast inference |
| xAI | [x.ai/api](https://x.ai/api) | $25 signup credits + $150/mo with data sharing |
| Z.AI (Zhipu) | [docs.z.ai/guides/overview/pricing](https://docs.z.ai/guides/overview/pricing) | Free tier for GLM-4.7 Flash (1 concurrency) |
| OpenRouter | [openrouter.ai/docs/api/reference/limits](https://openrouter.ai/docs/api/reference/limits) | 50 RPD free, 1000 RPD after $10 purchase |

For each, record:
- Rate limits (RPM, RPD, TPM, TPD)
- Available models
- Whether a credit card is required
- Whether limits are per-key, per-project, or per-account
- Terms around multiple accounts

## Step 4: Gather Public Benchmark Data

Consult these benchmark sources (in priority order):

1. **[Chatbot Arena (LMSYS)](https://lmarena.ai/)** -- Human preference rankings (Elo scores). The single most reliable quality signal.
2. **[LiveBench](https://livebench.ai/)** -- Contamination-resistant, regularly updated
3. **[SWE-Bench Verified](https://www.swebench.com/)** -- Real-world coding ability (critical for OpenClaw's coding use case)
4. **[MMLU-Pro](https://huggingface.co/spaces/TIGER-Lab/MMLU-Pro)** -- Broad knowledge and reasoning
5. **[GPQA Diamond](https://paperswithcode.com/dataset/gpqa)** -- Graduate-level reasoning
6. **[HumanEval / HumanEval+](https://paperswithcode.com/dataset/humaneval)** -- Code generation
7. **[Artificial Analysis](https://artificialanalysis.ai/)** -- Speed, throughput, and pricing comparisons

For each candidate model, record:
- Arena Elo (if available)
- SWE-Bench Verified score
- MMLU-Pro score
- GPQA Diamond score
- Median latency / throughput

## Step 5: Build the Comparison Matrix

Create a table with one row per candidate model and columns:

| Model | Arena Elo | SWE-Bench | MMLU-Pro | Cost (in/out $/M) | Free? | Latency | Context |
|---|---|---|---|---|---|---|---|

## Step 6: Identify the Pareto Frontier

A model is **Pareto-optimal** if no other model is strictly better on ALL of quality, speed, and cost simultaneously.

For each candidate, ask: "Is there another model that is better quality AND cheaper AND faster?" If yes, the candidate is dominated and can be eliminated. The remaining models form the Pareto frontier -- these are the only ones worth considering.

## Step 7: Assign Models to OpenClaw Roles

Map Pareto-optimal models to roles based on each role's optimisation target:

- **Primary**: Highest quality on the Pareto frontier. Prefer models with strong coding (SWE-Bench) and general chat (Arena Elo) scores.
- **Fallbacks**: 2-3 other Pareto-optimal models with different providers/strengths. Order by composite quality score descending. Ensure provider diversity for resilience.
- **Heartbeat**: Cheapest and fastest model with quality >= 0.5. This runs every 30min so cost/speed dominate.
- **Subagents**: Best quality-to-cost ratio. These run in the background so latency matters less, but quality matters since they handle real tasks.
- **Image model**: Best multimodal model available. Prefer free.

## Step 8: Design the Multi-Provider Failover Chain

The goal is to maximise free usage by chaining providers so that when one's rate limit is hit, OpenClaw automatically fails over to the next.

Principles:
1. Put the highest-quality free provider first
2. Order remaining providers by quality descending
3. Ensure each provider in the chain has independent rate limits (different providers, or different Google Cloud projects)
4. Calculate the combined daily/monthly capacity

## Step 9: Produce the Config Snippets

Generate two outputs:
1. **`openclaw.json`** -- The full config matching [OpenClaw's schema](https://github.com/openclaw/openclaw/blob/main/docs/gateway/configuration-reference.md)
2. **`default.nix` snippet** -- The nix-claw `initialConfig` block for the deployment at `/Users/jack/workspace/nix-claw/hosts/claw/default.nix`

## Step 10: Document the Analysis

Write up the full analysis with:
- The comparison matrix with source links
- The Pareto frontier
- The role assignments with rationale
- The free tier capacity estimates
- The config snippets
- References for every data point
