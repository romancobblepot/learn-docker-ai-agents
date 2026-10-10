# Checkpoint 2: Running LLMs Locally — Docker Model Runner

## Purpose

Verifies you understand why local model inference matters for agent development, how Docker Model Runner distributes and runs models, and how to call it from code.

## Verification Questions

<details>
<summary>Question 1: What are the main practical costs of always calling a cloud LLM API during agent development, that running locally avoids?</summary>

**Answer:** Per-token billing on every iteration while debugging, network latency on every call, data leaving your machine on every prompt, and no availability without internet access.

**Why this matters:** This is the actual business case for local inference — it's not "better than cloud models," it's "cheaper and faster for the specific activity of iterating on agent logic."
</details>

<details>
<summary>Question 2: What does it mean that Docker Model Runner distributes models as "OCI artifacts," and why does that matter?</summary>

**Answer:** OCI artifacts are the same packaging/distribution format container images use. This means pulling a model follows the identical mental model as pulling a Docker image — a name, a tag, a registry (Docker Hub, under the `ai/` namespace), cached locally after first pull.

**Why this matters:** You don't need to learn a separate model-distribution ecosystem — the registry/pull/cache concepts from core Docker transfer directly.
</details>

<details>
<summary>Question 3: What is Docker Model Runner's default inference engine, and what is it specifically for image-generation models?</summary>

**Answer:** `llama.cpp` is the default engine for text models. A separate `Diffusers` engine handles image-generation models (with `vLLM` also available for higher-throughput serving on supported NVIDIA GPUs).

**Why this matters:** Knowing there are multiple engines under the hood explains why some models behave/perform differently from others, even though you interact with all of them through the same `docker model` CLI.
</details>

<details>
<summary>Question 4: Why does Model Runner expose an OpenAI-compatible API specifically, rather than a custom API of its own?</summary>

**Answer:** Most existing agent frameworks, SDKs, and libraries are already written to talk to "an OpenAI-compatible endpoint." By matching that shape, any of that existing code can point at a local model just by changing the base URL — no rewrite needed.

**Why this matters:** This is what makes local-vs-cloud genuinely a drop-in swap for development purposes, not a separate integration to build and maintain.
</details>

<details>
<summary>Question 5: What's the difference between `docker model list` and `docker model ps`?</summary>

**Answer:** `docker model list` shows all models you've pulled locally (whether running or not) — the model equivalent of `docker images`. `docker model ps` shows only models currently running — the model equivalent of `docker ps`.

**Why this matters:** This exact "all vs. currently running" distinction is one you already learned for containers/images — recognizing the parallel means you don't have to re-learn the concept from scratch.
</details>

## Hands-On Self-Check

```bash
docker model list              # should show ai/smollm2 (or whichever model you pulled)
docker model ps                # should show it running, if you started it
docker model df                # should show non-zero disk usage
curl http://localhost:12434/engines/llama.cpp/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "ai/smollm2", "messages": [{"role": "user", "content": "test"}]}'
```

- [x] The model appears in `docker model list` after pulling
- [x] `docker model ps` correctly reflects whether it's running
- [x] The `curl` call returns a real JSON response with a model-generated message

## Next Step

Move to **Lesson 3: Giving Agents Tools Safely — MCP Toolkit & Gateway**.
