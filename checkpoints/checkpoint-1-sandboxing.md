# Checkpoint 1: Sandboxing Fundamentals

## Purpose

This checkpoint verifies you understand *why* AI agents need stronger isolation than regular containers, how Docker Sandboxes implements that isolation, and the core `sbx` workflow.

## Verification Questions

Try to answer each before revealing it.

<details>
<summary>Question 1: Why does an autonomous coding agent running in "YOLO mode" pose a different risk than ordinary pre-written code?</summary>

**Answer:** Ordinary code is written and typically reviewed by a human before it runs against real systems. An agent in YOLO mode decides *and* executes an action in the same step, often in a loop, with no human approving each individual action. The code being executed is LLM-generated in real time and was never reviewed before running.

**Why this matters:** This is the entire motivation for sandboxing agents specifically — the risk isn't "containers are insecure," it's "this specific class of code was never reviewed before executing."
</details>

<details>
<summary>Question 2: Why isn't a regular Docker container sufficient isolation for untrusted, agent-generated code?</summary>

**Answer:** A regular container still shares the host machine's kernel. If the agent's code triggers a kernel-level bug or container-escape vulnerability, it can potentially break out of that shared kernel and reach the host directly.

**Why this matters:** This is precisely the gap that microVM-based sandboxing (like `sbx`) closes — it's not that containers are "bad," it's that they share a kernel, and untrusted code raises the stakes of that sharing.
</details>

<details>
<summary>Question 3: What does a microVM give a sandbox that a regular container does not?</summary>

**Answer:** Its own, separate kernel — instead of sharing the host's kernel. Even a kernel-level exploit inside the sandbox only compromises that sandbox's private kernel, not the host machine.

**Why this matters:** This is the core technical mechanism, not just a marketing term — understanding "separate kernel" vs. "shared kernel" is what distinguishes a microVM from a container at a technical level.
</details>

<details>
<summary>Question 4: What's the actual difference between `sbx stop` and `sbx rm`?</summary>

**Answer:** `sbx stop` pauses a sandbox without deleting it (it still exists and can be resumed). `sbx rm` permanently deletes a sandbox. This directly parallels `docker stop` vs. `docker rm` from core Docker.

**Why this matters:** Confusing these could mean you think you've cleaned something up when you've only paused it, or vice versa.
</details>

<details>
<summary>Question 5: What problem do Sandbox Kits solve, and what problem do network policies (Open/Balanced/Locked Down) solve? Are they the same thing?</summary>

**Answer:** No — they're different, complementary controls. Network policies control *what the sandbox can reach over the network*, independent of anything else. Sandbox Kits are a broader declarative config (YAML) covering tools, environment variables, injected credentials, allowed domains, files, and startup commands — of which network access is just one part. A Kit can set a network allowlist, but Kits cover much more than networking alone.

**Why this matters:** Treating these as interchangeable would lead to misconfiguring one while assuming the other already covers it.
</details>

## Hands-On Self-Check

Run these and confirm the behavior matches what you expect:

```bash
sbx diagnose          # all checks should pass
sbx run claude         # launches an agent in a fresh sandbox
sbx ls                 # confirm the sandbox appears, status running
sbx stop <id>          # stop it
sbx ls                 # it should still appear, now stopped
sbx rm <id>            # remove it
sbx ls                 # it should no longer appear at all
```

- [x] `sbx diagnose` shows all checks passed
- [x] `sbx run claude` successfully creates and starts a sandbox
- [x] `sbx ls` shows the sandbox's actual status (running vs. stopped) correctly
- [x] After `sbx rm`, the sandbox no longer appears in `sbx ls` at all

## Next Step

Once comfortable with all five answers, move to **Lesson 2: Running LLMs Locally — Docker Model Runner**.
