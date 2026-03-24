---
title: "Decisions That Survive Contact With Code"
date: 2026-03-21
draft: true
tags: ["ai", "agents", "claude", "architecture", "go", "software-factory"]
categories: ["development", "ai"]
description: "Building a multi-agent software factory with Claude — Part 2 of 3. The six architectural decisions that had to be right before writing a single line of code."
cover:
  image: "/images/halalifarm/architectural_decisions_priority_map.svg"
  alt: "Architectural decisions priority map"
---

*Building a multi-agent software factory with Claude — Part 2 of 3*

---

In [Part 1](/blog/ai-agents-as-consultancy/), I described how I arrived at the idea for halalifarm — a software factory where AI agents collaborate to build multiple products. The mental model is a consultancy: the factory is the firm, each product is a client, and the agents are staff who move between clients.

Now comes the hard part: which decisions do you need to get right *before* writing code, and which can you safely evolve later?

This distinction matters more than usual with agent systems because the wrong early decision doesn't just mean refactoring — it means reprocessing hundreds of tasks across multiple projects. Migrating a task dependency format after you've got 50 tasks across 3 projects is not a fun afternoon.

## The Decision Map

Through the design conversation, Claude and I mapped every architectural decision into two buckets: "painful to change later" and "safe to evolve."

![Architectural decisions priority map](/images/halalifarm/architectural_decisions_priority_map.svg)

The right column — number of agents, model assignments, project templates, skill modules, review checkpoints — is all configuration. Adding a new agent is a JSON file. Swapping a model is changing one field. These are decisions you make once, learn from, and change freely.

The left column touches the core data structures and interfaces that everything else builds on. These are the load-bearing walls.

## Decision 1: How Do Agents Talk to Each Other?

When the PM consults the Architect and Security agent during planning, how does that conversation happen?

I was half joking when I suggested connecting them all to Slack, but the more I thought about it, it kinda made sense. Claude's counterpoint, however, was practical: bolting Slack into the critical path adds an external dependency that can fail, and it's overkill for the initial version. Claude suggested writing to a file and I thought while processes are running serially this might be fine but we will surely have race conditions as we speed things up:

> "What happens when everyone writes all at once?"

The answer: an **append-only message log**. It's a JSONL file per project — agents can only *add* messages, never edit previous ones. No race conditions possible.

```jsonl
{"ts":"2026-03-23T14:01:00Z","from":"pm","to":"architect","task":"task-3","msg":"Need input on auth approach"}
{"ts":"2026-03-23T14:01:12Z","from":"architect","to":"pm","task":"task-3","msg":"OAuth2 with PKCE, see decision doc"}
{"ts":"2026-03-23T14:01:30Z","from":"pm","to":"security","task":"task-3","msg":"Architect recommends OAuth2+PKCE, concerns?"}
```

Each agent reads the full log (or filtered by task) as part of its context, then writes its response as a new entry. When you eventually move to parallel execution, you just need a file lock on the append — one line of Go.

And here's the nice secondary benefit: you *could* mirror this to a real Slack channel as a read-only feed, so you can watch agents collaborate in real time. Slack becomes an observer, not a participant.

## Decision 2: Multi-Provider LLM Support

I knew from the start that I wanted different agents running on different models — Opus for security review, Sonnet for engineering, maybe GPT-4o-mini for lightweight naming checks. But I also wanted the flexibility to try Gemini or whatever comes next without rewriting the core.

The solution is a Go interface that every LLM backend implements:

```go
type Provider interface {
    Name() string
    Complete(ctx context.Context, req Request) (*Response, error)
    EstimateCost(req Request) CostEstimate
    CostPer1KTokens(model string) (float64, float64)
}
```

Each agent definition specifies its provider and model:

```json
{
  "name": "Security Engineer",
  "role": "security",
  "provider": "anthropic",
  "model": "claude-opus-4-20250514",
  "persona": "You are the Security Engineer for halalifarm..."
}
```

The factory doesn't care which provider an agent uses. It loads the definition, looks up the provider, and calls `Complete`. Adding a new provider later is just one more interface implementation.

## Decision 3: Cost Control That Actually Works

Here's the thing about running multiple AI agents across multiple projects: it's really easy to burn through API credits without realizing it. Especially when agents are collaborating — the PM asks the Architect a question, the Architect gives a detailed response, the PM synthesizes it and asks Security... that's three LLM calls just for one planning conversation.

I needed a system that makes cost *visible* and *controllable*.

![Provider and cost architecture](/images/halalifarm/halalifarm_provider_and_cost_architecture.svg)

Every LLM call flows through the same path: a **budget gate** checks limits before the call, the **provider router** dispatches to the right API, and the **token ledger** logs everything on the way back. The agent never talks to an API directly.

The budget gate enforces three limits:

```yaml
budgets:
  daily: 25.00
  per_task: 5.00
  per_project_monthly: 200.00
  warning_threshold: 0.80
```

Before every call, the gate checks: "will this estimated call push us over any limit?" If yes, it blocks and asks you. The factory physically cannot run up a huge bill without your approval.

But the real power is in the **context layer logging**. Every ledger entry records not just the total tokens, but *where* those tokens came from:

```json
{
  "agent": "engineer",
  "task": "task-42",
  "context_layers": {
    "persona": 340,
    "project_brief": 1200,
    "decisions": 890,
    "task_description": 450,
    "dependency_output": 2100,
    "messages": 600
  },
  "total_input_tokens": 5580,
  "cost": 0.042
}
```

Now you can ask: "Which context layer is eating the most tokens? Is loading all past decisions worth it, or are most of them irrelevant? Are we sending 2000 tokens of messages when only the last 3 matter?" You tune the system with data, not guesswork.

## Decision 4: The Task State Machine

Tasks need states. The question is how many. My instinct was more — a state is a word that carries a lot of meaning without a lot of explanation. If the whole point of the factory is to manage work, then the richer your state vocabulary, the more the system can tell you at a glance. I asked Claude whether there was any real cost to having more states — not dollar cost, but hidden complexity I hadn't considered.

The answer: not really. A state machine is just a string field and a transition table. The cost of adding a state is tiny; the cost of *not* having one when you need it — hunting through logs to figure out why a task stopped moving — is much higher.

So we went with the full set:

![Task state machine](/images/halalifarm/task_state_machine.svg)

Seven states, with clear transitions between them:

- **pending** → **ready**: all dependencies are met
- **ready** → **in_progress**: an agent picks it up
- **in_progress** → **in_review**: work is done
- **in_review** → **done**: QA approves it
- **in_review** → **in_progress**: rework needed (capped at 2-3 cycles)
- **in_progress** → **needs_human**: agent is stuck
- **pending** → **blocked**: a dependency failed, cascading downstream

The `needs_human` state is the escape valve. When an agent hits something it can't resolve — an ambiguous requirement, a decision that needs human judgment, a third-party API that's returning errors — it escalates instead of spinning. The rework cap between `in_review` and `in_progress` prevents infinite loops: if QA rejects work twice, it automatically escalates to `needs_human` rather than letting the agent try a third time.

The `blocked` state cascades. If task 3 fails, any task that depends on it (directly or transitively) gets blocked automatically. When task 3 is eventually fixed, the DAG re-evaluates and unblocks everything downstream.

## Decision 5: Task Dependencies as a DAG

When the PM breaks work into tasks, some tasks depend on others. "Write the API handler" can't start until "Design the API schema" is done. But "Write the frontend component" and "Write the API handler" might be independent.

We went with a lightweight DAG: each task has a `depends_on` field listing the IDs of tasks that must complete first.

```json
{
  "id": "task-5",
  "title": "Implement auth middleware",
  "agent": "engineer",
  "depends_on": ["task-2", "task-3"],
  "state": "pending"
}
```

The DAG resolver checks: are all of task-5's dependencies in the `done` state? If yes, task-5 transitions to `ready`. This is dead simple and it's enough for serial execution — the orchestrator just asks "what's ready?" and picks the highest-priority task.

When we eventually add parallel execution, the same DAG tells us which tasks *can* run simultaneously. Tasks with no unresolved dependencies are independent and can be dispatched to different agents in goroutines. The interface doesn't change — we just stop waiting.

## Decision 6: Context Assembly

This is the one Claude warned would bite hardest if I got it wrong. Every LLM call has a finite context window. When the Engineer agent is working on task 42, what gets loaded into its prompt?

We settled on a layered approach, in priority order:

1. **Agent persona** (always loaded) — who you are and how you work
2. **Project brief summary** (always loaded) — what we're building and why
3. **Relevant decisions** (loaded if they touch this task's domain) — past architectural choices
4. **Task description** (always loaded) — what specifically to do
5. **Dependency outputs** (loaded for tasks that built on prior work) — what the previous agent produced
6. **Message log** (recent messages relevant to this task) — what other agents have said

The factory assembles this context before each call. Too much and you waste tokens and confuse the model. Too little and the agent makes decisions that contradict earlier work.

The context layer logging I mentioned earlier is how you measure whether these choices are right. Over time, you look at the data and adjust — maybe you stop loading all decisions and only load ones tagged with the current task's domain. Maybe you trim the message log to the last 5 entries instead of all of them. The observability is built in from day one.

## What We Deliberately Deferred

Not everything needs an answer upfront. These are safe to evolve:

- **Number of agents**: Start with PM, Architect, Security, Engineer. Add QA, Ops, UX as needed.
- **Model assignments**: Start everything on Sonnet, then experiment with Opus for security and Haiku for lightweight reviews.
- **Project templates**: The brief template is a starting point. Refine it after onboarding the first real project.
- **Review checkpoints**: Add quality gates once the basic task flow works.
- **Skill modules**: Reusable knowledge can be added anytime without changing the core.

The principle: decide what's structural (data formats, interfaces, state machines) and defer what's configurable (which agents, which models, which templates).

---

**In Part 3**, we'll build the actual Go CLI — the `hf` command that ties everything together. We'll set up the project structure, implement the core types, and run a full end-to-end test: initializing the factory, pitching a project, switching contexts, and checking costs.

*[← Back to Part 1](/blog/ai-agents-as-consultancy/) · [Continue to Part 3: Bootstrapping a Factory That Builds Itself →](/blog/bootstrapping-factory-that-builds-itself/)*
