---
name: dogma
description: Iterative critic-loop reasoning. Simulates "more loops = deeper thinking" via solve → critic → refine, halting when the critic finds nothing substantive. Use when the user says "dogma", "/dogma", or asks for deep/iterative reasoning on a hard problem.
version: 1.0.0
author: Andreas Heige
tags:
  - reasoning
  - critic-loop
  - deep-thinking
  - iterative
---

# Dogma Command

Run a single-track critic loop on the user's task. The goal is to spend more inference compute per problem — more iterations, deeper thinking — by running solve → critic → refine cycles until the critic finds nothing substantive.

## When to Use

- Hard reasoning, design, or coding problems where one-shot answers feel shallow.
- The user explicitly asks for `/dogma` or "dogma" or "think deeper".
- NOT for trivial tasks — the loop adds 2–4x latency and token cost.

## Instructions

### Phase 1 — Solve (iteration 0)

Produce an initial answer to the user's task. Think carefully and produce the answer with reasoning visible. Use extended thinking. Write out your full reasoning.

### Phase 2 — Critic pass

Spawn a fresh subagent using the `task` tool with this configuration:

- **Agent type**: `code-review` for code tasks, `general-purpose` otherwise
- **Mode**: `sync`
- **Prompt template**:

```
Independent critic. Do NOT defend or extend the answer below — find what's wrong, missing, or weak.

Return a numbered list of substantive issues, or the literal string NO_SUBSTANTIVE_ISSUES if the answer holds up.

Substantive = correctness errors, logic gaps, missing edge cases, security issues, performance problems.
NOT substantive = style preferences, minor wording, formatting.

## Task
<the original user task>

## Answer to Critique
<the current answer>
```

### Phase 3 — Halt check

- If the critic returns `NO_SUBSTANTIVE_ISSUES` or only stylistic nits → **STOP** and present the current answer.
- Otherwise → continue to refine.

### Phase 4 — Refine

Address the critic's substantive issues. Do NOT rewrite from scratch — surgically revise only what the critic flagged.

### Phase 5 — Repeat

Repeat phases 2–4 up to **3 total critic passes**. If the answer is still contested after 3 iterations, present the final version plus an "unresolved tensions" note.

## Output Format

After halting, present:

1. **The final answer** — clean, complete, ready to use
2. **Meta line**: `Iterations: N. Halt reason: <critic-clean | max-iters>.`
3. **Unresolved tensions** (only if halt reason is max-iters): brief list of issues the critic still flagged

Do NOT output every intermediate draft — just the final answer plus the meta line. The user can ask to see critic transcripts if they want them.

## Constraints

- Maximum 3 critic passes — do not loop forever
- Each critic must be a FRESH subagent (no context leakage from prior iterations)
- Critic must be adversarial — never spawn a critic that has seen the solve reasoning
- Do not use this for trivial lookups or simple factual questions
- If the task is code-related, the critic agent should be `code-review` type
- If the task is general reasoning/design, the critic agent should be `general-purpose` type

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
