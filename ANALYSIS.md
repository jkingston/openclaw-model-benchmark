# Model Tiering Analysis -- February 2026

## OpenClaw Model Roles

OpenClaw has 5 model slots, each with different requirements ([configuration reference](https://github.com/openclaw/openclaw/blob/main/docs/gateway/configuration-reference.md)):

| Config Key | Purpose | Optimise For |
|---|---|---|
| `agents.defaults.model.primary` | Main interactive model | Quality |
| `agents.defaults.model.fallbacks` | Ordered failover chain | Breadth, reliability |
| `agents.defaults.heartbeat.model` | Periodic heartbeat checks (default every 30m) | Cost, speed |
| `agents.defaults.subagents.model` | Spawned background sub-tasks | Quality/cost balance |
| `agents.defaults.imageModel.primary` | Vision/image understanding | Multimodal capability |

Failover behaviour ([model-failover docs](https://github.com/openclaw/openclaw/blob/main/docs/concepts/model-failover.md)):
- Auth profiles rotate within a provider first (exponential backoff: 1min/5min/25min/1hr cap)
- After exhausting all profiles for a provider, advances to next model in `fallbacks`
- Multiple API keys for the same provider multiply your effective rate limit

**Known bugs** (as of Feb 2026):
- `heartbeat.model` override is reportedly ignored at runtime -- heartbeats use the primary model instead ([Issue #9556](https://github.com/openclaw/openclaw/issues/9556))
- `subagents.model` override is reportedly ignored -- subagents use the primary model instead ([Issue #10963](https://github.com/openclaw/openclaw/issues/10963))

Until these bugs are fixed, the heartbeat and subagent model assignments below are aspirational. In practice, all roles may use the primary model. This makes the primary choice even more important, and strengthens the case for the multi-provider fallback chain.

---

## Model Landscape (Feb 2026)

### Frontier Models

Sources: [Chatbot Arena](https://lmarena.ai/), [Awesome Agents Feb 2026](https://awesomeagents.ai/leaderboards/overall-llm-rankings-feb-2026/), [Artificial Analysis](https://artificialanalysis.ai/), [llm-stats.com](https://llm-stats.com/)

| Model | Arena Elo | SWE-Bench V. | MMLU-Pro | GPQA Diamond | Input $/M | Output $/M |
|---|---|---|---|---|---|---|
| Gemini 3.1 Pro | -- | **80.6%** | -- | **94.3%** | $2.00 | $12.00 |
| Gemini 3 Pro | ~1487 | 63.2% | 89.8% | 87.5% | $1.25 | $5.00 |
| GPT-5.2 | ~1475 | 58.2% | 86.3% | 88.0% | $1.75 | $14.00 |
| Claude Opus 4.6 | ~1468 | 72.5% | 88.2% | 89.0% | $15.00 | $75.00 |
| Grok 4.1 | ~1483 | 75.0% | -- | -- | $3.00 | $15.00 |
| DeepSeek V3.2-Speciale | ~1361 | **77.8%** | 85.9% | 85.3% | $0.40 | $1.20 |
| Qwen 3.5 Plus | -- | -- | -- | 88.4% | $0.50 | $2.00 |
| GPT-5.3-Codex | -- | 56.8% (Pro) | -- | -- | $3.00 | $12.00 |
| Kimi K2.5 | ~1473 | 76.8% | 87.1% | 87.6% | $0.23 | $3.00 |
| GLM-5 | -- | 77.8% | ~70.4% | ~68.2% | $0.30 | $2.55 |

Gemini 3.1 Pro (released Feb 19, 2026) is the new price/performance leader in the paid tier. ([source](https://www.nxcode.io/en/resources/news/gemini-3-1-pro-complete-guide-benchmarks-pricing-api-2026))

Qwen 3.5 (released Feb 15, 2026) is a 397B MoE (17B active) that claims to outperform GPT-5.2 on 80% of benchmarks. ([source](https://venturebeat.com/technology/alibabas-qwen-3-5-397b-a17-beats-its-larger-trillion-parameter-model-at-a/))

### Free Models on OpenRouter

Source: [OpenRouter free models](https://openrouter.ai/collections/free-models), [CostGoat](https://costgoat.com/pricing/openrouter-free-models), [TeamDay](https://www.teamday.ai/blog/best-free-ai-models-openrouter-2026)

All models below have confirmed `$0` pricing via the OpenRouter API (Feb 22, 2026). Models with `:free` suffix are permanent free variants; models at `$0` without `:free` may be temporary promotions.

| Model | ID | Params | Context | Standout Strength |
|---|---|---|---|---|
| **Qwen3-Coder 480B** (Alibaba) | `qwen/qwen3-coder:free` | 480B MoE (35B active) | 262K | Best free coding model, Arena ~1455 coding, SWE-Bench 67-70% |
| **Step 3.5 Flash** (StepFun) | `stepfun/step-3.5-flash:free` | 196B MoE (11B active) | 256K | SWE-Bench 74.4%, LiveCodeBench 86.4%, ultra-efficient |
| **DeepSeek R1 0528** | `deepseek/deepseek-r1-0528:free` | 671B MoE (37B active) | 164K | Top open reasoning, Arena ~1464, AIME'25 87.5% |
| **GPT-OSS-120B** (OpenAI) | `openai/gpt-oss-120b:free` | 120B | 131K | SWE-Bench 62.4%, OpenAI's open-weight model |
| **Qwen3-235B-Thinking** | `qwen/qwen3-235b-a22b-thinking-2507` | 235B MoE (22B active) | 131K | AIME'25 92%, Arena ~1472. $0 but no `:free` suffix |
| **GLM-4.5 Air** (Zhipu) | `z-ai/glm-4.5-air:free` | -- | 131K | SWE-Bench ~59.8%, toggleable thinking mode |
| **Qwen3 Next 80B** | `qwen/qwen3-next-80b-a3b-instruct:free` | 80B MoE | 262K | General-purpose, large context |
| **Llama 3.3 70B** (Meta) | `meta-llama/llama-3.3-70b-instruct:free` | 70B | 128K | GPT-4 class, also free via Groq |
| **Hermes 3 405B** (Nous) | `nousresearch/hermes-3-llama-3.1-405b:free` | 405B | 131K | Instruction following specialist |
| **Mistral Small 3.1 24B** | `mistralai/mistral-small-3.1-24b-instruct:free` | 24B | 128K | Fast, vision capable |
| **Trinity Large Preview** (Arcee) | `arcee-ai/trinity-large-preview:free` | 400B MoE (13B active) | 131K | Reasoning + tools, preview quality |
| **Nemotron 3 Nano** (NVIDIA) | `nvidia/nemotron-3-nano-30b-a3b:free` | 30B MoE | 256K | Hybrid Mamba-Transformer |
| **Gemma 3 27B** (Google) | `google/gemma-3-27b-it:free` | 27B | 131K | Vision-language, multimodal |
| **Nemotron Nano 12B VL** (NVIDIA) | `nvidia/nemotron-nano-12b-v2-vl:free` | 12B | 128K | Vision + tools |
| **Solar Pro 3** (Upstage) | `upstage/solar-pro-3:free` | -- | 128K | Multilingual |

**Not free on OpenRouter** (corrections from previous version):
- **DeepSeek V3.2**: $0.26/$0.38 -- no `:free` variant exists. See Budget Paid section.
- **MiMo-V2-Flash**: Deprecating on OpenRouter since Jan 26, 2026. Do not plan around this model.
- **Gemini 2.5 Flash**: $0.30/$2.50 on OpenRouter. Free only via Google's direct API.
- **GPT-5 Nano**: $0.05/$0.40. Budget paid only.
- **Grok 4.1 Fast**: $0.20/$0.50. Free promotional period ended.

OpenRouter free tier: 50 RPD baseline, **1,000 RPD after a one-time $10 credit purchase**. ([source](https://openrouter.ai/docs/api/reference/limits))

### Budget Paid Models

| Model | Input $/M | Output $/M | Context | Notes |
|---|---|---|---|---|
| DeepSeek V3.2 | $0.25 | $0.38 | 164K | ~90% frontier quality, Arena ~1421 |
| GLM-4.7 Flash | $0.06 | $0.40 | 128-200K | 30B/3B active, free on Z.AI direct |
| Gemini 2.5 Flash | $0.10 | $0.30 | 1M | Fast multimodal, huge context |
| GPT-5-nano | $0.05 | $0.40 | 400K | Cheapest OpenAI |
| MiniMax M2.5 | $0.30 | $1.10 | 196K | 80% SWE-Bench Multilingual |
| Qwen 3.5 | $0.50 | $2.00 | 32K | Claims to beat GPT-5.2 |
| Mistral Large 3 | $0.50 | $1.50 | 256K | ~92% of GPT-5.2 at 15% of the cost |

---

## Free Tier Provider Comparison

Sources: [Google Gemini pricing](https://ai.google.dev/gemini-api/docs/pricing), [Groq](https://console.groq.com/docs/rate-limits), [Mistral](https://docs.mistral.ai/deployment/ai-studio/tier), [Cerebras](https://www.cerebras.ai/pricing), [xAI](https://x.ai/api), [Z.AI](https://docs.z.ai/guides/overview/pricing), [OpenRouter](https://openrouter.ai/docs/api/reference/limits)

| Provider | Free Tier | RPD | Monthly Tokens | Best Models | CC Required? |
|---|---|---|---|---|---|
| **OpenRouter** | Permanent | 50 (1,000 w/ $10) | ~30K-300K req | 20+ free models | No |
| **Google Gemini** | Permanent | 100-1,000/project | ~3K-30K req/project | Gemini 2.5 Pro/Flash | No |
| **Groq** | Permanent | 1,000-14,400 | ~30K-430K req | Llama 4, Qwen3, GPT-OSS | No |
| **Mistral** | Permanent | ~unlimited | **1B tokens** | Small, Large, Codestral | No (phone) |
| **Cerebras** | Permanent | 1M tokens/day | ~30M tokens | Llama 4, Qwen3 235B | No |
| **Z.AI (Zhipu)** | Permanent | 1 concurrency | -- | GLM-4.7 Flash | No |
| **xAI** | Credits | Pay-per-use | $175 first month | Grok 4 | No |

### Key Details

**Google Gemini**: Quotas are per-project, not per-key. Creating separate Google Cloud projects gives independent rate limits. 3 projects = 3x limits. This is [documented and legitimate](https://discuss.ai.google.dev/t/questions-about-multiple-free-paid-tier-projects/84682).

**Groq**: 14,400 RPD on llama-3.1-8b-instant (ideal for heartbeats). 300+ tok/s via custom LPU hardware. ([source](https://console.groq.com/docs/rate-limits))

**Mistral**: 1 billion tokens/month on the Experiment plan -- by far the most generous monthly allowance. Phone verification required (one phone per plan). ([source](https://help.mistral.ai/en/articles/455206-how-can-i-try-the-api-for-free-with-the-experiment-plan))

**Cerebras**: 1M tokens/day, 30 RPM. ~20x GPU inference speed. ([source](https://inference-docs.cerebras.ai/support/rate-limits))

**Z.AI**: Free tier for GLM-4.7 Flash with 1 concurrency, no credit card. Self-hosting also free under MIT license. ([source](https://docs.z.ai/guides/overview/pricing))

**xAI**: $25 signup credits (no expiry stated) plus $150/month if you opt into data sharing. ([source](https://venturebeat.com/ai/xai-woos-developers-with-25-month-worth-of-api-credits-support-for-openai-anthropic-sdks/))

---

## Pareto Analysis

A model is Pareto-optimal if no other model is strictly better on all of quality, speed, and cost.

### Free Models Pareto Frontier

| Model | Quality | Speed | Why On Frontier |
|---|---|---|---|
| **Qwen3-Coder 480B** | SWE-Bench 67-70%, Arena ~1455 | Medium | Best free coding specialist |
| **Step 3.5 Flash** | SWE-Bench 74.4%, LiveCodeBench 86.4% | Very fast | Highest free SWE-Bench + ultra-efficient (11B active) |
| **DeepSeek R1 0528** | Arena ~1464, best reasoning | Slow | Unmatched reasoning at $0 |
| **GPT-OSS-120B** | SWE-Bench 62.4% | Medium | Solid all-rounder, OpenAI open-weight |
| **GLM-4.5 Air** | SWE-Bench ~59.8% | Fast | Efficient, toggleable thinking |
| **Mistral Small 3.1** | Good general | Fast | Best small free model, vision capable |

### Budget Paid Pareto Frontier

| Model | Quality | Speed | Cost | Why On Frontier |
|---|---|---|---|---|
| **DeepSeek V3.2** | Arena ~1421 | Medium | $0.25/$0.38 | 90% frontier, 1/50th price |
| **GLM-4.7 Flash** | High for size | Very fast | $0.06/$0.40 | Best per-dollar in budget tier |
| **DeepSeek V3.2-Speciale** | 77.8% SWE-Bench | Medium | $0.40/$1.20 | Best SWE-Bench per dollar |
| **Gemini 2.5 Flash** | Medium-high | Fast | $0.10/$0.30 | Cheapest paid, 1M context |

### Dominated Models (eliminated)

- **MiMo-V2-Flash**: Was #1 open SWE-Bench (73.4%) but **deprecating on OpenRouter since Jan 26, 2026**. Do not plan around.
- **MiniMax M2.5** ($0.30/$1.10): Dominated by DeepSeek V3.2 (better Arena Elo, lower cost)
- **Llama 3.1 8B**: Dominated by Mistral Small 3.1 24B (same cost, better quality) and Step 3.5 Flash (same cost, much better quality)
- **Llama 4 Maverick**: Previously listed as free but not confirmed in current API. Multimodal role covered by Gemma 3 27B.

---

## Final Recommendation

### Model Assignments

| Role | Model | Provider | Cost | Why |
|---|---|---|---|---|
| **Primary** | `qwen/qwen3-coder:free` | OpenRouter | Free | Best free coding model. Arena ~1455 coding, SWE-Bench 67-70%, 262K context. |
| **Fallback 1** | `stepfun/step-3.5-flash:free` | OpenRouter | Free | Highest free SWE-Bench (74.4%). Ultra-efficient, only 11B active params. |
| **Fallback 2** | `deepseek/deepseek-r1-0528:free` | OpenRouter | Free | Best free reasoning. Arena ~1464. |
| **Fallback 3** | `llama-3.3-70b-versatile` | Groq | Free | Independent provider. 300+ tok/s. |
| **Fallback 4** | `qwen3-235b` | Cerebras | Free | Independent provider. 1M tok/day. Strong reasoning. |
| **Fallback 5** | `openai/gpt-oss-120b:free` | OpenRouter | Free | SWE-Bench 62.4%. Solid all-rounder. |
| **Fallback 6** | `mistral-small-latest` | Mistral | Free | 1B tok/month. Ultimate safety net. |
| **Heartbeat** | `llama-3.1-8b-instant` | Groq | Free | 14,400 RPD. 300+ tok/s. Runs every 30m, speed is critical. |
| **Subagents** | `qwen/qwen3-coder:free` | OpenRouter | Free | Same as primary. Coding quality matters for background tasks. |
| **Image** | `google/gemma-3-27b-it:free` | OpenRouter | Free | Vision-language, 131K context. Confirmed `:free` variant. |

### Rationale for Primary

Qwen3 Coder 480B is the strongest free coding model on OpenRouter. It has an Arena Elo of ~1455 on the coding leaderboard, SWE-Bench Verified 67-70%, SWE-Bench Pro 38.7%, and 262K context window. It's purpose-built for coding tasks, which aligns directly with OpenClaw's primary use case.

Step 3.5 Flash has a higher SWE-Bench score (74.4%) but is newer with less Arena data. It's placed as Fallback 1 -- an excellent complement to Qwen3 Coder with a different architecture and provider lineage.

**Why not DeepSeek V3.2?** It has a higher Arena Elo (~1421 overall) but is **not free** on OpenRouter ($0.26/$0.38 per MTok, no `:free` variant). It remains an excellent choice for the "Optimised Paid" config below.

Other alternatives considered:
- **Kimi K2.5** ($0.23/$3.00): 76.8% SWE-Bench, Arena ~1473 coding. Outstanding but $3/M output makes it expensive.
- **Qwen 3.5 Plus** ($0.50/$2.00): Claims to beat GPT-5.2 but too new for Arena Elo confirmation.
- **DeepSeek R1 0528**: Arena ~1464 but SWE-Bench only 57.6% standalone. Excellent at reasoning but weaker at real-world coding. Placed as Fallback 2.

### Multi-Provider Failover Strategy

Configure multiple providers so OpenClaw's failover chains across independent rate limits:

**Primary chain** (interactive):
1. OpenRouter -- Qwen3 Coder (best free coding quality)
2. OpenRouter -- Step 3.5 Flash (highest SWE-Bench among free models)
3. OpenRouter -- DeepSeek R1 0528 (best free reasoning)
4. Groq -- Llama 3.3 70B (independent limits, 300+ tok/s)
5. Cerebras -- Qwen3 235B (independent limits, strong reasoning)
6. OpenRouter -- GPT-OSS 120B (solid backup)
7. Mistral -- Mistral Small (1B tok/month safety net)

**Combined free capacity estimate**:

| Metric | Daily | Monthly |
|---|---|---|
| Requests | ~15,000-20,000 | ~450,000-600,000 |
| Tokens | ~5-10M | ~1B+ (Mistral anchor) |
| Cost | $0 | $0 |

### Google Gemini Multi-Project Bonus

Create 3 Google Cloud projects for 3x Gemini free tier limits:
- 300 RPD of Gemini 2.5 Pro
- 750 RPD of Gemini 2.5 Flash
- 3,000 RPD of Gemini 2.5 Flash-Lite

Configure each project's API key as a separate auth profile. OpenClaw rotates through them automatically.

### OpenRouter $10 Tip

A one-time $10 credit purchase raises the free-model rate limit from 50 to 1,000 RPD -- a 20x increase. ([source](https://openrouter.ai/docs/api/reference/limits))

---

## Config Snippets

### openclaw.json

```json
{
  "gateway": {
    "mode": "local"
  },
  "channels": {
    "telegram": {
      "dmPolicy": "allowlist",
      "allowFrom": [8051055216]
    }
  },
  "models": {
    "providers": {
      "openrouter": {
        "baseUrl": "https://openrouter.ai/api/v1",
        "apiKey": "${OPENROUTER_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen/qwen3-coder:free",
            "name": "Qwen3 Coder 480B",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 262144,
            "maxTokens": 65536,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "stepfun/step-3.5-flash:free",
            "name": "Step 3.5 Flash",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 256000,
            "maxTokens": 16384,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "deepseek/deepseek-r1-0528:free",
            "name": "DeepSeek R1 0528",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 163840,
            "maxTokens": 65536,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "openai/gpt-oss-120b:free",
            "name": "GPT-OSS 120B",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 131072,
            "maxTokens": 16384,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "google/gemma-3-27b-it:free",
            "name": "Gemma 3 27B",
            "reasoning": false,
            "input": ["text", "image"],
            "contextWindow": 131072,
            "maxTokens": 8192,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          }
        ]
      },
      "groq": {
        "baseUrl": "https://api.groq.com/openai/v1",
        "apiKey": "${GROQ_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "llama-3.3-70b-versatile",
            "name": "Llama 3.3 70B (Groq)",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 131072,
            "maxTokens": 32768,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "llama-3.1-8b-instant",
            "name": "Llama 3.1 8B Instant (Groq)",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 131072,
            "maxTokens": 8192,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          }
        ]
      },
      "cerebras": {
        "baseUrl": "https://api.cerebras.ai/v1",
        "apiKey": "${CEREBRAS_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3-235b",
            "name": "Qwen3 235B (Cerebras)",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 131072,
            "maxTokens": 16384,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          }
        ]
      },
      "mistral": {
        "baseUrl": "https://api.mistral.ai/v1",
        "apiKey": "${MISTRAL_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "mistral-small-latest",
            "name": "Mistral Small (1B tok/mo free)",
            "reasoning": false,
            "input": ["text", "image"],
            "contextWindow": 131072,
            "maxTokens": 8192,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/qwen/qwen3-coder:free",
        "fallbacks": [
          "openrouter/stepfun/step-3.5-flash:free",
          "openrouter/deepseek/deepseek-r1-0528:free",
          "groq/llama-3.3-70b-versatile",
          "cerebras/qwen3-235b",
          "openrouter/openai/gpt-oss-120b:free",
          "mistral/mistral-small-latest"
        ]
      },
      "heartbeat": {
        "every": "30m",
        "model": "groq/llama-3.1-8b-instant"
      },
      "subagents": {
        "model": "openrouter/qwen/qwen3-coder:free"
      },
      "imageModel": {
        "primary": "openrouter/google/gemma-3-27b-it:free"
      },
      "models": {
        "openrouter/qwen/qwen3-coder:free": { "alias": "coder" },
        "openrouter/stepfun/step-3.5-flash:free": { "alias": "step" },
        "openrouter/deepseek/deepseek-r1-0528:free": { "alias": "reason" },
        "openrouter/openai/gpt-oss-120b:free": { "alias": "gptoss" },
        "openrouter/google/gemma-3-27b-it:free": { "alias": "gemma" },
        "groq/llama-3.3-70b-versatile": { "alias": "groq" },
        "cerebras/qwen3-235b": { "alias": "cerebras" },
        "mistral/mistral-small-latest": { "alias": "mistral" }
      }
    }
  }
}
```

### nix-claw default.nix snippet

Replace the `initialConfig` block in `/Users/jack/workspace/nix-claw/hosts/claw/default.nix` (lines 93-121):

```nix
  systemd.services.openclaw-gateway.preStart = let
    initialConfig = builtins.toJSON {
      gateway.mode = "local";
      channels.telegram = {
        dmPolicy = "allowlist";
        allowFrom = [ 8051055216 ];
      };
      models.providers = {
        openrouter = {
          baseUrl = "https://openrouter.ai/api/v1";
          apiKey = "\${OPENROUTER_API_KEY}";
          api = "openai-completions";
          models = [
            {
              id = "qwen/qwen3-coder:free";
              name = "Qwen3 Coder 480B";
              reasoning = false;
              input = [ "text" ];
              contextWindow = 262144;
              maxTokens = 65536;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "stepfun/step-3.5-flash:free";
              name = "Step 3.5 Flash";
              reasoning = true;
              input = [ "text" ];
              contextWindow = 256000;
              maxTokens = 16384;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "deepseek/deepseek-r1-0528:free";
              name = "DeepSeek R1 0528";
              reasoning = true;
              input = [ "text" ];
              contextWindow = 163840;
              maxTokens = 65536;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "openai/gpt-oss-120b:free";
              name = "GPT-OSS 120B";
              reasoning = false;
              input = [ "text" ];
              contextWindow = 131072;
              maxTokens = 16384;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "google/gemma-3-27b-it:free";
              name = "Gemma 3 27B";
              reasoning = false;
              input = [ "text" "image" ];
              contextWindow = 131072;
              maxTokens = 8192;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
          ];
        };
        groq = {
          baseUrl = "https://api.groq.com/openai/v1";
          apiKey = "\${GROQ_API_KEY}";
          api = "openai-completions";
          models = [
            {
              id = "llama-3.3-70b-versatile";
              name = "Llama 3.3 70B (Groq)";
              reasoning = false;
              input = [ "text" ];
              contextWindow = 131072;
              maxTokens = 32768;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "llama-3.1-8b-instant";
              name = "Llama 3.1 8B Instant (Groq)";
              reasoning = false;
              input = [ "text" ];
              contextWindow = 131072;
              maxTokens = 8192;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
          ];
        };
        cerebras = {
          baseUrl = "https://api.cerebras.ai/v1";
          apiKey = "\${CEREBRAS_API_KEY}";
          api = "openai-completions";
          models = [{
            id = "qwen3-235b";
            name = "Qwen3 235B (Cerebras)";
            reasoning = true;
            input = [ "text" ];
            contextWindow = 131072;
            maxTokens = 16384;
            cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
          }];
        };
        mistral = {
          baseUrl = "https://api.mistral.ai/v1";
          apiKey = "\${MISTRAL_API_KEY}";
          api = "openai-completions";
          models = [{
            id = "mistral-small-latest";
            name = "Mistral Small (1B tok/mo free)";
            reasoning = false;
            input = [ "text" "image" ];
            contextWindow = 131072;
            maxTokens = 8192;
            cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
          }];
        };
      };
      agents.defaults = {
        model = {
          primary = "openrouter/qwen/qwen3-coder:free";
          fallbacks = [
            "openrouter/stepfun/step-3.5-flash:free"
            "openrouter/deepseek/deepseek-r1-0528:free"
            "groq/llama-3.3-70b-versatile"
            "cerebras/qwen3-235b"
            "openrouter/openai/gpt-oss-120b:free"
            "mistral/mistral-small-latest"
          ];
        };
        heartbeat = {
          every = "30m";
          model = "groq/llama-3.1-8b-instant";
        };
        subagents.model = "openrouter/qwen/qwen3-coder:free";
        imageModel.primary = "openrouter/google/gemma-3-27b-it:free";
        models = {
          "openrouter/qwen/qwen3-coder:free" = { alias = "coder"; };
          "openrouter/stepfun/step-3.5-flash:free" = { alias = "step"; };
          "openrouter/deepseek/deepseek-r1-0528:free" = { alias = "reason"; };
          "openrouter/openai/gpt-oss-120b:free" = { alias = "gptoss"; };
          "openrouter/google/gemma-3-27b-it:free" = { alias = "gemma"; };
          "groq/llama-3.3-70b-versatile" = { alias = "groq"; };
          "cerebras/qwen3-235b" = { alias = "cerebras"; };
          "mistral/mistral-small-latest" = { alias = "mistral"; };
        };
      };
    };
  in ''
    mkdir -p /home/claw/.openclaw
    if [ ! -f /home/claw/.openclaw/openclaw.json ]; then
      echo '${initialConfig}' > /home/claw/.openclaw/openclaw.json
    fi
  '';
```

Add API keys to `/home/claw/.openclaw/env`:

```bash
OPENROUTER_API_KEY=sk-or-...
GROQ_API_KEY=gsk_...
CEREBRAS_API_KEY=csk-...
MISTRAL_API_KEY=...
```

---

## Alternative: Haiku Primary / Opus Subagents / Ollama Heartbeat

A popular community pattern uses Anthropic models with local inference for heartbeats. Sources: [OpenClaw-Token-Optimization](https://github.com/wassupjay/OpenClaw-Token-Optimization), [Opus-as-Orchestrator discussion #858](https://github.com/openclaw/openclaw/discussions/858), [VelvetShark multi-model routing](https://velvetshark.com/openclaw-multi-model-routing).

### The Pattern

| Role | Model | Cost |
|---|---|---|
| Primary | Claude Haiku 4.5 | $1.00/$5.00 per MTok |
| Subagents | Claude Opus 4.6 | $5.00/$25.00 per MTok |
| Heartbeat | Ollama llama3.2:3b (local) | $0 |

The idea: cheap Haiku handles high-volume interactions, expensive Opus handles complex background reasoning, local Ollama handles heartbeats for free.

OpenClaw supports Ollama natively -- set `OLLAMA_API_KEY="ollama-local"` or configure it explicitly as a provider with `baseUrl: "http://localhost:11434"`. ([Ollama integration docs](https://docs.ollama.com/integrations/openclaw))

### Anthropic Pricing (Feb 2026)

Source: [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing), [OpenRouter](https://openrouter.ai/provider/anthropic)

| Model | Input $/MTok | Output $/MTok | Cache Read | Cache Write (5m) |
|---|---|---|---|---|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.6 | $5.00 | $25.00 | $0.50 | $6.25 |

### Monthly Cost Estimate

Assuming moderate personal usage: ~1,000 messages/day, ~500 tokens avg, 100 subagent calls/day at ~2,000 tokens:

| Component | Monthly Tokens | Rate | Monthly Cost |
|---|---|---|---|
| Haiku input | 18M | $1.00/MTok | $18 |
| Haiku output | 12M | $5.00/MTok | $60 |
| Opus input | 3.6M | $5.00/MTok | $18 |
| Opus output | 2.4M | $25.00/MTok | $60 |
| Ollama | -- | $0 | $0 |
| **Total** | | | **~$156/month** |

With prompt caching (~50% hit rate): **~$140/month**. With Batch API on subagents: **~$117/month**.

### Better Alternatives Exist for Both Roles

**Haiku replacement -- multiple models are higher quality AND cheaper:**

| Model | Arena Elo | Input $/MTok | Output $/MTok | vs Haiku |
|---|---|---|---|---|
| Claude Haiku 4.5 | ~1280 | $1.00 | $5.00 | baseline |
| **DeepSeek V3.2** | **~1421** | **$0.26** | **$0.38** | +141 Elo, 13x cheaper output |
| **Qwen3 Coder 480B** | **~1455** (coding) | **$0.00** | **$0.00** | +175 Elo (coding), free |
| Gemini 2.5 Flash | ~1300 | $0.10 | $0.30 | +20 Elo, 17x cheaper output |
| GLM-4.7 Flash | -- | $0.06 | $0.40 | 12x cheaper output |

DeepSeek V3.2 ($0.26/$0.38, not free) is objectively better than Haiku on Arena Elo (~1421 vs ~1280) at 13x less output cost. Qwen3 Coder beats Haiku on the coding Arena leaderboard (~1455) while being completely free. Haiku's advantage is speed and consistency, but not quality or cost.

**Opus replacement -- Gemini 3.1 Pro beats it on SWE-Bench at half the cost:**

| Model | SWE-Bench V. | Arena Elo | Input $/MTok | Output $/MTok | vs Opus |
|---|---|---|---|---|---|
| Claude Opus 4.6 | 72.5% | ~1468 | $5.00 | $25.00 | baseline |
| **Gemini 3.1 Pro** | **80.6%** | -- | **$2.00** | **$12.00** | +8.1% SWE-Bench, 2x cheaper |
| Gemini 3 Pro | 63.2% | ~1487 | $1.25 | $5.00 | +19 Elo, 5x cheaper |
| DeepSeek V3.2-Speciale | 77.8% | ~1361 | $0.40 | $1.20 | +5.3% SWE-Bench, 21x cheaper |
| Kimi K2.5 | 76.8% | ~1473 | $0.23 | $3.00 | +4.3% SWE-Bench, 8x cheaper |

### Optimised Paid Alternative

If you want to pay for quality rather than use the free tier:

| Role | Model | Cost | Why |
|---|---|---|---|
| Primary | DeepSeek V3.2 | $0.25/$0.38 | Higher Arena Elo than Haiku, 13x cheaper |
| Subagents | Gemini 3.1 Pro | $2.00/$12.00 | Higher SWE-Bench than Opus, half the cost |
| Heartbeat | Ollama llama3.2:3b | $0 | Local, instant |

**Estimated monthly cost: ~$15-25/month** (vs ~$156 for Haiku/Opus, vs $0 for free tier).

### Comparison Summary

| | Free Tier (recommended) | Haiku/Opus/Ollama | Optimised Paid |
|---|---|---|---|
| Primary quality | Qwen3-Coder Arena ~1455 (coding) | Arena ~1280 | DeepSeek V3.2 Arena ~1421 |
| Subagent quality | Qwen3-Coder (rivals Sonnet) | Arena ~1468 (Opus) | 80.6% SWE-Bench (Gemini 3.1 Pro) |
| Monthly cost | **$0** | ~$156 | ~$15-25 |
| Reliability | Multi-provider failover | Single provider | Dual provider |
| Capacity | ~1B+ tok/month | Unlimited (pay) | Unlimited (pay) |

**Verdict:** The Haiku/Opus pattern was a reasonable approach when Haiku was the best cheap model available. In February 2026, free models like Qwen3 Coder (Arena ~1455 coding) and Step 3.5 Flash (SWE-Bench 74.4%) surpass Haiku in coding quality at zero cost. DeepSeek V3.2 ($0.26/$0.38) surpasses Haiku on general Arena Elo at 13x less. Gemini 3.1 Pro has surpassed Opus on coding benchmarks at half the price. The free-tier multi-provider strategy is the best value; the optimised paid alternative gives similar quality to Haiku/Opus at ~1/6th the cost.

---

## Notable Models Not Recommended (and why)

| Model | Why Not Primary/Fallback |
|---|---|
| **DeepSeek V3.2** ($0.26/$0.38) | Arena ~1421 overall -- highest quality budget model. But **not free** on OpenRouter (no `:free` variant). Recommended for the "Optimised Paid" config instead. |
| **Kimi K2.5** ($0.23/$3.00) | Strong (76.8% SWE-Bench, Arena ~1473 coding) but $3/M output. Not free on OpenRouter. |
| **GLM-5** ($0.30/$2.55) | 77.8% SWE-Bench, MIT, record-low hallucination. But $2.55/M output and not free on OpenRouter. |
| **GLM-4.7 Flash** ($0.06/$0.40) | Excellent for its size (30B/3B active). Has free Z.AI tier. Could replace Groq heartbeat model if Z.AI proves reliable. |
| **Qwen 3.5 Plus** ($0.50/$2.00) | Claims to beat GPT-5.2. Too new (Feb 15) for Arena Elo confirmation. Monitor closely -- could become primary if claims hold. |
| **Mistral Large 3** ($0.50/$1.50) | ~92% of GPT-5.2 at 15% price. Good but not free. |
| **MiniMax M2.5** ($0.30/$1.10) | 80% SWE-Bench Multilingual. Dominated by DeepSeek V3.2 (better Arena, cheaper). |
| **MiMo-V2-Flash** (was free) | Was #1 open SWE-Bench (73.4%) but **deprecating on OpenRouter since Jan 26, 2026**. |
| **Trinity Large Preview** (free) | Free on OpenRouter, 131K context. Included in free models table but preview quality -- not yet proven enough for the primary fallback chain. |

---

## References

### Benchmark Sources
- [Chatbot Arena (LMSYS)](https://lmarena.ai/)
- [LMSYS Coding Leaderboard](https://aidevdayindia.org/blogs/lmsys-chatbot-arena-current-rankings/lmsys-chatbot-arena-coding-leaderboard-2026.html)
- [Awesome Agents Overall LLM Rankings Feb 2026](https://awesomeagents.ai/leaderboards/overall-llm-rankings-feb-2026/)
- [LiveBench](https://livebench.ai/)
- [llm-stats.com](https://llm-stats.com/)
- [Artificial Analysis](https://artificialanalysis.ai/)

### Model Sources
- [Gemini 3.1 Pro guide](https://www.nxcode.io/en/resources/news/gemini-3-1-pro-complete-guide-benchmarks-pricing-api-2026)
- [GPT-5.3-Codex](https://openai.com/index/introducing-gpt-5-3-codex/)
- [MiMo-V2-Flash](https://github.com/XiaomiMiMo/MiMo-V2-Flash)
- [Qwen3 blog](https://qwenlm.github.io/blog/qwen3/)
- [Qwen 3.5 announcement](https://venturebeat.com/technology/alibabas-qwen-3-5-397b-a17-beats-its-larger-trillion-parameter-model-at-a/)
- [Devstral 2](https://mistral.ai/news/devstral-2-vibe-cli)
- [Kimi K2.5 tech blog](https://www.kimi.com/blog/kimi-k2-5.html)
- [Kimi K2.5 on OpenRouter](https://openrouter.ai/moonshotai/kimi-k2.5)
- [GLM-5 deep dive](https://medium.com/@gsaidheeraj/glm-5-deep-dive-745b-moe-beast-crushing-swe-bench-code-benchmarks-5757a3022251)
- [GLM-5 on OpenRouter](https://openrouter.ai/z-ai/glm-5)
- [GLM-4.7 Flash on OpenRouter](https://openrouter.ai/z-ai/glm-4.7-flash)
- [GLM-4.7 Flash analysis](https://towardsai.net/p/machine-learning/glm-4-7-flash-z-ais-free-coding-model-and-what-the-benchmarks-say)
- [Step 3.5 Flash on OpenRouter](https://openrouter.ai/stepfun/step-3.5-flash:free)
- [Step 3.5 Flash GitHub](https://github.com/stepfun-ai/Step-3.5-Flash)
- [Trinity Large Preview](https://www.arcee.ai/blog/trinity-large)
- [Nemotron 3 Nano](https://nvidianews.nvidia.com/news/nvidia-debuts-nemotron-3-family-of-open-models)
- [MiniMax M2.5](https://artificialanalysis.ai/models/minimax-m2-5)
- [Mistral Large 3](https://artificialanalysis.ai/models/mistral-large-3)
- [Grok 4.1](https://x.ai/news/grok-4-1)

### Pricing and Free Tier Sources
- [OpenRouter pricing](https://openrouter.ai/pricing)
- [OpenRouter rate limits](https://openrouter.ai/docs/api/reference/limits)
- [OpenRouter free models](https://openrouter.ai/collections/free-models)
- [CostGoat free models](https://costgoat.com/pricing/openrouter-free-models)
- [TeamDay free models guide](https://www.teamday.ai/blog/best-free-ai-models-openrouter-2026)
- [TeamDay OpenRouter ranked](https://www.teamday.ai/blog/top-ai-models-openrouter-2026)
- [Google Gemini pricing](https://ai.google.dev/gemini-api/docs/pricing)
- [Gemini rate limits](https://www.aifreeapi.com/en/posts/gemini-api-free-tier-rate-limits)
- [Google multi-project](https://discuss.ai.google.dev/t/questions-about-multiple-free-paid-tier-projects/84682)
- [Groq rate limits](https://console.groq.com/docs/rate-limits)
- [Mistral free tier](https://help.mistral.ai/en/articles/455206-how-can-i-try-the-api-for-free-with-the-experiment-plan)
- [Mistral tiers](https://docs.mistral.ai/deployment/ai-studio/tier)
- [Cerebras pricing](https://www.cerebras.ai/pricing)
- [Cerebras rate limits](https://inference-docs.cerebras.ai/support/rate-limits)
- [Z.AI pricing](https://docs.z.ai/guides/overview/pricing)
- [xAI credits](https://venturebeat.com/ai/xai-woos-developers-with-25-month-worth-of-api-credits-support-for-openai-anthropic-sdks/)

### OpenClaw Configuration
- [Configuration reference](https://github.com/openclaw/openclaw/blob/main/docs/gateway/configuration-reference.md)
- [Model concepts](https://github.com/openclaw/openclaw/blob/main/docs/concepts/models.md)
- [Model failover](https://github.com/openclaw/openclaw/blob/main/docs/concepts/model-failover.md)
- [nix-openclaw](https://github.com/openclaw/nix-openclaw)
- [Heartbeat model override bug #9556](https://github.com/openclaw/openclaw/issues/9556)
- [Subagent model config bug #10963](https://github.com/openclaw/openclaw/issues/10963)
- [Ollama integration docs](https://docs.ollama.com/integrations/openclaw)

### Community Configuration Guides
- [OpenClaw-Token-Optimization (wassupjay)](https://github.com/wassupjay/OpenClaw-Token-Optimization)
- [Opus-as-Orchestrator discussion #858](https://github.com/openclaw/openclaw/discussions/858)
- [VelvetShark multi-model routing](https://velvetshark.com/openclaw-multi-model-routing)
- [Dual agent setup guide](https://dual-agent-guide.vercel.app/)
- [AI model orchestration cost report](https://gist.github.com/ClaudiaCodeMaster/a7f47cf7781a8ca199d1fc7e2adb793d)
- [Running OpenClaw without burning money](https://gist.github.com/digitalknk/ec360aab27ca47cb4106a183b2c25a98)
- [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
