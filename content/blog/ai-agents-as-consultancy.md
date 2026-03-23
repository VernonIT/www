+++
title = "What If Your AI Agents Were a Consultancy?"
date = 2026-03-22
draft = false
tags = ["ai", "agents", "claude", "architecture", "software-factory"]
categories = ["development", "ai"]
description = "Building a multi-agent software factory with Claude — Part 1 of 3. The mental model that unlocked the architecture: the factory is the firm, each product is a client, and the agents are the staff."
[cover]
  image = "/images/halalifarm/halalifarm_factory_architecture.svg"
  alt = "Halalifarm factory architecture diagram"
+++

*Building a multi-agent software factory with Claude — Part 1 of 3*

---

I've been circling the same idea for months: what if I could pitch a product idea to an AI team and have them plan it, architect it, build it, and review it — like a real dev team, but one that lives in my terminal?

Not a single chatbot that writes code. A *team*. A Project Manager who breaks down the work. An Architect who designs the system. A Security Engineer who pokes holes in it. A Software Engineer who builds it. QA who makes sure it works. Ops who makes sure it ships.

I kept getting stuck on one thing: how do you tell a team of AI agents to build something that isn't itself?

This is the story of how I figured that out, and it started with someone else's project.

## Starting Point: bdfinst's Agentic Dev Team

I stumbled across [bdfinst/agentic-dev-team](https://github.com/bdfinst/agentic-dev-team) on GitHub — a Claude Code plugin that adds a full persona-driven AI development team to any project. It's clever: an Orchestrator routes tasks to specialized agents, review checkpoints catch quality issues during implementation, and reusable skill modules give agents domain knowledge.

The agents share memory through a simple file-based system — a `memory/` directory that all agents can read from and write to. There's also integration with [Beads](https://github.com/beads-dev/beads), a git-backed issue tracker that gives agents structured task context across sessions.

It's pragmatic and well-designed. But it solves a different problem than mine.

bdfinst's project enhances a *single repository*. The agents live inside the repo they're working on. They improve the codebase they sit in.

My problem was different. I don't have one project. I have *several* — dashcode, 1lid, vida, and more in the pipeline. I needed something that could manage all of them.

## The Insight: The Factory Isn't the Product

Here's where the conversation with Claude got interesting. I was explaining my project — halalifarm — and Claude initially assumed it was the product I wanted the agents to build. I had to correct that:

> "Actually, halalifarm *is* the software factory. The projects are things like dashcode, 1lid, vida..."

That reframed everything. Halalifarm isn't a product the agents build. It's the *firm* that employs the agents. When I say "build dashcode," the factory spins up a project context, creates or navigates to the dashcode repo, and the agents do their work *there*.

The mental model clicked when Claude described it as a consultancy:

> "Halalifarm is the firm. dashcode, 1lid, vida are the clients. Each client has its own folder, its own brief, its own backlog. The agents are the staff who move between clients but always know which client they're currently working for."

![Halalifarm factory architecture](/images/halalifarm/halalifarm_factory_architecture.svg)

This diagram shows the full picture. You pitch an idea to the factory. The PM agent receives it, consults with SME agents, and produces a plan. The project registry tracks all active projects. And each project lives in its own separate workspace — completely decoupled from the factory itself.

## The Self-Improvement Paradox

Once we had this architecture, I raised a question that had been nagging me: if halalifarm is a software factory, and software factories need improvement... shouldn't halalifarm be able to improve *itself* using its own process?

> "Sometimes you work the work. Sometimes you work on the business. And since the business is a product itself, there should be some way to use our own processes to make it better."

The solution was elegant: halalifarm is just another entry in its own project registry. It's a project like any other, with its own brief, its own backlog, its own task list. The only difference is the `type` field:

```json
{
  "halalifarm": {
    "workspace": ".",
    "type": "factory-meta",
    "brief": "docs/project-brief.md"
  },
  "dashcode": {
    "workspace": "~/projects/dashcode",
    "type": "product",
    "brief": "docs/project-brief.md"
  }
}
```

When agents work on a `factory-meta` project, they know they're modifying the factory itself — agent definitions, the registry schema, orchestration logic. When they're on a `product` project, they don't touch any of that.

Think of it like a construction company that renovates its own office. Same blueprinting process, same quality checks, same project management. The only difference is awareness that "we're working on our own shop right now, so be careful not to knock out a load-bearing wall while we're standing in it."

## The Key Architectural Decision

The thing that makes the factory/product separation work is physical. Halalifarm never has a `src/` directory. It's pure orchestration — agent definitions, the registry, templates, and commands. The moment any agent starts writing code, they're doing it in `~/projects/dashcode/` or wherever the active project lives.

![Halalifarm directory structure](/images/halalifarm/halalifarm_directory_structure.svg)

Every product project follows the same `docs/` structure as halalifarm's own docs. The templates directory holds the standard structure that gets stamped into every new project. And the agent definitions live at the top level, separate from project-specific work.

This gives you a clean bootstrapping story:

1. **Now**: Work on halalifarm as the active project. Build the PM agent, the registry, the workspace conventions.
2. **First real test**: Switch to an existing project, have the PM onboard it.
3. **Feedback loop**: When something doesn't work, switch back to halalifarm, file a task against the factory, fix it with the same process.

That third step is where it gets elegant — the factory improves itself using its own workflow, which validates that the workflow actually works.

## A Note on Building With Claude

I want to be transparent: this entire architecture was designed in conversation with Claude. Not "Claude wrote it while I watched" — more like a design session with a colleague who has strong opinions and infinite patience.

I'd describe what I wanted. Claude would sketch an approach. I'd push back ("but halalifarm *is* the factory, not the product"). Claude would reframe. I'd add constraints ("agents need to use different LLM providers"). Claude would adapt.

The diagrams in this series came out of that conversation too — Claude generated them inline as we talked through the architecture, which made it much easier to spot gaps and misunderstandings in real time.

It's a genuinely useful way to design software. Not because the AI has all the answers, but because explaining your half-formed idea to something that can sketch it back to you forces clarity faster than thinking alone.

---

**In Part 2**, we'll dig into the architectural decisions that had to be right before writing code: the task state machine, multi-provider LLM support with cost tracking, and why your agents should communicate through an append-only message log instead of a shared document.

*[Continue to Part 2: Decisions That Survive Contact With Code →](/blog/decisions-survive-contact-with-code/)*
