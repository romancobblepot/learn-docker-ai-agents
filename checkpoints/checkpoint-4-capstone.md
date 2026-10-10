# Checkpoint 4: Capstone — A Containerized Agent Stack

## Purpose

Verifies you understand why the three pillars (sandbox, model, tools) must be combined, what `docker agent` (cagent) actually does, and why sandboxing still matters even with a well-architected agent.

## Verification Questions

<details>
<summary>Question 1: Why isn't a well-chosen model alone, or a well-scoped set of tools alone, enough to make a useful, safe agent?</summary>

**Answer:** A brain (model) with no hands (tools) can't take any actions. Hands with no brain have nothing deciding what to do. And a brain-plus-hands combination running unsandboxed on your host is back to the original risk from Lesson 1 — none of the three pillars substitutes for the others.

**Why this matters:** This is the structural argument for why the course builds toward a capstone that combines all three, rather than treating any single lesson as sufficient on its own.
</details>

<details>
<summary>Question 2: What does `docker agent` (cagent) actually replace in an agent project?</summary>

**Answer:** Custom orchestration code that would otherwise connect a model to its tools, manage the conversation loop, and handle multi-agent delegation by hand. It replaces that with a single declarative YAML file describing the model, instructions, and tools.

**Why this matters:** Recognizing this as a declarative-config tool (like a Dockerfile or compose.yaml, applied to agent definitions) rather than a new agent framework to learn from scratch makes its purpose click faster.
</details>

<details>
<summary>Question 3: In a multi-agent `docker agent` setup, how do sub-agent "specialists" relate to the root agent and the user?</summary>

**Answer:** Sub-agents are coordinated by the root agent and are hidden from the user — only the root agent is directly invoked by the user; it calls on sub-agent specialists as needed.

**Why this matters:** This is a specific architectural detail that affects how you'd design a multi-agent config — specialists aren't independently user-facing.
</details>

<details>
<summary>Question 4: Why run the agent inside `sbx` even after the Gateway already restricts which tools it can call?</summary>

**Answer:** This is defense in depth — the Gateway restricts which tools can be called and injects credentials safely, but the agent is still fundamentally autonomous code execution. The sandbox ensures that even if something unexpected happens (a bug, an unexpected tool side effect, a prompt injection that slips through), the blast radius is contained to a disposable microVM, not the host machine.

**Why this matters:** This directly ties back to Lesson 1's core argument — good tool-scoping reduces risk, it doesn't eliminate the value of isolation as a separate, independent layer.
</details>

<details>
<summary>Question 5: What does the command `sbx run docker-agent -- run agent.yaml` actually do, piece by piece?</summary>

**Answer:** `sbx run docker-agent` creates an isolated sandbox and launches the `docker-agent` program inside it; everything after `--` (`run agent.yaml`) is passed through as the arguments to `docker-agent` itself, telling it to run the agent defined in `agent.yaml`. So it's "create a sandbox, and inside it, run this specific agent config" — nesting the sandbox lesson and the agent lesson into one command.

**Why this matters:** Being able to parse a nested command like this into its component pieces is exactly the skill needed to combine any two tools from this course together.
</details>

## Hands-On Self-Check

```bash
docker agent new                      # scaffold a config
docker agent run agent.yaml             # run it directly
sbx run docker-agent -- run agent.yaml   # run it sandboxed
sbx ls                                     # confirm the sandboxed run appears
sbx prune                                   # clean up
```

- [x] `docker agent run agent.yaml` runs without error directly on the host
- [x] The same agent also runs successfully via `sbx run docker-agent -- run agent.yaml`
- [x] You can explain, in your own words, what changes between those two runs (isolation, not behavior)

## Course Complete

All four checkpoints done — you've verified understanding of sandboxed execution, local model serving, safe tool access via MCP, and declarative agent orchestration, both separately and combined.
