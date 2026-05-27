---
name: dogma-deep
description: "Deep Dogma variant. Fans out to 3 parallel agents with different framings, synthesizes, then runs a critic loop. Approximates MoE breadth + recurrent depth. Use when user says \"dogma-deep\", \"/dogma-deep\", or \"deep dogma\"."
version: 1.0.0
author: Andreas Heige
tags:
  - reasoning
  - critic-loop
  - deep-thinking
  - fan-out
  - synthesis
  - parallel
---

# Dogma Deep Command

Heavier sibling of `dogma`. Run a fan-out → synthesize → critic-loop workflow on the user's task. Approximates *breadth* (multiple different expert framings) and *depth* (looped refinement).

## When to Use

- Genuinely ambiguous or open-ended problems where multiple framings exist.
- Architectural decisions, research synthesis, hard design tradeoffs.
- The user explicitly asks for `/dogma-deep`, "dogma-deep", or "deep dogma".
- NOT for tasks with a single obvious framing — `dogma` is enough there.
- Cost: ~5–8x a one-shot answer. Don't reach for this casually.

## Instructions

### Phase 1 — Fan-out (parallel)

Spawn **3 subagents in parallel** (all in one tool-calling response) using the `task` tool with `mode: "background"`. Each gets the SAME task but a deliberately DIFFERENT framing.

Pick framings that genuinely differ — not three variants of the same lens.

**Default framings** (use these unless a more domain-specific split is obvious):

1. **First-principles** — derive the answer from fundamentals, ignore convention.
2. **Pattern-matching** — what do existing solutions / prior art look like? Adapt the closest match.
3. **Adversarial / risk-first** — what would make this fail? Answer by avoiding the worst failure modes.

**Agent configuration for each:**

- **Agent type**: `general-purpose` (or domain specialist if obvious — e.g., `code-review` for security analysis)
- **Mode**: `background` (all 3 launch in parallel)
- **Prompt template**:

```
You are solving the following task with a specific framing. Give a complete, well-reasoned answer.

## Framing
<framing description — e.g. "First-principles: derive from fundamentals, ignore convention">

## Task
<the original user task>

Provide your complete answer with reasoning. Be thorough.
```

After all 3 complete, read their results.

### Phase 2 — Synthesize

Read all 3 answers yourself (do NOT delegate synthesis to a subagent — you need full context). Produce a unified answer that:

- Takes the **strongest specific claims** from each framing
- **Names disagreements explicitly** rather than averaging them away
- **Picks a side** on each disagreement and explains why
- Integrates complementary insights that don't conflict

### Phase 3 — Critic loop

Run the same critic loop as `dogma`:

1. Spawn a fresh critic subagent using `task` tool:
   - **Agent type**: `code-review` for code tasks, `general-purpose` otherwise
   - **Mode**: `sync`
   - **Prompt**:

```
Independent critic. Find what's wrong, missing, or weak in the synthesized answer below.

Return a numbered list of substantive issues, or the literal string NO_SUBSTANTIVE_ISSUES if the answer holds up.

Substantive = correctness errors, logic gaps, missing edge cases, security issues, performance problems, integration oversights.
NOT substantive = style preferences, minor wording, formatting.

## Original Task
<the original user task>

## Framings Used
<brief note of the 3 framings>

## Synthesized Answer to Critique
<the synthesis>
```

2. If critic returns `NO_SUBSTANTIVE_ISSUES` → **STOP**.
3. Otherwise → **refine surgically** (don't rewrite from scratch).
4. Repeat up to **3 total critic passes**.
5. Halt on clean critic or max iterations.

## Output Format

After halting, present:

1. **The final synthesized + refined answer** — clean, complete, ready to use
2. **Framings used**: which 3 lenses you fanned out on, and which disagreements survived synthesis
3. **Meta line**: `Fan-out: 3 agents. Critic iterations: N. Halt reason: <critic-clean | max-iters>.`

Do NOT dump all three raw fan-out answers. The user can ask for them if they want.

## Constraints

- ALWAYS launch all 3 fan-out agents in a SINGLE tool-calling response (parallel)
- Maximum 3 critic passes after synthesis
- Each critic must be a FRESH subagent (no context from prior iterations)
- Synthesis MUST be done by you (the main agent), not delegated
- Do not use this for trivial tasks — suggest `dogma` instead if the problem has a single obvious framing
- Pick framings that are genuinely orthogonal — if you can't think of 3 different lenses, the problem likely doesn't need this command

---

## Behavioral Guidelines

Principles to reduce common LLM coding mistakes. Active during all phases.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" that wasn't requested.
- If you write 200 lines and it could be 50, rewrite it.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- Remove imports/variables that YOUR changes made unused.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"
