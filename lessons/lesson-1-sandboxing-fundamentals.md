# Lesson 1: Sandboxing Fundamentals — Why Agents Need Isolation

**Time:** 45-60 minutes
**Tool:** `sbx` (Docker Sandboxes CLI)

## Learning Objectives

By the end of this lesson, you will:
- Understand the specific risk profile of autonomous AI agents, distinct from ordinary code
- Understand why a regular container isn't sufficient isolation for untrusted/agent-generated code
- Understand microVM isolation and how Docker Sandboxes implements it
- Run a real agent inside a real sandbox, inspect it, and clean it up

---

## Concept 1: Why Agents Need Isolation At All

### Why?

Traditional software has a human checkpoint: a developer writes code, another developer (or the same one) reviews it, and only then does it run against real systems. An autonomous AI coding agent collapses that checkpoint — the LLM decides *and* executes an action (run a shell command, call an API, edit a file) in the same breath, often in a loop, with no human approving each individual step. Docker's own materials call unsupervised, auto-approved agent execution **"YOLO mode"** — and it's increasingly the default way people actually use coding agents, because stopping to approve every single step defeats the purpose of automation.

This creates a category of risk that didn't really exist before: code that nobody reviewed, generated in real time, executing with real permissions.

### What?

Concretely, this risk shows up as:
- **Destructive actions** — an agent runs `rm -rf` on the wrong path because it misjudged a file location.
- **Credential/secret exfiltration** — an agent reads an API key from an environment variable and sends it somewhere, whether through its own error or because a prompt injection (malicious instructions hidden in a file or web page it read) told it to.
- **Supply-chain risk** — an agent installs and runs a package it was tricked into trusting.
- **Unbounded network access** — an agent makes outbound calls you never intended, whether for exfiltration or just unexpected cost and side effects.

### How?

In practice, you see this risk every time you let an agent run with auto-approval turned on, in a directory that also contains anything you care about — personal files, credentials in `.env` files, SSH keys, or production access. The fix isn't "review every step" (that defeats automation) — it's **run the agent somewhere that physically cannot touch the things you care about**, no matter what it decides to do. That's the whole motivation for this lesson.

---

## Concept 2: Why Containers Alone Aren't Enough — Containers vs. microVMs

### Why?

You already know from core Docker that a container isolates a process's filesystem and process namespace from the host. So why not just run an agent in a regular Docker container and call it a day? Because a regular container still shares the host's **kernel** — the core operating system program that mediates every system call (file access, networking, process control, memory). If an agent's code triggers a kernel-level bug or a container-escape vulnerability, it can, in principle, break out of that shared kernel and reach the host machine directly.

For a container running an image *you* chose and trust (like `nginx`), that shared-kernel risk is low and well-studied. For a container running **arbitrary code an LLM just generated**, it's a meaningfully different, higher-stakes bet.

### What?

A **microVM** closes this gap by giving each sandboxed environment its **own, separate kernel** — not a shared one. This is a much stronger isolation boundary than a container: even a kernel-level exploit inside the sandbox only compromises that sandbox's own private kernel, not the host's. microVMs are heavier than plain containers (there's a real, if small, kernel boot involved), but dramatically lighter and faster than a traditional full virtual machine.

**Docker Sandboxes (`sbx`)** is Docker's product built specifically around this idea: every sandbox runs as its own microVM. Notably, Docker built their own virtual machine monitor for this (rather than reusing an existing one like Firecracker), specifically so sandboxes work on macOS and Windows, not just Linux — which matters, since most developers aren't running Linux as their primary desktop.

### How?

You can check whether your machine even supports this kind of virtualization with `sbx diagnose`, which we ran earlier on this exact machine:

```bash
sbx diagnose
```

Relevant line from that real output:
```
Platform
  ✓ Virtualization — supported
      kern.hv_support is 1
```

`kern.hv_support is 1` is macOS confirming hardware virtualization support is available — the prerequisite that lets `sbx` actually boot a microVM per sandbox, rather than falling back to something weaker.

---

## Concept 3: The Core `sbx` Workflow

### Why?

Once you accept that agents need their own disposable, isolated environment, you need a simple, repeatable way to create one, use it, and throw it away — without that process itself being a chore that makes people skip it under deadline pressure.

### What?

`sbx`'s core commands map directly onto that lifecycle:

| Command | Purpose |
|---|---|
| `sbx run <agent>` | Creates a sandbox and launches the named agent inside it (e.g. `sbx run claude` for Claude Code). |
| `sbx ls` | Lists sandboxes — running and stopped. |
| `sbx exec` | Runs a command inside an already-running sandbox, without starting a new one — conceptually identical to `docker exec`. |
| `sbx attach` | Attaches to a sandbox (starting it first if it's stopped). |
| `sbx stop` | Stops a sandbox without deleting it. |
| `sbx rm` | Permanently removes one or more sandboxes. |
| `sbx prune` | Removes **all** stopped sandboxes in one go — supporting the "disposable by default" philosophy. |
| `sbx cp` | Copies files between a sandbox and the host — the one deliberate, explicit bridge across the isolation boundary. |
| `sbx ports` | Manages port publishing from a sandbox, for when you need to reach something running inside it. |

### How?

A typical session looks like:

```bash
sbx run claude          # launch Claude Code inside a fresh, isolated sandbox
sbx ls                  # see it listed, confirm it's running
sbx exec <id> ls /       # peek inside it without disrupting the running agent
sbx stop <id>            # stop it when done
sbx rm <id>               # or remove it entirely
sbx prune                 # clean up every stopped sandbox at once
```

Because sandboxes are disposable by default, there's no accumulating cruft the way long-lived VMs tend to collect — the expected pattern is to spin one up per task and throw it away afterward.

---

## Concept 4: Network Policies

### Why?

Even inside an isolated sandbox, an agent with unrestricted internet access could still exfiltrate data or reach somewhere it shouldn't — isolation from the *host filesystem* doesn't automatically mean isolation from the *network*. You need a separate, deliberate control over what a sandbox is allowed to talk to.

### What?

`sbx` ships with three named network policies you choose from (configured at login, and overridable per sandbox):

| Policy | Behavior |
|---|---|
| **Open** | No network restrictions — the sandbox can reach anything the host can. |
| **Balanced** | Permits traffic to common development services (package registries, common APIs) while blocking everything else — a sensible default for most agentic coding work. |
| **Locked Down** | Blocks everything except an explicit allowlist you define yourself — the strictest option, appropriate for the highest-risk or most sensitive tasks. |

### How?

The policy is selected the first time you log in (`sbx login` prompts for a default), and Sandbox Kits (next concept) let you override it per-sandbox by explicitly declaring allowed domains.

---

## Concept 5: Sandbox Kits

### Why?

Different tasks need different setups: one agent run might need access to a specific API key and a specific package registry; another might need none of that at all. Re-typing all these settings by hand every time is error-prone and easy to get wrong in exactly the situations where getting it wrong matters most (credentials, network access).

### What?

A **Sandbox Kit** is a YAML file that declares, once, everything a given sandbox should have: which tools are available, which environment variables and credentials get injected, which network domains are allowed, which files are present, and what startup commands run automatically. It's the same idea as a Dockerfile or a `compose.yaml` — infrastructure-as-declarative-config — applied to sandbox setup instead of image building.

### How?

A kit is applied at sandbox-creation time with a `--kit` flag:

```bash
sbx run claude --kit my-kit.yaml
```

This is still an evolving, early-access feature of `sbx` as of this writing (Docker's own docs mark `sbx kit` and `sbx env` as experimental), so expect command details to shift — but the underlying principle (declare your sandbox's exact capabilities instead of configuring by hand each time) is the durable part worth understanding now.

---

## Hands-On Walkthrough

Run these yourself, in order, in a terminal:

```bash
# 1. Confirm sbx is healthy
sbx diagnose

# 2. List sandboxes (starts the daemon if it isn't running yet)
sbx ls

# 3. Launch an agent inside a fresh sandbox
sbx run claude

# 4. In another terminal, list sandboxes again - you should see it running
sbx ls

# 5. Run a command inside the sandbox without disturbing the agent
sbx exec <sandbox-id> whoami

# 6. When you're done, stop and clean up
sbx stop <sandbox-id>
sbx rm <sandbox-id>
# or, to clean up everything stopped at once:
sbx prune
```

---

## Summary

| Concept | One-line takeaway |
|---|---|
| Agent risk | Agents execute LLM-generated code with no human review step — categorically riskier than reviewed code. |
| Containers vs. microVMs | Containers share the host kernel; microVMs give each sandbox its own — a stronger isolation boundary for untrusted code. |
| `sbx` lifecycle | `run` → `ls`/`exec` → `stop`/`rm`/`prune` — disposable by design. |
| Network policies | Open / Balanced / Locked Down — control what the sandbox can reach, independent of filesystem isolation. |
| Sandbox Kits | Declarative YAML config for a sandbox's tools, credentials, and network allowlist. |

## Command Reference

```bash
sbx diagnose              # check installation/daemon health
sbx login                 # authenticate via Docker OAuth
sbx run <agent>            # create a sandbox and launch an agent in it
sbx run <agent> --kit <file>  # launch using a declarative Sandbox Kit
sbx ls                     # list sandboxes
sbx exec <id> <command>    # run a command inside a running sandbox
sbx attach <id>             # attach to a sandbox (starting it if stopped)
sbx cp <src> <dest>          # copy files between sandbox and host
sbx stop <id>                # stop a sandbox
sbx rm <id>                   # remove a sandbox
sbx prune                      # remove all stopped sandboxes
```

**Next:** Checkpoint 1, then Lesson 2 — running the LLM "brain" of an agent locally with Docker Model Runner.
