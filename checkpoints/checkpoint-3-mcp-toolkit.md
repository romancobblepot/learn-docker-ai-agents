# Checkpoint 3: Giving Agents Tools Safely — MCP Toolkit & Gateway

## Purpose

Verifies you understand why agents need tools, what MCP standardizes, why Docker containerizes MCP servers, and the Gateway's role.

## Verification Questions

<details>
<summary>Question 1: What problem does MCP solve that existed before it?</summary>

**Answer:** Before MCP, every agent framework defined and called tools in its own bespoke way, so a tool built for one framework couldn't be reused in another without rewriting it — an N×M integration problem across frameworks and tools. MCP standardizes the protocol so one tool implementation (an MCP server) works with any MCP-compatible agent framework (an MCP client).

**Why this matters:** Understanding MCP as a standardization/interoperability solution (not a new capability) clarifies why it matters even though "tool calling" already existed before it.
</details>

<details>
<summary>Question 2: What's the difference between an MCP server and an MCP client?</summary>

**Answer:** An MCP server implements the protocol to expose a specific set of tools (e.g. filesystem access, web fetch). An MCP client (built into an agent framework) speaks the same protocol to discover and call those tools.

**Why this matters:** These are two distinct roles in the same protocol — confusing them would make it unclear which side you're building or running at any given time.
</details>

<details>
<summary>Question 3: What specific risks does running an MCP server directly on your host (e.g. via `npx some-mcp-server`) carry, that containerizing it addresses?</summary>

**Answer:** The tool's code runs with your full user permissions, its dependencies install directly on your machine (version conflicts, supply-chain risk), and there's no consistent way to manage credentials it needs beyond however that specific package handles environment variables.

**Why this matters:** This is the direct motivation for the MCP Toolkit — the same container-isolation benefits you learned in core Docker, applied specifically to agent tool servers.
</details>

<details>
<summary>Question 4: What specific problem does the MCP Gateway solve that containerizing the servers alone does not?</summary>

**Answer:** Even with servers containerized, an agent connecting directly to many different tool servers still means scattered credential handling and no central place to enforce which agent can call which tools, or audit what was called. The Gateway centralizes authentication, credential injection, access control, and routing behind one endpoint.

**Why this matters:** Containerization and the Gateway solve different, complementary problems — isolation of each server vs. centralized control over all of them together.
</details>

<details>
<summary>Question 5: Where does `docker mcp secret` store credentials, and why does that matter compared to a plaintext environment variable?</summary>

**Answer:** It stores secrets in the local OS Keychain, rather than in a plaintext `.env` file or shell environment variable that could be accidentally logged, committed to a repo, or read by any process that can access the environment.

**Why this matters:** This is a concrete, meaningful security improvement over the "ad-hoc env var" pattern common in un-containerized MCP server setups.
</details>

## Hands-On Self-Check

```bash
docker mcp catalog ls        # confirm servers are browsable
docker mcp server enable <server-name>
docker mcp gateway run        # confirm the gateway starts
docker mcp tools ls             # confirm tools are listed through the gateway
```

- [ ] You can browse the MCP catalog and identify at least one server
- [ ] You can enable a server without errors
- [ ] The gateway starts and lists the tools that server exposes

## Next Step

Move to **Lesson 4: Capstone — A Containerized Agent Stack**.
