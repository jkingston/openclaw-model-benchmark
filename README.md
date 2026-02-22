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
| Primary | `deepseek/deepseek-v3.2` | OpenRouter | Free (rate-limited) / $0.25/$0.38 |
| Fallback 1 | `qwen/qwen3-coder-480b-a35b` | OpenRouter | Free |
| Fallback 2 | `llama-3.3-70b-versatile` | Groq | Free |
| Fallback 3 | `qwen3-235b` | Cerebras | Free |
| Fallback 4 | `stepfun/step-3.5-flash` | OpenRouter | Free |
| Fallback 5 | `mistral-small-latest` | Mistral | Free (1B tok/mo) |
| Heartbeat | `llama-3.1-8b-instant` | Groq | Free |
| Subagents | `qwen/qwen3-coder-480b-a35b` | OpenRouter | Free |
| Image | `meta-llama/llama-4-maverick` | OpenRouter | Free |

Multi-provider failover across OpenRouter + Groq + Cerebras + Mistral gives ~15,000-20,000 free requests/day and ~1B+ free tokens/month at $0 cost. See [ANALYSIS.md](ANALYSIS.md) for full details and config snippets for both `openclaw.json` and `nix-claw`.

## License

MIT
