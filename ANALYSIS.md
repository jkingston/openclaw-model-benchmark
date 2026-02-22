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

| Model | Params | Context | Standout Strength |
|---|---|---|---|
| **MiMo-V2-Flash** (Xiaomi) | 309B MoE (15B active) | 262K | #1 open SWE-Bench (73.4%), 150 tok/s |
| **Qwen3-Coder 480B** (Alibaba) | 480B MoE (35B active) | 262K | Rivals Claude Sonnet on coding |
| **Devstral 2** (Mistral) | 123B dense | 262K | Agentic coding (72.2% SWE-Bench), MIT |
| **Qwen3-235B-Thinking** | 235B MoE (22B active) | 131K | AIME'25 92%, MATH-500 98% |
| **DeepSeek R1 0528** | 671B MoE (37B active) | 164K | Top open reasoning |
| **Llama 4 Maverick** (Meta) | 400B MoE (17B active) | 256K | GPT-4 class, multimodal, Arena ~1320 |
| **Step 3.5 Flash** (StepFun) | 196B MoE (11B active) | 256K | Ultra-efficient reasoning, trending |
| **GLM-4.5 Air** (Zhipu) | -- | 131K | Agentic, toggleable thinking mode |
| **Trinity Large Preview** (Arcee) | 400B MoE (13B active) | 512K | US-built, 2-3x faster than peers |
| **Nemotron 3 Nano** (NVIDIA) | 30B MoE | 256K | Hybrid Mamba-Transformer, 1M context |
| **GPT-OSS-120B** (OpenAI) | 120B | 131K | OpenAI's open-weight model |
| **Llama 3.1 405B** (Meta) | 405B dense | 131K | Largest dense open model |
| **Gemini 2.0 Flash Exp** (Google) | -- | 1M | 1M context, multimodal |
| **Hermes 3 405B** (Nous) | 405B | 131K | Instruction following specialist |
| **Mistral Small 3.1 24B** | 24B | 128K | Fast, vision capable |
| **Gemma 3 27B** (Google) | 27B | 131K | Vision-language |

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
| **Qwen3-Coder 480B** | Highest coding | Medium | Best free coding model |
| **MiMo-V2-Flash** | 73.4% SWE-Bench | Fast (150 tok/s) | Best free SWE-Bench + fast |
| **DeepSeek R1 0528** | Best reasoning | Slow | Unmatched reasoning at $0 |
| **Llama 4 Maverick** | Arena ~1320, multimodal | Medium | Best free general + vision |
| **Step 3.5 Flash** | Strong reasoning | Very fast | Best efficiency (11B active) |
| **Mistral Small 3.1** | Good general | Fast | Best small free model |

### Budget Paid Pareto Frontier

| Model | Quality | Speed | Cost | Why On Frontier |
|---|---|---|---|---|
| **DeepSeek V3.2** | Arena ~1421 | Medium | $0.25/$0.38 | 90% frontier, 1/50th price |
| **GLM-4.7 Flash** | High for size | Very fast | $0.06/$0.40 | Best per-dollar in budget tier |
| **DeepSeek V3.2-Speciale** | 77.8% SWE-Bench | Medium | $0.40/$1.20 | Best SWE-Bench per dollar |
| **Gemini 2.5 Flash** | Medium-high | Fast | $0.10/$0.30 | Cheapest paid, 1M context |

### Dominated Models (eliminated)

- **MiniMax M2.5** ($0.30/$1.10): Dominated by DeepSeek V3.2 (better Arena Elo, lower cost)
- **GPT-4o-mini**: Dominated by free models with higher quality
- **Llama 3.1 8B**: Dominated by Mistral Small 3.1 24B (same cost, better quality) and Step 3.5 Flash (same cost, much better quality)

---

## Final Recommendation

### Model Assignments

| Role | Model | Provider | Cost | Why |
|---|---|---|---|---|
| **Primary** | `deepseek/deepseek-v3.2` | OpenRouter | Free / $0.25/$0.38 | Arena Elo ~1421 (4th overall). Best quality-to-cost ratio in the market. |
| **Fallback 1** | `qwen/qwen3-coder-480b-a35b` | OpenRouter | Free | Rivals Claude Sonnet on coding. Different provider lineage. |
| **Fallback 2** | `llama-3.3-70b-versatile` | Groq | Free | Independent rate limits from OpenRouter. 300+ tok/s. |
| **Fallback 3** | `qwen3-235b` | Cerebras | Free | Independent provider. 1M tok/day. Strong reasoning. |
| **Fallback 4** | `stepfun/step-3.5-flash` | OpenRouter | Free | 196B MoE, only 11B active. Ultra-efficient. |
| **Fallback 5** | `mistral-small-latest` | Mistral | Free | 1B tok/month. Ultimate safety net. |
| **Fallback 6** | `meta-llama/llama-4-maverick` | OpenRouter | Free | Multimodal fallback. GPT-4 class. |
| **Fallback 7** | `deepseek/deepseek-r1-0528` | OpenRouter | Free | Reasoning specialist fallback. |
| **Heartbeat** | `llama-3.1-8b-instant` | Groq | Free | 14,400 RPD. 300+ tok/s. Runs every 30m, speed is critical. |
| **Subagents** | `qwen/qwen3-coder-480b-a35b` | OpenRouter | Free | Strong coding + general. Background tasks need quality. |
| **Image** | `meta-llama/llama-4-maverick` | OpenRouter | Free | Native multimodal. |

### Rationale for Primary

DeepSeek V3.2 has the highest Arena Elo (~1421) among models that are free or near-free on OpenRouter. It scores 4th overall on Chatbot Arena, behind only Gemini 3 Pro, GPT-5.2, and Claude Opus 4.6 -- all of which cost 5-200x more. It's available free on OpenRouter (rate-limited) and only $0.25/$0.38/M paid.

Alternatives considered:
- **Qwen 3.5 Plus** ($0.50/$2.00): Claims to beat GPT-5.2 but newer and less proven. Would be a strong primary if Arena Elo confirms the claims.
- **Kimi K2.5** ($0.23/$3.00): 76.8% SWE-Bench and 262K context. Strong but higher output cost and not free.
- **GLM-5** ($0.30/$2.55): 77.8% SWE-Bench, record-low hallucination, MIT license. Strong but not free and output cost is higher.

### Multi-Provider Failover Strategy

Configure multiple providers so OpenClaw's failover chains across independent rate limits:

**Primary chain** (interactive):
1. OpenRouter -- DeepSeek V3.2 (best quality)
2. Groq -- Llama 3.3 70B (fastest, independent limits)
3. Cerebras -- Qwen3 235B (independent limits, strong reasoning)
4. OpenRouter -- Step 3.5 Flash, Maverick, R1 (additional free models)
5. Mistral -- Mistral Small (1B tok/month safety net)

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
            "id": "deepseek/deepseek-v3.2",
            "name": "DeepSeek V3.2",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 163840,
            "maxTokens": 16384,
            "cost": { "input": 0.25, "output": 0.38, "cacheRead": 0.025, "cacheWrite": 0.25 }
          },
          {
            "id": "qwen/qwen3-coder-480b-a35b",
            "name": "Qwen3 Coder 480B",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 262144,
            "maxTokens": 65536,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "stepfun/step-3.5-flash",
            "name": "Step 3.5 Flash",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 262144,
            "maxTokens": 16384,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "meta-llama/llama-4-maverick",
            "name": "Llama 4 Maverick",
            "reasoning": false,
            "input": ["text", "image"],
            "contextWindow": 262144,
            "maxTokens": 65536,
            "cost": { "input": 0.0, "output": 0.0, "cacheRead": 0.0, "cacheWrite": 0.0 }
          },
          {
            "id": "deepseek/deepseek-r1-0528",
            "name": "DeepSeek R1 0528",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 163840,
            "maxTokens": 65536,
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
        "primary": "openrouter/deepseek/deepseek-v3.2",
        "fallbacks": [
          "openrouter/qwen/qwen3-coder-480b-a35b",
          "groq/llama-3.3-70b-versatile",
          "cerebras/qwen3-235b",
          "openrouter/stepfun/step-3.5-flash",
          "mistral/mistral-small-latest",
          "openrouter/meta-llama/llama-4-maverick",
          "openrouter/deepseek/deepseek-r1-0528"
        ]
      },
      "heartbeat": {
        "every": "30m",
        "model": "groq/llama-3.1-8b-instant"
      },
      "subagents": {
        "model": "openrouter/qwen/qwen3-coder-480b-a35b"
      },
      "imageModel": {
        "primary": "openrouter/meta-llama/llama-4-maverick"
      },
      "models": {
        "openrouter/deepseek/deepseek-v3.2": { "alias": "default" },
        "openrouter/qwen/qwen3-coder-480b-a35b": { "alias": "coder" },
        "openrouter/stepfun/step-3.5-flash": { "alias": "step" },
        "openrouter/meta-llama/llama-4-maverick": { "alias": "maverick" },
        "openrouter/deepseek/deepseek-r1-0528": { "alias": "reason" },
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
              id = "deepseek/deepseek-v3.2";
              name = "DeepSeek V3.2";
              reasoning = false;
              input = [ "text" ];
              contextWindow = 163840;
              maxTokens = 16384;
              cost = { input = 0.25; output = 0.38; cacheRead = 0.025; cacheWrite = 0.25; };
            }
            {
              id = "qwen/qwen3-coder-480b-a35b";
              name = "Qwen3 Coder 480B";
              reasoning = false;
              input = [ "text" ];
              contextWindow = 262144;
              maxTokens = 65536;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "stepfun/step-3.5-flash";
              name = "Step 3.5 Flash";
              reasoning = true;
              input = [ "text" ];
              contextWindow = 262144;
              maxTokens = 16384;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "meta-llama/llama-4-maverick";
              name = "Llama 4 Maverick";
              reasoning = false;
              input = [ "text" "image" ];
              contextWindow = 262144;
              maxTokens = 65536;
              cost = { input = 0.0; output = 0.0; cacheRead = 0.0; cacheWrite = 0.0; };
            }
            {
              id = "deepseek/deepseek-r1-0528";
              name = "DeepSeek R1 0528";
              reasoning = true;
              input = [ "text" ];
              contextWindow = 163840;
              maxTokens = 65536;
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
          primary = "openrouter/deepseek/deepseek-v3.2";
          fallbacks = [
            "openrouter/qwen/qwen3-coder-480b-a35b"
            "groq/llama-3.3-70b-versatile"
            "cerebras/qwen3-235b"
            "openrouter/stepfun/step-3.5-flash"
            "mistral/mistral-small-latest"
            "openrouter/meta-llama/llama-4-maverick"
            "openrouter/deepseek/deepseek-r1-0528"
          ];
        };
        heartbeat = {
          every = "30m";
          model = "groq/llama-3.1-8b-instant";
        };
        subagents.model = "openrouter/qwen/qwen3-coder-480b-a35b";
        imageModel.primary = "openrouter/meta-llama/llama-4-maverick";
        models = {
          "openrouter/deepseek/deepseek-v3.2" = { alias = "default"; };
          "openrouter/qwen/qwen3-coder-480b-a35b" = { alias = "coder"; };
          "openrouter/stepfun/step-3.5-flash" = { alias = "step"; };
          "openrouter/meta-llama/llama-4-maverick" = { alias = "maverick"; };
          "openrouter/deepseek/deepseek-r1-0528" = { alias = "reason"; };
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

**Haiku replacement -- DeepSeek V3.2 is higher quality AND cheaper:**

| Model | Arena Elo | Input $/MTok | Output $/MTok | vs Haiku |
|---|---|---|---|---|
| Claude Haiku 4.5 | ~1280 | $1.00 | $5.00 | baseline |
| **DeepSeek V3.2** | **~1421** | **$0.25** | **$0.38** | +141 Elo, 13x cheaper output |
| Gemini 2.5 Flash | ~1300 | $0.10 | $0.30 | +20 Elo, 17x cheaper output |
| GLM-4.7 Flash | -- | $0.06 | $0.40 | 12x cheaper output |

DeepSeek V3.2 is objectively better than Haiku on Arena Elo (~1421 vs ~1280) while costing 13x less on output. Haiku's advantage is speed and consistency, but not quality.

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
| Primary quality | Arena ~1421 | Arena ~1280 | Arena ~1421 |
| Subagent quality | Qwen3-Coder (rivals Sonnet) | Arena ~1468 (Opus) | 80.6% SWE-Bench (Gemini 3.1 Pro) |
| Monthly cost | **$0** | ~$156 | ~$15-25 |
| Reliability | Multi-provider failover | Single provider | Dual provider |
| Capacity | ~1B+ tok/month | Unlimited (pay) | Unlimited (pay) |

**Verdict:** The Haiku/Opus pattern was a reasonable approach when Haiku was the best cheap model available. In February 2026, DeepSeek V3.2 has surpassed Haiku in quality at a fraction of the cost, and Gemini 3.1 Pro has surpassed Opus on coding benchmarks at half the price. The free-tier multi-provider strategy is the best value; the optimised paid alternative gives similar quality to Haiku/Opus at ~1/6th the cost.

---

## Notable Models Not Recommended (and why)

| Model | Why Not Primary/Fallback |
|---|---|
| **Kimi K2.5** ($0.23/$3.00) | Strong (76.8% SWE-Bench, Arena ~1473) but $3/M output is 8x DeepSeek V3.2. Not free on OpenRouter. |
| **GLM-5** ($0.30/$2.55) | 77.8% SWE-Bench, MIT, record-low hallucination. But $2.55/M output and not free on OpenRouter. |
| **GLM-4.7 Flash** ($0.06/$0.40) | Excellent for its size (30B/3B active). Has free Z.AI tier. Could replace Groq heartbeat model if Z.AI proves reliable. |
| **Qwen 3.5 Plus** ($0.50/$2.00) | Claims to beat GPT-5.2. Too new (Feb 15) for Arena Elo confirmation. Monitor closely -- could become primary if claims hold. |
| **Mistral Large 3** ($0.50/$1.50) | ~92% of GPT-5.2 at 15% price. Good but not free. |
| **MiniMax M2.5** ($0.30/$1.10) | 80% SWE-Bench Multilingual. Dominated by DeepSeek V3.2 (better Arena, cheaper). |
| **GLM-4.5 Air** (free) | Free on OpenRouter. Good agentic model but less proven than the recommended free models. Worth adding to fallbacks if primary recommendations underperform. |
| **Trinity Large Preview** (free) | Free on OpenRouter, 512K context. Preview quality -- monitor for stability before promoting to fallbacks. |

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
