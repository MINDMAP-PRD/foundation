---
name: continuous-agent-loop
description: Use when designing or running autonomous or semi-autonomous agent workflows for implementation, review, verification, CI-style iteration, staged prompt pipelines, persistent sessions, parallel agents, merge coordination, or quality-gated development loops. Use this whenever the user wants a project, module, or foundation workstream to keep iterating with minimal supervision, whether the underlying tool is Codex, Claude Code, or another agent runner.
---

# Continuous Agent Loop

Patterns and architectures for autonomous and semi-autonomous agent loops.

This skill is tool-agnostic. It should work for Codex, Claude Code, and other agent runners.

If older notes or imported workflows refer to `autonomous-loops`, treat that as this skill. `continuous-agent-loop` is the canonical local skill name.

Use this skill to choose and document the right loop pattern, not to replace the governed package or the foundation ownership workflow.

## Read First

Before designing a loop for foundation-based work, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. the governed package, if one already exists
3. `../../foundations/README.md` when the project builds on the foundations
4. `../../foundations/FOUNDATION_MODULE_RULES.md` when the project builds on the foundations
5. `../../foundations/FOUNDATION_LIBRARY_ARCHITECTURE.md` when the loop affects module/foundation operating model
6. [`references/loop-patterns.md`](references/loop-patterns.md)

## When To Activate

Activate this skill when the task involves any of the following:

- autonomous implementation loops
- continuous build / review / verify cycles
- CI-style agent orchestration
- staged prompt pipelines
- persistent agent sessions
- parallel sub-agent execution
- merge coordination for parallel work
- context persistence across iterations
- de-sloppify or cleanup passes after implementation
- choosing between simple loops and DAG-style orchestration

## Core Role

This skill should:

- choose the lightest loop architecture that fits the job
- keep loop state explicit across iterations
- add quality gates, verification passes, and cleanup passes
- reduce context drift by using filesystem state and governed package state as continuity layers
- separate author, reviewer, verifier, and merger responsibilities when the workflow is complex enough to justify that split

This skill should not:

- bypass the governed package
- bypass foundation ownership checks
- assume one specific agent tool or CLI
- invent multi-agent complexity when a sequential loop is sufficient

## Foundation-Aware Rules

For foundation-based work:

1. Start with `foundation-module-factory` if the governed package does not exist yet.
2. Run `foundation-extension-auditor` before a loop is allowed to normalize shared-contract assumptions.
3. Treat the governed package as the continuity layer across loop iterations.
4. Keep module-local implementation inside the module unless promotion is explicitly approved.
5. Do not let autonomous loops silently patch the foundations as a side effect of module work.

## Default Workflow

1. Identify the work shape:
   - single focused change
   - repeated iterative improvement
   - spec-driven generation
   - long-running branch / PR loop
   - multi-unit parallel implementation
2. Identify the safety and coordination needs:
   - does the loop need quality gates?
   - does it need persistence between runs?
   - does it need parallelism?
   - does it need merge/conflict coordination?
3. Choose the simplest loop pattern that fits:
   - sequential pipeline
   - persistent session loop
   - infinite / wave-based generation loop
   - continuous PR loop
   - RFC / DAG orchestration
4. Define the loop contract:
   - trigger
   - step order
   - context bridge
   - exit conditions
   - cost / duration / retry bounds
   - verification gates
   - cleanup / de-sloppify pass
5. Record where progress lives:
   - governed package progress files
   - shared task notes
   - branch / worktree state
   - loop status file if needed
6. If the loop touches implementation quality, pair with `coding-standards`.
7. If the loop touches security-sensitive behavior, pair with `secure-coding-practices` or the appropriate security specialist.

## Non-Negotiable Rules

1. Every loop must have an exit condition.
2. Every loop must have a context bridge between iterations.
3. Every loop must have an explicit quality gate before merge or completion.
4. Parallel agents must have a merge or conflict strategy.
5. Cleanup passes should be separate steps instead of burying negative instructions inside implementer prompts.
6. Use separate agents or separate passes for authoring and review when the stakes justify it.
7. Prefer simple sequential loops unless the work clearly benefits from persistent sessions, PR iteration, or DAG orchestration.

## Output Expectations

When this skill is used, it should leave behind one or more of the following:

- a chosen loop architecture with rationale
- a loop contract or runbook
- a staged automation plan for implementation / review / verify / merge
- a persistence plan for progress and context
- explicit quality-gate and exit-condition rules

## Reference Guide

Use [`references/loop-patterns.md`](references/loop-patterns.md) for the detailed pattern catalog, decision matrix, and tool-agnostic examples.
