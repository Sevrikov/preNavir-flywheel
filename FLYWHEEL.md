# The Flywheel

> "It's not a single breakthrough. It's a loop that accelerates."

## Core Loop

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   SCOUT         SCORE         FORGE         TEST           │
│  (find it)  →  (evaluate) →  (build it) →  (verify)       │
│                                                ↓            │
│   RECALL        LEARN                        SAVE          │
│  (use it)  ←  (remember) ←───────────────────┘            │
│     ↓                                                       │
│   next SCOUT is smarter                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Why "Flywheel"

A flywheel is heavy and hard to start. But once spinning, each push costs less energy than the last. The flywheel stores momentum.

This system works the same way:
- **First revolution** is manual — human writes the stone, runs the test
- **Second revolution** uses `recall` — the system knows what worked before
- **Third revolution** uses `forge` — the system builds stones from templates
- **Nth revolution** — system finds tool, scores it, builds stone, tests it, remembers it, with minimal human input

## What Each Component Adds to the Flywheel

| Component | Contribution |
|-----------|-------------|
| `memory_core` | Stores what worked. Prevents re-inventing. Cross-session persistence. |
| `self_forge` | Removes the human from stone scaffolding. Templates → code in one call. |
| `source_scout` | Systematic discovery — 25+ domain queries, automatic scoring. |
| `test_protocol` | Measures idealism vs reality. Prevents phantom progress. |
| `SkillRunner` | Chains skills into protocols. One call, full battle rhythm. |

## The Friction Points (Where the Loop Can Break)

1. **Scoring is wrong** — bad tools get promoted, waste integration effort
2. **Stone generation fails** — template doesn't match tool's API pattern  
3. **Memory degrades** — wrong embedding strategy loses cross-lingual recall
4. **No verification** — stones pass tests but don't actually work with real data
5. **No soul** — system can act but can't reason about what to do next

## Acceleration Milestones

```
Day 0: Manual stone writing (100% human)
Day 1: self_forge templates (50% human)
Day 2: source_scout + scoring (25% human — finds and evaluates automatically)
Day 3: full_cycle protocol (10% human — end-to-end with one command)
Day N: scheduled loop (0% human — true autonomy)
```

## The "Idealism Score" Problem

Every system looks perfect in planning. The test protocol was built specifically to measure
how much of the architecture is **real** vs **theoretical**.

96.2% idealism score means: 25 out of 26 non-skipped tests pass on real hardware
with real Chrome, real SQLite, real Python imports, real subprocess calls.

The 3.8% gap = 1 warning (Gauntlet debug binary). Real but minor.

This metric matters more than any architecture diagram.
