# evolving-agent

Tanzu **agent_buildpack** demo: chat, then persist a taught skill to GitHub (pull request by default, or push/merge to `main` when asked).

- Org/space: `HubOrg1` / `HubSpace1` on om1 (`api.homelab1.fynesy.com`)
- URL: https://evolving-agent.homelab1.fynesy.com
- Repo (source of truth for `AGENTS.md` + `skills/`): this repository

Runtime skill files in the droplet are **not** durable. After merge to `main`, **`cf push`** this directory (restart/restage alone reloads the old droplet).

## Layout

```
AGENTS.md
manifest.yaml
skills/skill-manager/SKILL.md
skills/taught-knowledge/SKILL.md
```

## Deploy

Target the space, register GitHub creds as a user-provided service (do not commit the JSON), then push.

```bash
cf target -o HubOrg1 -s HubSpace1

# github-creds.json (local only):
# { "token": "<github pat>", "repo_url": "https://github.com/tmfynes/evolving-agent.git" }
cf cups evolving-agent-github -p github-creds.json

cf push -f manifest.yaml -p .
```

The manifest binds the same LLM, memory, and MCP instances as `hubagent` (`qwen38`, `paddy-brain`, `hub-mcp`, `wavefront-mcp`) plus `evolving-agent-github`. Bind SSO after push so the callback matches this app:

```bash
cf bind-service evolving-agent hub-sso --wait -c '{
  "grant_types": ["authorization_code"],
  "redirect_uris": ["https://evolving-agent.homelab1.fynesy.com/api/auth/callback"],
  "scopes": ["openid"]
}'
cf restage evolving-agent
```

Optional: `cf set-env evolving-agent GITHUB_TOKEN '<pat>'` so `gh` sees the token without parsing `VCAP_SERVICES`. Prefer the cups binding as the registered copy.

## Demo loop

1. Give a verbose procedure in chat.
2. Ask: “save that as a skill so next time I can just say …”
3. Agent opens a PR (or pushes `main` if you said so).
4. Merge if needed, then `cf push` from `main`.
5. Repeat with the short trigger.
