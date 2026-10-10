# Docker for AI Agents

After getting comfortable the core docker fundamentals, it really made sense me getting excited about learning Docker's AI-agent-specific tooling, so I put together this short course and worked through it myself — four lessons, each with real commands I ran and verified on my own machine (nothing fabricated).

> **Prerequisite:** this course assumes you already know Docker fundamentals (containers, images, networking). If you don't, start with [learn-docker](https://github.com/romancobblepot/learn-docker) first.

## What's inside

| Lesson | Topic |
|---|---|
| 1 | **Sandboxing Fundamentals** — why untrusted agent code needs isolation, and Docker Sandboxes (`sbx`) |
| 2 | **Running LLMs Locally** — Docker Model Runner, pulling and calling an open-weight model |
| 3 | **Giving Agents Tools Safely** — MCP Toolkit & Gateway for containerized tool servers |
| 4 | **Capstone** — combining a local model, MCP tools, and a sandbox into one agent with `docker agent` (cagent) |

## Structure

- `lessons/` — the course content, one file per lesson
- `checkpoints/` — short exercises to check you understood each lesson

## Who it's for

Anyone building agentic or LLM-based apps who wants a quick, practical tour of Docker's agent tooling instead of reading through separate docs for each tool.

## Requirements

Docker Desktop with Sandboxes, Model Runner, MCP Toolkit, and cagent enabled.

If you'd want to study these lessons in an interactive easy-to-read manner, then i would suggest you to follow the below section:

## Browsing it locally

Clone the repo and run the included preview server (no dependencies beyond Python 3):

```bash
git clone git@github.com:romancobblepot/learn-docker-ai-agents.git
cd learn-docker-ai-agents
python3 preview.py
```

Then open `http://127.0.0.1:8000`. It lists this repo's folders and files, and renders any `.md` file as formatted markdown in the browser.
