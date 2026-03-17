# Loop Patterns

Use this reference to choose and document autonomous or semi-autonomous agent loops.

This guide is tool-agnostic. Map the steps to Codex, Claude Code, CI runners, local scripts, or another agent framework as needed.

## 1. Pattern Spectrum

| Pattern | Complexity | Best For |
|---|---|---|
| Sequential Pipeline | Low | Daily dev steps, scripted workflows, straightforward implementation |
| Persistent Session Loop | Low | Interactive ongoing work with session continuity |
| Infinite / Wave Loop | Medium | Repeated generation, exploration, spec-driven variations |
| Continuous PR Loop | Medium | Multi-iteration implementation with branch, CI, and merge gates |
| De-Sloppify Pass | Add-on | Cleanup after any implementation step |
| RFC / DAG Orchestration | High | Large feature sets with parallel work units and merge coordination |

## 2. Decision Matrix

Use the lightest viable option.

```text
Is the task a single focused change?
├─ Yes -> Sequential pipeline or persistent session loop
└─ No -> Is there a written spec or governed package?
         ├─ Yes -> Do you need parallel implementation?
         │        ├─ Yes -> RFC / DAG orchestration
         │        └─ No -> Continuous PR loop
         └─ No -> Do you need repeated variations or exploratory output?
                  ├─ Yes -> Infinite / wave loop
                  └─ No -> Sequential pipeline with a cleanup pass
```

## 3. Sequential Pipeline

The simplest loop.

Use it when:

- the work can be done in a clear ordered sequence
- each step can rely on filesystem state from the previous step
- parallelism is unnecessary

### Shape

1. implement
2. cleanup / de-sloppify
3. verify
4. commit or checkpoint

### Tool-Agnostic Example

```text
step 1: run agent with "Implement the feature from the governed package"
step 2: run agent with "Cleanup unnecessary complexity and keep real logic"
step 3: run agent or CI with "Run build, lint, type-check, and tests"
step 4: checkpoint or commit
```

### Design Rules

- each step gets a fresh or intentionally bounded context
- filesystem state is the bridge
- failures stop the sequence until resolved

## 4. Persistent Session Loop

Use when:

- you want continuity across turns
- exploration is interactive but still structured
- context growth is acceptable and managed

### Design Rules

- persist history to a file or durable session store
- checkpoint important decisions into the governed package, not only the live session
- reset or summarize the session before it becomes noisy

## 5. Infinite / Wave Loop

Use when:

- the loop is generating many iterations or variants
- each agent needs a distinct direction
- the orchestrator can assign uniqueness deliberately

### Design Rules

- assign iteration numbers explicitly
- assign unique creative or technical directions explicitly
- batch or wave agents instead of leaving the system fully unbounded
- stop on explicit cost, count, duration, or quality thresholds

## 6. Continuous PR Loop

Use when:

- the work spans many iterations
- each iteration should land in a branch or PR
- CI results should feed the next iteration

### Shape

1. create branch or worktree
2. run implementation agent
3. optionally run separate review pass
4. commit and push
5. open or update PR
6. wait for CI
7. on failure, feed CI output back into a fix pass
8. merge when gates pass
9. repeat until exit condition is hit

### Required Elements

- cost or run limit
- duration limit
- completion signal or completion threshold
- CI retry policy
- shared task notes or package progress update

## 7. De-Sloppify Pass

Use this after implementation when the model tends to overbuild.

### Why It Exists

Negative instructions inside implementation prompts often degrade good behavior along with bad behavior.

A separate cleanup pass is safer.

### Cleanup Targets

- tests that only prove framework or language behavior
- redundant defensive checks for impossible states
- console logging and dead scaffolding
- commented-out code
- unnecessary abstraction added too early

Keep legitimate business-logic tests and meaningful edge-case handling.

## 8. RFC / DAG Orchestration

Use when:

- the feature can be decomposed into multiple work units
- some units can run in parallel
- merge conflicts or dependency order matter

### Shape

1. read RFC / package
2. decompose into work units
3. define dependency DAG
4. assign each unit to a worktree or isolated branch
5. run unit pipelines by DAG layer
6. review and verify units independently
7. land through a merge queue with conflict handling

### Work Unit Shape

- `id`
- `name`
- `description`
- `dependencies`
- `acceptance criteria`
- `complexity tier`

### Tiering

- trivial: implement -> test
- small: implement -> test -> review
- medium: research -> plan -> implement -> test -> review -> fix
- large: research -> plan -> implement -> test -> multi-review -> fix -> final review

## 9. Quality Gates

Every loop should define:

- build gate
- lint / type-check gate
- test gate
- review gate when risk is non-trivial
- package-progress update gate

For foundation-based work, the loop should also respect:

- foundation ownership rules
- governed package scope
- module-local vs promotion boundaries

## 10. Context Bridges

Each loop iteration needs a continuity mechanism.

Use one or more of:

- governed package progress files
- `SHARED_TASK_NOTES.md`
- branch or worktree state
- loop-status JSON or Markdown
- explicit artifact outputs from one stage to the next

Do not rely on memory alone.

## 11. Anti-Patterns

Avoid these:

- loops with no exit condition
- retries with no new context
- parallel agents with no merge plan
- negative instructions instead of cleanup passes
- treating the same agent context as implementer, reviewer, and verifier for high-risk work
- letting autonomous loops change foundation ownership without explicit approval

## 12. Foundation Integration

For foundation-based projects:

- create or resume the governed package first
- run ownership analysis before autonomous implementation
- keep package progress updated after each loop cycle
- treat autonomous execution as downstream of the package, not a replacement for it
