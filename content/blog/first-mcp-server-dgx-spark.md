---
title: "So You've Never Built an MCP Server Before"
date: 2026-11-23
draft: false
tags: ["mcp", "ai", "claude", "python", "docker", "dgx-spark"]
categories: ["engineering"]
description: "The shortest path from 'we have models and solvers on DGX Sparks' to 'Claude can call them': one container, one or two tools, about a week."
showToc: true
cover:
  image: "https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=1200&q=80"
  alt: "Rack of servers with network cables"
---

Your team builds models and Python solvers, and most of that work happens on DGX Sparks. Now people want to use those tools straight from Claude: "re-solve problem 42 with more iterations" typed into a chat, not a script.

This is the shortest path there. By the end of the week you can have one container running on a Spark that wraps one or two tools around code you already have.

The main idea: **an MCP server is a thin, lightweight layer.** Put it in a container and keep it separate from the heavy compute. Then where it runs is a deployment choice, and moving it never means a redesign.

Writing the code won't be the hard part. Writing good tool descriptions will.

---

## What an MCP server is

An MCP server is a small program that exposes your code as **tools** that an AI assistant like Claude can call. MCP (the [Model Context Protocol](https://modelcontextprotocol.io)) is the open standard that defines how the assistant finds those tools and calls them.

The server does four things. It advertises a list of tools (name, inputs, description), receives a call, runs your code, and returns the result. It is **not** the model and doesn't run the model. Most of its time is spent waiting on other systems, so it needs very little CPU and no GPU.

Think of it as **a menu plus a waiter.** The menu tells the customer what can be ordered, and the waiter takes the order to the kitchen. The kitchen, meaning your solver or model, does the real work.

---

## Where it should run

Start by running the MCP server in a container on a Spark. Once other people depend on it, move that same container to always-on infrastructure. A Spark handles the load easily. The trouble is that a Spark is one developer's workstation, not a server.

```
        Claude Desktop or Claude Code
                     │  MCP over streamable HTTP
                     ▼
   ┌────────────────────────────────────────────┐
   │ MCP server (container)                     │
   │ Tool definitions, login token, call logging│
   │ Starts on a Spark; moves to always-on later│
   └────────────────────────────────────────────┘
                     │  HTTP or gRPC
          ┌──────────┴───────────┐
   ┌── DGX Sparks: team-owned GPU compute ────────┐
   │ ┌─────────────────┐  ┌──────────────────────┐│
   │ │ Model service   │  │ Python solver service││
   │ │ (modeling team) │  │ (small FastAPI app)  ││
   │ └─────────────────┘  └──────────────────────┘│
   └──────────────────────────────────────────────┘

Rule: the MCP server never imports solver or model code;
      it only calls them as services.
```

Clients only ever talk to the MCP container. The heavy compute stays on the Sparks behind it.

### Why not leave it on a Spark for good

- **No redundancy.** If the box reboots, gets reimaged, or a big training run takes all its memory, the MCP server goes down for everyone.
- **The box changes constantly.** The team upgrades drivers, CUDA, and Python whenever their work needs it, and the MCP server shouldn't break because of that.
- **ARM chip.** Sparks use NVIDIA's Grace CPU, so images built there are ARM images and may not run as-is on x86 servers.
- **Security and access.** Opening a team dev box to wider use raises questions about logins, firewalls, and logging.

### Why a container

1. **Portable.** The same image runs on a Spark today and on a VM, a Kubernetes cluster, or a cloud service later. Build for ARM and x86 from day one.
2. **Isolated.** It carries its own dependencies, so library updates on the Spark can't break it.
3. **Versioned.** Deploying means picking an image version, and rolling back means picking the previous one.
4. **Easier to secure.** A container's permissions and network access are simpler to lock down than a shared workstation's.
5. **Clean ownership.** The modeling team owns the model and solver services, and whoever owns the MCP server owns that layer. Each ships on its own schedule.

### The phased plan

- **Now:** run the MCP container on a Spark next to the model and solver services.
- **Later:** move only the MCP container to always-on infrastructure. The compute stays on the Sparks, and nothing gets rewritten.

---

## Build it

The recommended stack is the official Python MCP SDK, the streamable HTTP transport, and Docker, which comes preinstalled on DGX OS.

| Piece | Choice | Why |
| --- | --- | --- |
| Language and SDK | Python, `pip install "mcp>=2.3,<3"` | The team already works in Python, and one decorator turns a function into a tool |
| Transport | Streamable HTTP | Reachable over the network; stdio only works on one machine |
| Packaging | Docker container | Preinstalled on DGX OS and portable to any server later |
| Testing | MCP Inspector (`npx @modelcontextprotocol/inspector`) | Lets you call tools by hand before connecting Claude |

> **Heads-up on SDK versions:** in version 2 of the Python SDK, `FastMCP` was renamed `MCPServer`, and `host`/`port` moved from the constructor to `run()`. Many tutorials still show the v1 `from mcp.server.fastmcp import FastMCP` form, which fails with an import error on 2.x. The code below targets 2.x (tested on 2.3.0).

### Step 1: Put the solver behind a small API

The MCP server should call the solver over HTTP, not import its code. A small FastAPI app with an endpoint like `POST /solve` keeps the two apart.

Calling the solver directly from the MCP process is fine for a first-week experiment. Split them before anyone else depends on it.

### Step 2: Write the MCP server

Here's `server.py`. The tool name and parameters are placeholders, so swap in your solver's real ones.

```python
import os, httpx
from mcp.server.mcpserver import MCPServer

SOLVER_URL = os.getenv("SOLVER_URL", "http://solver:8001")
mcp = MCPServer("solver-mcp")

@mcp.tool()
async def run_solver(problem_id: str, max_iterations: int = 1000) -> dict:
    """Run the solver on a saved problem and return the solution summary.
    Use this when the user asks to solve or re-solve a named problem."""
    async with httpx.AsyncClient(timeout=300) as c:
        r = await c.post(f"{SOLVER_URL}/solve",
                         json={"problem_id": problem_id, "max_iterations": max_iterations})
        r.raise_for_status()
        return r.json()

if __name__ == "__main__":
    # serves at http://<host>:8000/mcp
    mcp.run(transport="streamable-http", host="0.0.0.0", port=8000)
```

The docstring matters a lot. It is what the model reads to decide when to call this tool and how. More on that below.

### Step 3: Containerize it

```dockerfile
FROM python:3.12-slim
WORKDIR /app
RUN pip install --no-cache-dir "mcp>=2.3,<3" httpx
COPY server.py .
EXPOSE 8000
CMD ["python", "server.py"]
```

Build and run it on the Spark:

```bash
docker build -t solver-mcp .
docker run -d -p 127.0.0.1:8000:8000 \
  --add-host=host.docker.internal:host-gateway \
  -e SOLVER_URL=http://host.docker.internal:8001 solver-mcp
```

Two things to watch:

- **Solver address.** On Linux, `host.docker.internal` only resolves if you pass the `--add-host` flag shown above. For the long term, Docker Compose running both services together is cleaner.
- **Chip type.** Sparks are ARM, so a plain `docker build` there produces an ARM image. The `python:3.12-slim` base supports both ARM and x86, so `docker buildx build --platform linux/arm64,linux/amd64 -t solver-mcp .` produces both.

### Step 4: Test before connecting Claude

Run the MCP Inspector:

```bash
npx @modelcontextprotocol/inspector
```

Point it at `http://localhost:8000/mcp`, choose the streamable HTTP transport, and call `run_solver` by hand. Once that works, connect Claude. In Claude Code, that's one command:

```bash
claude mcp add --transport http solver http://localhost:8000/mcp
```

**A realistic first milestone:** one tool that calls the real solver, run from Claude, within about a week.

---

## Mistakes to avoid

Most early problems come from tool design, not code.

1. **Thin tool descriptions.** The model reads each tool's docstring to decide when and how to use it. Write it the way you'd brief a smart colleague: what the tool does, when to use it, and what each input means.
2. **Too many small tools.** Three tools that each do a meaningful job beat thirty thin ones, so don't mirror every solver function.
3. **Making one call wait on a long job.** If a solve takes more than a minute or two, split it in two: one tool that starts the job and returns an ID, and another that checks its status.
4. **Opening it up too early.** At first, bind to `127.0.0.1` or the internal network only. Add a login token before anyone outside the team uses it.
5. **No logging.** Log every tool call with its inputs, duration, and errors. When something breaks, that log is the first thing you'll need.
6. **Write access on day one.** Start with tools that look things up or run solves. Hold off on anything that deletes or overwrites data until people trust the setup.

---

## What's next

Once other people depend on it, move the MCP container to always-on infrastructure, such as a VM, a Kubernetes cluster, or a managed container service. Point `SOLVER_URL` back at the Sparks, add a login token and call logging, and you're done.

Nothing gets rewritten, because the MCP server was only ever a thin layer between Claude and the services you already had.
