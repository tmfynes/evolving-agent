---
name: taught-knowledge
description: Curated facts and shortcuts taught by the user. Use when answering with house conventions or when they say remember/next time.
---

# Taught knowledge

Append durable facts here (not secrets). Newest last. Each entry: **trigger** → **what to do**.

## Platform

- **This agent** is `evolving-agent` in org `HubOrg1`, space `HubSpace1`, om1 foundation (`https://api.homelab1.fynesy.com`). Route: `https://evolving-agent.homelab1.fynesy.com`.
- **Source of truth** is `https://github.com/tmfynes/evolving-agent`. Skill updates go there (PR by default, main if asked). Applying them to the running app needs `cf push` from `main`.
- **LLM / memory / MCP / SSO** match `hubagent` in this space: `qwen38`, `paddy-brain` (agent-memory plan `memory`), `hub-mcp`, `wavefront-mcp`, `hub-sso` (`p-identity` plan `hub`).
- GitHub credentials are the bound user-provided service `evolving-agent-github` (`cf cups`). Do not print them.

## Shortcuts

- “save this skill” / “remember this” → `skill-manager` (PR unless they say push/merge to main)
- “skip the PR” / “just push” → `skill-manager` direct to `main`
