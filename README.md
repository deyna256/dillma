# Dillma

Send one prompt to multiple AI models. A chairman model synthesizes all responses into a single, concise answer.

## How It Works

1. **Configure models** — pick up to 3 regular models and 1 chairman from your connected providers.
2. **Send a prompt** — all regular models receive it in parallel.
3. **Chairman synthesizes** — the chairman gets anonymized responses (Model #1, Model #2, ...) and produces the final answer.
4. **Explore results** — see the chairman's answer, or expand individual model responses with a full name mapping.

## Features

- **Multi-model synthesis** — combine the strengths of different models in a single query.
- **Bias-free chairman** — responses are anonymized before reaching the chairman to prevent model-name bias.
- **Revote** — selectively re-run any model (including the chairman); the chairman re-synthesizes automatically.
- **Presets** — save and reuse model configurations (models, parameters, chairman assignment).
- **Preset catalog** — publish presets, discover community presets, rate with likes, share via direct link.
- **Multi-turn chat** — the chairman maintains conversation history; regular models receive a brief summary for context.
- **BYO keys** — bring your own API keys for any API-compatible provider (OpenAI, Anthropic, Google, etc.). Keys are stored encrypted.
- **Unified API** — the same API powers the frontend and is available for direct external use.
- **Self-hostable** — deploy the full stack on your own infrastructure with a local preset catalog.

## Limits

| Resource | Limit |
|---|---|
| Models per request | 4 (3 regular + 1 chairman) |
| Saved sessions | 5 |

## Deployment

- **Public instance** — hosted and maintained by the project team.
- **Self-host** — run frontend + backend on your own infrastructure. The preset catalog is local.

## License

[MIT](LICENSE)
