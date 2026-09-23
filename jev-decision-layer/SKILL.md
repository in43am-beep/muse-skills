---
name: jev-decision-layer
description: >
  Decision-routing layer for multi-agent and multi-step work: turn a messy agent
  graph into a controlled loop of state → scored routes → confidence threshold →
  execute or die → verify winners → next state. Use when a task has branching
  decisions (which worker/model/tool runs next, is a result good enough, does
  this need approval), when parallel workers must not improvise, or when agent
  loops are burning expensive model calls on yes/no decisions. Separates
  reasoning (expensive model) from decision-making (cheap explicit routing) so
  decisions become measurable, fast, and cheap.
---

# Jev Decision Layer

Learned from @0xRicker's "Jev Engineering" pattern (decision layer for agent
swarms). The core insight: **a swarm knows how to execute, but not when to
stop, what to run next, or whether a result is good enough. Those are
decisions — and they should not cost an LLM call each.**

## The loop

```
state → Jev decision layer → parallel routes → execution → verification → next state
```

Expanded:

1. **State enters** — a structured state object (task, constraints, prior
   results), not prose.
2. **Routes get scored** — enumerate the candidate actions (which worker, which
   model, which tool, skip, escalate). Score each against explicit criteria.
3. **Confidence threshold decides** — routes above the threshold execute; the
   rest die. No LLM deliberation per route.
4. **Cheapest capable executes** — the surviving route runs on the weakest
   model/tool that can do the job. Expensive models only touch decisions that
   survived the filter.
5. **Winners get verified** — every output is checked against acceptance
   criteria (machine-verifiable where possible: file exists, test passes, count
   matches, checksum). Unverified results do not advance.
6. **New state emerges** — verified results collapse into the next state; loop
   repeats until a stop condition is met.

## Rules

- **Decisions don't generate prose.** The decision layer routes → scores →
  blocks → approves. It never writes paragraphs about what to do.
- **Separate reasoning from decision-making.** LLM reasons about the task once;
  the decision layer handles every yes/no, pick-next, and good-enough call.
- **One rulebook.** Thresholds, scoring criteria, and stop conditions are
  written down before the loop starts — not improvised per branch.
- **Parallelism without improvisation.** Branches may fan out, but none gets
  permission to change the plan, expand scope, or invent new routes.
- **Verify before completion.** A result is done only when it passes its
  checks, never when a worker says it finished.
- **Make the layer measurable.** Track per-loop: decisions made, cost per
  decision, latency, kill rate of bad routes. (Source claims up to 193x faster /
  444x cheaper vs LLM-every-loop in their tests — treat as their benchmark,
  measure your own.)

## Execution gates

Before any irreversible or expensive action (sends, publishes, purchases,
deletes), the route must pass an explicit gate: is approval present, are
prerequisites verified, is this the cheapest capable path? Blocked routes die
with a logged reason — they never silently retry.

## Anti-patterns

- LLM → agent → tool → another LLM → another guess (every decision is an
  expensive call).
- A worker deciding on its own to expand scope, re-run, or "improve" beyond
  the brief.
- Marking work complete from intent ("task spawned") instead of verified state
  ("artifact exists and passes checks").
- Routing everything through the strongest model because it's easier than
  scoring.

## Applying it in this workspace

- **Multi-worker production runs** (e.g. 110 Etsy products): coordinator holds
  the state; workers get scored briefs; QA checks are the verification gate;
  nothing is reported done until the file count matches.
- **Browser/purchase flows**: the decision layer is the confirmation boundary —
  prepare, then gate, then execute only on explicit approval.
- **Research tasks**: score sources (freshness, authority) before reading;
  kill low-value routes early; verify claims against a second source before
  they enter state.
