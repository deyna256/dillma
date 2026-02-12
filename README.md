# dillma

LLM consensus engine — get better answers by making multiple models debate, review, and vote.

## The idea

No single LLM is the best at everything. dillma sends the same query to multiple models, then has them evaluate each other's responses and picks the winner through a configurable voting process. Think of it as a panel of experts instead of a single advisor.

## How it works

1. **Generate** — multiple LLM models answer the same question in parallel
2. **Evaluate** — models review and score each other's responses
3. **Aggregate** — a final answer is selected (or synthesized) based on the scores

The entire pipeline is configurable. You choose the models, the voting mode, and the aggregation strategy.

## Voting modes

| Mode | Description |
|---|---|
| **Hierarchical** | A designated "chairman" model evaluates all responses and picks the best one |
| **Democratic** | Every model reviews every other model's response; votes are tallied |
| **Tournament** | Pairwise elimination brackets until a single winner remains |
| **Hybrid** | Query is categorized first, then routed to specialized model committees |

## Architecture

Event-driven, plugin-based. Every component (orchestrator, model runner, evaluator, aggregator) communicates through an event bus and is independently replaceable.

See [docs/architecture.md](docs/architecture.md) for the full design.

## License

[MIT](LICENSE)
