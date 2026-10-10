# Lesson 2: Running LLMs Locally — Docker Model Runner

**Time:** 45-60 minutes
**Tool:** `docker model` (Docker Model Runner)

## Learning Objectives

By the end of this lesson, you will:
- Understand why running an LLM locally (vs. only calling a cloud API) matters for agent applications
- Understand what Docker Model Runner actually is and how it distributes models
- Pull and run a real open-weight model locally
- Call that model's OpenAI-compatible API from your own code

---

## Concept 1: Why Run an LLM Locally At All?

### Why?

Every agent needs a "brain" — the LLM that reasons about what to do next. The default path is calling a cloud API (OpenAI, Anthropic, etc.), which is simple and gives you the strongest models, but comes with real costs for agent-development work specifically: per-token billing on every iteration while you're debugging an agent loop, network latency on every single call, data leaving your machine for every prompt (a real concern if you're experimenting with sensitive code or data), and zero availability without internet access.

### What?

Running a model **locally** means the model's weights live on your own disk and inference happens on your own CPU/GPU — no network call, no per-token bill, no data leaving the machine. The trade-off is capability: local open-weight models are generally smaller and less capable than the largest hosted models, and your hardware limits which model sizes are practical to run.

### How?

In practice, this is most valuable during **development and testing** of agent logic — iterating on prompts, tool-calling behavior, and control flow — where you're making many rapid, low-stakes calls and don't need the absolute best model to validate that your agent's *logic* works. You'd typically still use a stronger hosted model for production quality, but prototype and test against a local one.

---

## Concept 2: What Docker Model Runner Actually Is

### Why?

If local inference is useful, you still need a consistent way to get a model onto your machine, manage versions of it, and run it — ideally using tooling you already know, instead of learning an entirely separate model-serving ecosystem.

### What?

**Docker Model Runner** is Docker's built-in way to run open-weight LLMs locally, integrated directly into Docker Desktop/Engine. Its distinguishing idea: models are distributed and versioned as **OCI artifacts** — the same packaging format container images use — through Docker Hub, under the `ai/` namespace (e.g. `ai/smollm2`, `ai/llama3.2`). This means pulling a model uses the exact same mental model as pulling an image: a name, a tag, a registry, cached locally after the first pull.

Under the hood, Model Runner uses `llama.cpp` as its default inference engine (with `vLLM` available for higher-throughput serving on supported NVIDIA GPUs, and a `Diffusers` engine for image models) — but as a user, you mostly interact with it through the `docker model` CLI, not the underlying engine directly.

### How?

The core commands, confirmed directly from `docker model --help` on this machine:

| Command | Purpose |
|---|---|
| `docker model pull <model>` | Downloads a model (as an OCI artifact) from Docker Hub or Hugging Face to your local environment. |
| `docker model list` | Lists models you've already pulled locally. |
| `docker model run <model>` | Runs a model (starting it if needed). |
| `docker model ps` | Lists currently running models. |
| `docker model df` | Shows disk usage consumed by local models. |
| `docker model inspect <model>` | Shows detailed information about one model. |
| `docker model purge` | Removes all local models. |
| `docker model package` | Packages your own model into a Docker Model OCI artifact, for distribution. |

---

## Concept 3: The OpenAI-Compatible API

### Why?

If every local model-runner had its own bespoke API, every agent framework would need a custom integration per backend. Reusing an API shape the entire ecosystem already speaks removes that friction entirely.

### What?

Docker Model Runner exposes an **OpenAI-compatible REST API** — the same request/response shape as OpenAI's own `/chat/completions` endpoint. This means any library, SDK, or agent framework already written to talk to "an OpenAI-compatible endpoint" (which is most of them, at this point) can talk to your local model by simply pointing its base URL at Model Runner instead of `api.openai.com` — no code changes, just a different URL and no API key requirement for the local model itself.

### How?

Once a model is running, you call it like any OpenAI-compatible endpoint:

```bash
curl http://localhost:12434/engines/llama.cpp/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "ai/smollm2",
    "messages": [{"role": "user", "content": "Say hello in one sentence."}]
  }'
```

From Python, this means pointing an OpenAI SDK client's `base_url` at that local address instead of OpenAI's — the rest of your agent code doesn't need to know the difference.

---

## Hands-On Walkthrough

```bash
# 1. Enable Model Runner (one-time)
docker desktop enable model-runner

# 2. Confirm it's enabled and empty so far
docker model list

# 3. Pull a small model
docker model pull ai/smollm2

# 4. Confirm it's now listed locally
docker model list

# 5. Run it
docker model run ai/smollm2

# 6. In another terminal, confirm it's running
docker model ps

# 7. Call its OpenAI-compatible API
curl http://localhost:12434/engines/llama.cpp/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "ai/smollm2", "messages": [{"role": "user", "content": "Hello!"}]}'

# 8. Check disk usage
docker model df
```

---

## Summary

| Concept | One-line takeaway |
|---|---|
| Why local | No per-token cost, no latency, no data leaving the machine — ideal for agent development/testing, not necessarily production quality. |
| Docker Model Runner | Runs local LLMs using the same pull/registry mental model as container images, via OCI artifacts under the `ai/` namespace. |
| OpenAI-compatible API | Point any existing OpenAI-client code at a local URL instead of `api.openai.com` — no integration rewrite needed. |

## Command Reference

```bash
docker desktop enable model-runner   # enable the feature
docker model pull <model>             # download a model
docker model list                      # list local models
docker model run <model>                # run a model
docker model ps                          # list running models
docker model inspect <model>              # detailed model info
docker model df                            # disk usage
docker model purge                          # remove all local models
```

**Next:** Checkpoint 2, then Lesson 3 — giving this model's agent safe access to external tools via the MCP Toolkit & Gateway.
