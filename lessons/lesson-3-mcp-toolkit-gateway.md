# Lesson 3: Giving Agents Tools Safely — MCP Toolkit & Gateway

**Time:** 45-60 minutes
**Tool:** `docker mcp` (Docker MCP Toolkit / Gateway)

## Learning Objectives

By the end of this lesson, you will:
- Understand why agents need "tools," and the specific risk of giving them tools directly
- Understand what MCP (Model Context Protocol) is and why Docker containerizes MCP servers
- Understand the MCP Gateway's role as a security/credential choke point
- Run a containerized MCP server and connect it through the gateway

---

## Concept 1: Why Agents Need Tools

### Why?

An LLM by itself can only produce text — it can't check today's weather, query a database, read a file, or call an API on its own. To be useful as an "agent" rather than just a chatbot, it needs a way to actually **take actions** in the world, and to receive structured results back. This capability is usually called "tool calling" or "function calling."

### What?

A **tool** is simply a defined capability — a function with a name, a description, and a schema for its inputs/outputs — that the LLM can decide to invoke, usually by generating a structured request your code then executes on its behalf, and feeds the result back into the conversation.

### How?

In practice, before MCP existed, every agent framework defined tools its own bespoke way — a tool written for one framework couldn't be reused in another without rewriting it. This fragmentation is exactly the problem MCP was created to solve (next concept).

---

## Concept 2: What MCP (Model Context Protocol) Is

### Why?

If every agent framework and every tool provider used a different, incompatible way of describing and calling tools, you'd need a custom integration for every (framework × tool) pair — an N×M combinatorial mess that doesn't scale as either number grows.

### What?

**MCP (Model Context Protocol)** is a standardized protocol for exposing tools (and other context, like resources and prompts) to LLM applications, regardless of which framework or model is on the other end. An "MCP server" implements this protocol to expose a specific set of tools (e.g. a filesystem server, a web-fetch server, a database server); an "MCP client" (built into an agent framework) speaks the same protocol to discover and call those tools. Write a tool as an MCP server once, and any MCP-compatible agent can use it — the N×M problem collapses to N+M.

### How?

You don't typically write raw MCP protocol messages by hand — you either run an existing MCP server (for a capability someone already built, like filesystem access or web fetching) or build a small one exposing your own custom logic, and let your agent framework's MCP client handle the protocol details.

---

## Concept 3: Why Containerize MCP Servers

### Why?

The most common way to run an MCP server historically has been `npx some-mcp-server` or `pip install && run` directly on your host machine. That means the tool's code runs with your full user permissions, its dependencies are installed directly on your machine (version conflicts, supply-chain risk from whatever that package pulls in), and there's no consistent way to manage credentials it might need (an API key for a weather service, database credentials, etc.) beyond however that specific package decided to read environment variables.

### What?

Docker's answer is to run MCP servers **as containers** instead — giving each one the same isolation, reproducibility, and resource-boundary benefits any containerized process gets, applied specifically to agent tools. The **MCP Toolkit** is the Docker Desktop-integrated system for discovering, configuring, and running these containerized MCP servers; the **MCP Catalog** is the curated registry of available MCP server images you can pull from.

### How?

The core `docker mcp` subcommands, confirmed from `docker mcp --help` on this machine:

| Command | Purpose |
|---|---|
| `docker mcp catalog` | Manage MCP server OCI catalogs — browse what's available. |
| `docker mcp server` | Manage (enable/disable/configure) individual MCP servers. |
| `docker mcp client` | Manage MCP clients connecting to your servers. |
| `docker mcp secret` | Manage secrets in the local OS Keychain — the credential-handling piece. |
| `docker mcp gateway` | Manage the MCP Server gateway (next concept). |
| `docker mcp tools` | Manage/inspect tools exposed by your servers. |
| `docker mcp oauth` | Manage OAuth app authorizations for servers that need them. |

---

## Concept 4: The MCP Gateway as a Security Choke Point

### Why?

Even with servers containerized, an agent connecting *directly* to many different tool servers still means scattered credential handling, no central place to enforce "this agent can use these three tools and no others," and no single point to audit what tools were actually called.

### What?

The **MCP Gateway** sits between your agent and all its MCP tool servers as a single, centralized point for authentication, credential injection, access control (which agent can call which tools), and routing. Instead of an agent connecting to five different tool servers directly, it connects to one gateway, and the gateway handles dispatching each call to the right containerized server behind it — while enforcing whatever policy you've configured.

### How?

```bash
docker mcp gateway run
```

This starts the gateway, which then becomes the single endpoint your agent's MCP client connects to, instead of connecting to each tool server individually.

---

## Hands-On Walkthrough

```bash
# 1. Browse available MCP servers
docker mcp catalog ls

# 2. Enable a server (example: a simple fetch/filesystem-style tool)
docker mcp server enable <server-name>

# 3. Store any credential it needs in the OS keychain, rather than a plaintext env var
docker mcp secret set <KEY>=<value>

# 4. Start the gateway so an agent can connect through it
docker mcp gateway run

# 5. Inspect what tools are actually exposed through the gateway
docker mcp tools ls
```

(Exact subcommand flags are confirmed live when we run this lesson together — the Toolkit is an actively evolving product, so we verify current syntax at that point rather than assume it here.)

---

## Summary

| Concept | One-line takeaway |
|---|---|
| Why tools | An LLM alone only produces text — tools let an agent actually take actions and get structured results back. |
| MCP | A standardized protocol so one tool implementation works across any MCP-compatible agent framework — solves the N×M integration problem. |
| Why containerize | Running tool servers as containers gives isolation, reproducibility, and centralized credential handling instead of ad-hoc host installs. |
| MCP Gateway | A single security/credential/access-control choke point between an agent and all its tool servers. |

## Command Reference

```bash
docker mcp catalog ls              # browse available MCP servers
docker mcp server enable <name>     # enable a specific server
docker mcp secret set <k>=<v>        # store a credential in the OS keychain
docker mcp gateway run                # start the gateway
docker mcp tools ls                    # list tools exposed through the gateway
docker mcp client ls                    # list connected MCP clients
```

**Next:** Checkpoint 3, then Lesson 4 — the capstone, combining the model, the tools, and a sandbox into one agent stack.
