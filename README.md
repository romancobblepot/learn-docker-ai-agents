# Docker for AI Agents

A short, hands-on course on Docker's tooling for building AI agents — four lessons, each with real commands run and verified on an actual machine (nothing fabricated).

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
