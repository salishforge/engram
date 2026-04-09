# Engram Research Plan

## Phase 1: Literature Survey

### Existing Benchmarks to Study
- [ ] **LongMemEval** (ICLR 2025) — 500 questions, 5 memory abilities, conversation histories. [Paper](https://arxiv.org/abs/2410.10813), [Code](https://github.com/xiaowu0162/LongMemEval)
- [ ] **LoCoMo** (ACL 2024) — Multi-session dialogue, 300+ turns, temporal reasoning. [Paper](https://arxiv.org/abs/2402.17753)
- [ ] **MemoryAgentBench** (ICLR 2026) — Multi-turn incremental, conflict resolution (<7% on all systems). [Paper](https://arxiv.org/abs/2507.05257), [HuggingFace](https://huggingface.co/datasets/ai-hyz/MemoryAgentBench)
- [ ] **MEMTRACK** (NeurIPS 2025) — Cross-platform state tracking (Slack/Git/Linear). [Paper](https://arxiv.org/abs/2510.01353)
- [ ] **Agent Memory Benchmark** — Aggregated leaderboard. [Site](https://agentmemorybenchmark.ai/)
- [ ] **Letta Leaderboard** — Read/write/update memory evaluation. [Blog](https://www.letta.com/blog/benchmarking-ai-agent-memory)
- [ ] **FABLE** — Forest-based adaptive retrieval. [Paper](https://arxiv.org/abs/2601.18116)

### Memory System Papers
- [ ] Complementary Learning Systems (McClelland et al., 1995) — Fast/slow learning theory
- [ ] Sparse Distributed Memory (Kanerva, 1988) — Distributed pattern storage
- [ ] Hierarchical Temporal Memory (Hawkins, 2004) — Sequence-based cortical learning
- [ ] Elastic Weight Consolidation (Kirkpatrick et al., 2017) — Preventing catastrophic forgetting
- [ ] Prioritized Experience Replay (Schaul et al., 2015) — Learning from surprising experiences
- [ ] REALM (Guu et al., 2020) — Retrieval-augmented language model pretraining
- [ ] RETRO (Borgeaud et al., 2022) — Retrieval-enhanced transformer
- [ ] MemWalker (Chen et al., 2023) — Interactive memory navigation

### Evaluation Methodology
- [ ] MTEB (Muennighoff et al., 2023) — Massive text embedding benchmark methodology
- [ ] RAGAS (Es et al., 2023) — RAG assessment framework
- [ ] Semantic similarity evaluation (BERTScore, semantic F1)
- [ ] LLM-as-judge methodology (GPT-4o agreement rates with human eval)

## Phase 2: Gap Analysis

For each existing benchmark, document:
1. What dimensions of memory quality does it measure?
2. What dimensions does it miss?
3. What task types does it cover?
4. How does it handle temporal evolution of knowledge?
5. Does it measure agent task performance (not just retrieval)?

Key gaps we expect to find:
- No benchmark tests **temporal accuracy** (returning the latest version of a fact)
- No benchmark tests **context efficiency** (tokens per correct answer)
- No benchmark tests **agent task performance delta** from memory
- MemoryAgentBench shows all systems achieve ≤7% on **multi-hop conflict resolution** — this is an unsolved problem
- No benchmark tests **long-horizon retention** (months of accumulated knowledge)

## Phase 3: Scenario Design

Design test scenarios that fill the identified gaps:

### Temporal Accuracy Scenarios
- Facts that change: "API endpoint moved from /v1 to /v2"
- Preferences that evolve: "I used to prefer Python, now I prefer Rust"
- Team changes: "Alice left the team, Bob is the new lead"
- Technology migrations: "We switched from MongoDB to PostgreSQL"

### Contradiction Resolution Scenarios
- Direct contradiction: "The server is on port 8080" vs "The server is on port 3000"
- Implicit contradiction: "We always use Docker" vs "The new project runs on bare metal"
- Temporal contradiction: "JWT auth" → "Session auth" (not really contradicting, just evolving)
- Source conflict: "Alice said X" but "the documentation says Y"

### Long-Horizon Retention Scenarios
- Critical early fact that must survive 1000 sessions of other data
- Rarely-accessed but important knowledge (disaster recovery procedure)
- Knowledge that compounds: early fact A + later fact B together answer a question neither could alone

### Agent Task Performance Scenarios
- Multi-session code review: agent reviews PRs over weeks, should learn team patterns
- Customer support: agent handles tickets over months, should improve response quality
- Research assistant: agent reads papers over time, should synthesize cross-paper insights

## Phase 4: Implementation

1. Build the adapter interface (Python, async)
2. Implement dataset loaders (YAML scenarios → structured test cases)
3. Build evaluators (exact match, semantic match, LLM judge, temporal precedence)
4. Build the runner (ingest → age → query → score)
5. Build the reporter (per-scenario, per-tier, composite Engram Score)
6. Implement MemForge adapter as reference implementation
7. Implement adapters for hippo, MemPalace, Zep, Letta
8. Run initial benchmarks and publish results

## Phase 5: Validation

- Compare Engram Tier 1 scores against published LongMemEval/LoCoMo results (should correlate)
- Validate LLM judge agreement with human evaluation (target >95%)
- Test-retest reliability: same system should score consistently across runs
- Publish methodology paper
