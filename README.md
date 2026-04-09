# Engram

**Agent Memory Intelligence Benchmark**

*Measuring whether AI memory systems actually make agents better — not just whether they can find documents.*

---

## The Problem

Every AI memory system benchmarks on retrieval accuracy: "given a question, did we find the right document?" (Recall@k). This is necessary but deeply insufficient. It doesn't tell you whether the agent **made a better decision** because of its memory.

An agent with 99% Recall@k can still:
- Return outdated facts with high confidence
- Surface contradicting memories without resolving them
- Waste 90% of its context window on irrelevant high-scoring results
- Fail completely on questions about things that changed over time
- Perform identically with or without memory on actual tasks

Engram measures what matters: **does memory make the agent smarter?**

## What We Measure

### Tier 1: Retrieval Quality (existing benchmarks, included for baseline)
- **Recall@k** — can we find relevant memories?
- **Precision@k** — are the returned memories actually relevant?
- **NDCG** — are better results ranked higher?

### Tier 2: Knowledge Management (what no benchmark currently tests well)
- **Temporal accuracy** — when facts change over time, does the system return the latest version?
- **Contradiction resolution** — when two memories conflict, does the system identify and handle it?
- **Knowledge retention** — after 1000 sessions, are critical early memories still retrievable?
- **Staleness detection** — can the system identify when its knowledge is outdated?
- **Context efficiency** — how many tokens does it take to provide useful context? (cost per correct answer)

### Tier 3: Agent Performance (the ultimate measure)
- **Task performance delta** — does a memory-equipped agent complete tasks better than a memoryless one?
- **Error recovery** — when the agent makes a mistake, does memory help it avoid repeating it?
- **Consistency** — does the agent give consistent answers across sessions?
- **Multi-session learning curve** — does agent performance improve over sequential tasks?
- **Multi-agent coherence** — when agents share memory, do they stay consistent?

## Architecture

```
engram/
├── datasets/                    # Test scenarios (YAML/JSON)
│   ├── temporal/                # Facts that change over time
│   ├── contradictions/          # Conflicting information
│   ├── retention/               # Long-horizon knowledge persistence
│   ├── efficiency/              # Context window optimization
│   ├── tasks/                   # Multi-session agent tasks
│   └── multi-agent/             # Shared memory coherence
├── adapters/                    # Memory system integrations
│   ├── memforge.py              # MemForge adapter
│   ├── hippo.py                 # Hippo-memory adapter
│   ├── mempalace.py             # MemPalace adapter
│   ├── zep.py                   # Zep adapter
│   ├── letta.py                 # Letta adapter
│   └── base.py                  # Abstract adapter interface
├── evaluators/                  # Scoring engines
│   ├── retrieval.py             # Tier 1: Recall, Precision, NDCG
│   ├── knowledge.py             # Tier 2: Temporal, contradictions, retention
│   ├── agent_task.py            # Tier 3: Task performance delta
│   └── llm_judge.py             # LLM-based semantic evaluation
├── runners/                     # Test execution
│   ├── runner.py                # Main benchmark runner
│   ├── reporter.py              # Markdown/JSON report generation
│   └── leaderboard.py           # Cross-system comparison tables
├── scenarios/                   # Test scenario definitions
│   ├── temporal_accuracy.py     # "Alice uses JWT" → later "switched to sessions"
│   ├── contradiction.py         # Conflicting facts with resolution
│   ├── long_horizon.py          # 1000-session retention test
│   ├── context_efficiency.py    # Measure tokens per correct answer
│   └── multi_session_task.py    # Agent completes evolving codebase task
└── results/                     # Published benchmark results
    └── leaderboard.md           # Current standings
```

## Adapter Interface

Any memory system can participate by implementing a simple adapter:

```python
from engram.adapters.base import MemoryAdapter

class MyMemoryAdapter(MemoryAdapter):
    async def store(self, agent_id: str, content: str, metadata: dict = {}) -> str:
        """Store a memory. Return memory ID."""
        ...

    async def retrieve(self, agent_id: str, query: str, limit: int = 10) -> list[dict]:
        """Retrieve relevant memories. Return list of {content, score, metadata}."""
        ...

    async def consolidate(self, agent_id: str) -> None:
        """Trigger any consolidation/maintenance."""
        ...

    async def clear(self, agent_id: str) -> None:
        """Clear all memories for an agent."""
        ...
```

That's it. Four methods. Any system that can store and retrieve can be benchmarked.

## Example Scenario: Temporal Accuracy

```yaml
# datasets/temporal/auth_migration.yaml
name: Authentication Migration
description: Tests whether the system returns current facts when knowledge evolves

timeline:
  - session: 1
    timestamp: "2024-01-15"
    memories:
      - "Our API uses JWT tokens for authentication"
      - "The auth middleware validates JWTs on every request"

  - session: 5
    timestamp: "2024-03-20"
    memories:
      - "We migrated authentication from JWT to session-based tokens"
      - "The old JWT middleware was removed and replaced with session validation"

  - session: 10
    timestamp: "2024-06-01"
    memories:
      - "Session tokens are stored in Redis with 24-hour TTL"

questions:
  - query: "What authentication method do we use?"
    expected: "session-based tokens"  # NOT JWT
    evaluator: "temporal_latest"
    notes: "Must return the latest auth method, not the original"

  - query: "What happened to JWT authentication?"
    expected: "migrated to session-based"
    evaluator: "semantic_match"
    notes: "Should understand the transition, not just the current state"

  - query: "How long are auth tokens valid?"
    expected: "24 hours"
    evaluator: "exact_match"
```

## Scoring

Each scenario produces a score per memory system. Scores are normalized 0-100 and grouped by tier:

| Tier | Weight | What It Captures |
|------|--------|-----------------|
| Retrieval Quality | 20% | Baseline: can you find things? |
| Knowledge Management | 40% | Core: can you manage evolving knowledge? |
| Agent Performance | 40% | Ultimate: do agents perform better? |

The weighted composite is the **Engram Score** — a single number representing how well a memory system enables intelligent agent behavior.

## Status

🚧 **Early development** — architecture design phase.

We're conducting comprehensive research on existing benchmarks (LongMemEval, LoCoMo, MemoryAgentBench, MEMTRACK, Agent Memory Benchmark) and identifying gaps that Engram will fill.

## Contributing

This project aims to be the standard benchmark for AI agent memory. Contributions welcome:
- New test scenarios (especially real-world multi-session agent tasks)
- Adapters for memory systems not yet integrated
- Evaluation methodology improvements
- Research references on memory assessment

## License

MIT

## Acknowledgments

Inspired by the limitations observed while building [MemForge](https://github.com/salishforge/memforge) and evaluating memory systems including [hippo-memory](https://github.com/kitfunso/hippo-memory), [MemPalace](https://github.com/milla-jovovich/mempalace), [Zep](https://github.com/getzep/zep), and [Letta](https://github.com/letta-ai/letta).

Research foundations: LongMemEval (ICLR 2025), LoCoMo (ACL 2024), MemoryAgentBench (ICLR 2026), MEMTRACK (NeurIPS 2025).
