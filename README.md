# Dogma Skills

> Iterative critic-loop reasoning for AI coding agents. More loops = deeper thinking.

Two skills that make LLMs think harder by spending more inference compute — solve → critic → refine loops that halt when the critic finds nothing substantive.

## The Idea

Frontier models are theorized to use **recurrent-depth architectures** — transformer layers that loop in latent space, spending more compute on harder problems. Depth comes from iteration, not parameter count.

Standard LLMs produce answers in a single forward pass. For hard problems, that's not enough. These skills simulate recurrent-depth behavior at the **orchestration layer**:

| Architectural property | Orchestration analog |
|---|---|
| Looping layers in latent space | Solve → Critic → Refine cycles |
| MoE breadth across domains | Fan-out to 3 parallel agents with different framings |
| Adaptive halting (ACT) | Stop when critic finds nothing substantive |
| Expert routing / gating | Synthesis picks strongest claims, discards weak |

## The Problems

From [Andrej Karpathy](https://x.com/karpathy/status/2015883857489522876):

> "The models make wrong assumptions on your behalf and just run along with them without checking. They don't manage their confusion, don't seek clarifications, don't surface inconsistencies, don't present tradeoffs, don't push back when they should."

> "They really like to overcomplicate code and APIs, bloat abstractions, don't clean up dead code... implement a bloated construction over 1000 lines when 100 would do."

## The Solution

| Skill | Shape | Cost |
|-------|-------|------|
| **dogma** | Solve → Critic → Refine (up to 3 passes) | ~2–4x |
| **dogma-deep** | 3 parallel framings → Synthesize → Critic loop | ~5–8x |

Both embed four behavioral principles:

| Principle | Addresses |
|-----------|-----------|
| **Think Before Coding** | Wrong assumptions, hidden confusion, missing tradeoffs |
| **Simplicity First** | Overcomplication, bloated abstractions |
| **Surgical Changes** | Orthogonal edits, touching code you shouldn't |
| **Goal-Driven Execution** | Weak success criteria, no verification loop |

## Install

**Claude Code:**
```bash
mkdir -p ~/.claude/skills && cp skills/*.md ~/.claude/skills/
```

**GitHub Copilot (`.github/copilot-instructions.md`):**
```bash
cat skills/*.md >> .github/copilot-instructions.md
```

**OpenAI Codex (`.codex/instructions.md`):**
```bash
mkdir -p .codex && cat skills/*.md >> .codex/instructions.md
```

**Cursor:**
```bash
mkdir -p .cursor/rules && cp skills/*.md .cursor/rules/
```

**OpenCode (`opencode.md`):**
```bash
cat skills/*.md >> opencode.md
```

## Usage

Tell your AI agent:

- `dogma` or `/dogma` — single-track critic loop
- `dogma-deep` or `/dogma-deep` — fan-out + synthesis + critic
- `think deeper` — triggers dogma
- `deep dogma` — triggers dogma-deep

## How to Know It's Working

- Critic catches real issues (correctness, logic gaps, overcomplication) — not style nits
- Final answers are simpler than first drafts
- Refinements are surgical — every change traces to a specific critic issue
- Meta line shows iteration count and halt reason
- For dogma-deep: framings genuinely disagree, synthesis picks sides

## Key Insight

> "LLMs are exceptionally good at looping until they meet specific goals... Don't tell it what to do, give it success criteria and watch it go." — Karpathy

The critic loop captures this: transform a one-shot answer into a verified result by looping until an adversarial critic finds nothing substantive.

## Author

Andreas Heige

## License

MIT
