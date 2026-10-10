# Lesson 4: Capstone — A Containerized Agent Stack

**Time:** 45-60 minutes
**Tool:** `docker agent` (cagent)

## Learning Objectives

By the end of this lesson, you will:
- Understand why the three previous pillars need to be combined, not used in isolation
- Understand what `docker agent` (cagent) is and how its declarative config works
- Build and run a minimal real agent combining a local model and a tool
- Understand how to run that agent's execution inside a sandbox for defense-in-depth

---

## Concept 1: Why Combine All Three Pillars

### Why?

Each of the first three lessons solved one piece in isolation: a safe place to *execute* agent actions (`sbx`), a local *brain* to reason with (`docker model`), and safe *hands* to act with (`docker mcp`). A real agent needs all three working together — a brain with no hands can't do anything useful; hands with no brain have nothing deciding what to do; and a brain-plus-hands combination running directly on your host, unsandboxed, is right back to the original risk from Lesson 1.

### What?

Think of it as: **brain** (the model, deciding what to do) + **hands** (MCP tools, taking actions) + **safe room** (the sandbox, containing whatever happens). None of the three substitutes for the others.

### How?

The practical question becomes: how do you *declare* this combination once, rather than wiring model endpoints and tool connections together by hand in custom code every time? That's exactly what `docker agent` (cagent) is for.

---

## Concept 2: What `docker agent` (cagent) Is

### Why?

Writing custom orchestration code to connect a model to its tools, manage the conversation loop, and handle multi-agent delegation is real, repetitive engineering work — the same shape of problem every agent project faces, independent of what the agent is actually for.

### What?

`docker agent` (the tool formerly known as **cagent**) is Docker's command-line utility for running AI agents defined in a single declarative YAML file (conventionally `agent.yaml` / `cagent.yaml`). The file declares which model to use, the agent's instructions/behavior, and which tools (including MCP toolsets) it has access to. Multi-agent setups are supported too: a root agent can coordinate hidden sub-agent "specialists," each only invoked by the root, not directly by the user.

Confirmed from `docker agent --help` on this machine, the real command surface includes:

| Command | Purpose |
|---|---|
| `docker agent new` | Create a new agent configuration. |
| `docker agent run <file>` | Run an agent from its YAML config. |
| `docker agent setup` | Interactively set up a model (built-in provider, local, custom endpoint, or Claude Code). |
| `docker agent doctor` | Diagnose model and credential setup. |
| `docker agent models` | List available models. |
| `docker agent toolsets` | List built-in toolset types. |
| `docker agent share` | Share an agent (via an OCI registry). |

### How?

A minimal `agent.yaml` conceptually looks like:

```yaml
name: my-first-agent
model: ai/smollm2          # a Docker Model Runner model, referenced by name
instructions: |
  You are a helpful assistant with access to filesystem tools.
toolsets:
  - type: mcp
    ref: <an MCP server/tool reference from Lesson 3>
```

Then you run it with:

```bash
docker agent run agent.yaml
```

(We confirm the exact, current YAML schema live when we build this together — cagent/`docker agent` is actively evolving, so we verify against `docker agent new` and `docker agent --help` at that point rather than assume the schema here.)

---

## Concept 3: Running the Agent Inside a Sandbox

### Why?

Even a well-architected agent (good model, well-scoped tools through a Gateway) is still, fundamentally, autonomous code execution. Lesson 1's reasoning still applies: the safest default is to not let it touch your host directly at all, regardless of how trustworthy its individual pieces seem.

### What?

This is **defense in depth**: the Gateway restricts *which* tools the agent can call and injects credentials safely; the sandbox ensures that even if something still goes wrong — a bug, an unexpected tool side effect, a prompt injection that slips through — the blast radius is contained to a disposable microVM, not your actual machine.

### How?

```bash
sbx run docker-agent -- run agent.yaml
```

This nests everything you've learned into one command: `sbx` creates the isolated execution environment, and inside it, `docker agent run agent.yaml` starts your declaratively-defined agent — brain, hands, and safe room, together.

---

## Hands-On Capstone Walkthrough

```bash
# 1. Confirm all three pillars are ready
sbx diagnose
docker model list
docker mcp tools ls

# 2. Scaffold a new agent config interactively
docker agent new

# 3. Edit the generated YAML to reference your local model and MCP toolset
#    (model: ai/smollm2, toolsets: [the MCP server from Lesson 3])

# 4. Run it directly first, to confirm it works
docker agent run agent.yaml

# 5. Then run it properly isolated inside a sandbox
sbx run docker-agent -- run agent.yaml

# 6. Clean up
sbx ls
sbx prune
```

---

## Summary

| Concept | One-line takeaway |
|---|---|
| Why combine | Brain (model) + hands (tools) + safe room (sandbox) — each solves a different piece; none substitutes for the others. |
| `docker agent` (cagent) | Declaratively defines an agent's model, instructions, and tools in one YAML file, instead of hand-wired orchestration code. |
| Sandbox + agent together | `sbx run docker-agent -- run agent.yaml` nests execution isolation around the whole declaratively-defined agent — defense in depth. |

## Command Reference

```bash
docker agent new                  # scaffold a new agent.yaml
docker agent run <file>            # run an agent from its config
docker agent setup                  # interactively configure a model/provider
docker agent doctor                   # diagnose model/credential setup
docker agent models                    # list available models
docker agent toolsets                   # list built-in toolset types
sbx run docker-agent -- run <file>        # run the agent fully sandboxed
```

## Course Complete

You've now covered all four pillars of Docker's AI-agent tooling, hands-on, on your own machine: sandboxed execution (`sbx`), local model serving (`docker model`), safe tool access (`docker mcp`), and declarative agent orchestration (`docker agent`) — combined into one working, isolated agent stack.
