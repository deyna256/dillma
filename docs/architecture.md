# dillma Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                            USER / FRONTEND                           │
└────────────────────────────────────┬────────────────────────────────┘
                                     │ HTTP/WebSocket
                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          API GATEWAY                                 │
│  • Request validation                                                │
│  • Event publishing/subscribing                                      │
└────────────────────────────────────┬────────────────────────────────┘
                                     │ Events
                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         EVENT BUS (Message Broker)                   │
│                                                                      │
│  Channels:                                                           │
│  • query.submitted                                                   │
│  • generation.start / generation.complete                            │
│  • evaluation.start / evaluation.complete                            │
│  • aggregation.start                                                 │
│  • result.ready                                                      │
└────────┬────────────┬─────────────┬─────────────┬───────────────────┘
         │            │             │             │
    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐  ┌─────▼─────┐
    │Orchestr-│  │  Model  │  │Evaluator│  │Aggregator │
    │  ator   │  │  Runner │  │         │  │           │
    └─────────┘  └─────────┘  └─────────┘  └───────────┘
```

## Components

### 1. API Gateway
**Role:** Entry point, user-facing communication

**Responsibilities:**
- Accepts HTTP/WebSocket requests
- Validates input data
- Publishes `query.submitted`
- Listens for `result.ready` and delivers the response to the user

---

### 2. Orchestrator
**Role:** Managing the request lifecycle

**Responsibilities:**
- Listens for `query.submitted`
- Loads a VotingMode from the config
- Executes the pipeline: generation → evaluation → aggregation
- Publishes events for the next phase

**Pluggable parts:**
```
VotingModeRegistry {
  "hierarchical": HierarchicalMode,
  "democratic": DemocraticMode,
  "tournament": TournamentMode,
  "hybrid": HybridMode
}
```

**VotingMode interface:**
```
get_pipeline(config) -> Pipeline
  Pipeline = [
    {phase: "generation", models: [...], parallel: true},
    {phase: "evaluation", strategy: "..."},
    {phase: "aggregation", strategy: "..."}
  ]
```

---

### 3. Model Runner
**Role:** LLM model queries

**Responsibilities:**
- Listens for `generation.start`
- Queries models in parallel via a provider (OpenRouter)
- Retry + timeout + rate limiting
- Publishes `generation.complete`

**Pluggable parts:**
```
ModelProviderRegistry {
  "openrouter": OpenRouterProvider,
  "ollama": OllamaProvider,
  ...
}
```

---

### 4. Evaluator
**Role:** Response quality assessment

**Responsibilities:**
- Listens for `evaluation.start`
- Applies the chosen evaluation strategy
- Publishes `evaluation.complete` (with scores)

**Pluggable parts:**
```
EvaluationStrategyRegistry {
  "weighted_voting": WeightedVotingStrategy,
  "ensemble": EnsembleStrategy,
  "argument_mining": ArgumentMiningStrategy,
  "two_phase": TwoPhaseStrategy
}
```

**EvaluationStrategy interface:**
```
evaluate(responses) -> Scores
```

---

### 5. Aggregator
**Role:** Final answer composition

**Responsibilities:**
- Listens for `aggregation.start`
- Applies the aggregation strategy
- Publishes `result.ready`

**Pluggable parts:**
```
AggregationStrategyRegistry {
  "winner_takes_all": WinnerTakesAllStrategy,
  "synthesis": SynthesisStrategy,
  "ranked_presentation": RankedPresentationStrategy
}
```

**AggregationStrategy interface:**
```
aggregate(responses, scores) -> FinalAnswer
```

---

## Data Flow Example

```
1. User → API Gateway
   Query: "Python vs Java?"
   Config: {voting_mode: "democratic", models: ["gpt-4", "claude", "gemini"]}

2. API Gateway → Event Bus
   Event: query.submitted {query_id, query, config}

3. Event Bus → Orchestrator
   Orchestrator loads DemocraticMode
   Pipeline: [generation, peer_review, evaluation, aggregation]

4. Orchestrator → Event Bus
   Event: generation.start {query_id, models: [...], type: "initial"}

5. Event Bus → Model Runner
   Parallel requests to 3 models
   Event: generation.complete {query_id, responses: [...]}

6. Orchestrator → Event Bus
   Event: generation.start {query_id, models: [...], type: "peer_review"}

7. Event Bus → Model Runner
   Each model reviews the others
   Event: generation.complete {query_id, reviews: [...]}

8. Orchestrator → Event Bus
   Event: evaluation.start {query_id, responses, reviews, strategy: "weighted"}

9. Event Bus → Evaluator
   Applies WeightedVotingStrategy
   Event: evaluation.complete {query_id, scores: {...}, winner: "gpt-4"}

10. Orchestrator → Event Bus
    Event: aggregation.start {query_id, responses, scores}

11. Event Bus → Aggregator
    Applies WinnerTakesAllStrategy
    Event: result.ready {query_id, final_answer, metadata}

12. Event Bus → API Gateway → User
    WebSocket: final answer + metadata
```

---

## Event Schema

```
BaseEvent {
  event_type: string        // Event type
  query_id: string          // Request UUID
  timestamp: datetime
  payload: {...}            // Event data
  metadata: {
    trace_id: string        // Distributed tracing
    parent_event_id: string // Event chain
  }
}
```

---

## Voting Modes

### Hierarchical
```
Pipeline:
1. Generation (parallel: all models)
2. Chairman Review (a designated chairman evaluates all responses)
3. Aggregation
```

### Democratic
```
Pipeline:
1. Generation (parallel: all models)
2. Peer Review (each model evaluates the others)
3. Evaluation (vote counting)
4. Aggregation
```

### Tournament
```
Pipeline:
1. Generation (all models)
2. Round 1: pairwise matchups → winners
3. Round 2: semifinals
4. Finals: ultimate winner
5. Aggregation
```

### Hybrid
```
Pipeline:
1. Query Categorization (determine query type)
2. Committee Generation (specialized groups)
3. Committee Consensus (local voting)
4. Arbiter Decision (final choice)
5. Aggregation
```

---

## Design Principles

### 1. Plugin Architecture
Every component uses the Registry Pattern:
- New mode = new file + registration
- New strategy = new file + registration
- No changes to core logic

### 2. Event-Driven
- Loose coupling between components
- Easy to add new components (just listen for events)
- Natural asynchrony
- Horizontal scalability

### 3. Separation of Concerns
- Orchestrator = coordination
- Model Runner = LLM queries only
- Evaluator = scoring only
- Aggregator = synthesis only

### 4. Configuration over Code
```yaml
voting_modes:
  hierarchical:
    enabled: true
    pipeline: [...]

models:
  available:
    - id: gpt-4-turbo
      provider: openrouter
```

