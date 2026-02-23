# OpenClaw Model Tiering

Optimal model selection for [OpenClaw](https://github.com/openclaw/openclaw) using OpenRouter and direct-provider free tiers.

## Documents

| Document | Purpose |
|---|---|
| [ANALYSIS.md](ANALYSIS.md) | Full analysis: benchmarks, free tiers, Pareto frontier, recommendation, config snippets |
| [METHODOLOGY.md](METHODOLOGY.md) | How to rebuild this analysis from scratch |
| [REFRESH.md](REFRESH.md) | When to refresh, what triggers changes, models to watch |

## Current Recommendation (Feb 2026)

| Role | Model | Provider | Cost |
|---|---|---|---|
| Primary | `qwen/qwen3-coder:free` | OpenRouter | Free |
| Fallback 1 | `stepfun/step-3.5-flash:free` | OpenRouter | Free |
| Fallback 2 | `deepseek/deepseek-r1-0528:free` | OpenRouter | Free |
| Fallback 3 | `llama-3.3-70b-versatile` | Groq | Free |
| Fallback 4 | `qwen3-235b` | Cerebras | Free |
| Fallback 5 | `gemini-2.5-flash` | Google Gemini | Free |
| Fallback 6 | `Meta-Llama-3.1-405B-Instruct` | SambaNova | Free |
| Fallback 7 | `openai/gpt-oss-120b:free` | OpenRouter | Free |
| Fallback 8 | `mistral-small-latest` | Mistral | Free (1B tok/mo) |
| Heartbeat | `openai/gpt-oss-20b` | Groq | Free |
| Subagents | `qwen/qwen3-coder:free` | OpenRouter | Free |
| Image | `qwen/qwen3-vl-30b-a3b-thinking` | OpenRouter | Free |

Multi-provider failover across 6 providers (OpenRouter + Groq + Cerebras + Google Gemini + SambaNova + Mistral) gives ~20,000-25,000 free requests/day and ~1B+ free tokens/month at $0 cost. See [ANALYSIS.md](ANALYSIS.md) for full details and config snippets for both `openclaw.json` and `nix-claw`.

## License

MIT
