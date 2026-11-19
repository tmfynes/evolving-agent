---
name: skill-manager
description: Create or update AGENTS.md and skills in the GitHub repo via PR or push to main. Use when the user wants to remember, save, evolve, or persist a skill or agent definition.
---

# Skill manager

Preserve new understanding in Git. The repo is the durable copy; the running droplet is not.

## Credentials (never print)

Resolve in this order:

1. `$GITHUB_TOKEN` or `$TANZU_AGENT_GITHUB_TOKEN` already in the environment
2. User-provided service `evolving-agent-github` in `$VCAP_SERVICES` (`token`, `github_token`, or `password`)

```bash
eval "$(python3 - <<'PY'
import json, os, shlex
token = os.environ.get("GITHUB_TOKEN") or os.environ.get("TANZU_AGENT_GITHUB_TOKEN") or ""
repo = os.environ.get("SKILLS_REPO_URL") or "https://github.com/tmfynes/evolving-agent.git"
branch = os.environ.get("SKILLS_REPO_BRANCH") or "main"
vcap = os.environ.get("VCAP_SERVICES") or "{}"
try:
    data = json.loads(vcap)
except Exception:
    data = {}
for inst in data.get("user-provided", []):
    if inst.get("name") != "evolving-agent-github":
        continue
    creds = inst.get("credentials") or {}
    token = token or creds.get("token") or creds.get("github_token") or creds.get("password") or ""
    repo = creds.get("repo_url") or repo
print(f"export GITHUB_TOKEN={shlex.quote(token)}")
print(f"export SKILLS_REPO_URL={shlex.quote(repo)}")
print(f"export SKILLS_REPO_BRANCH={shlex.quote(branch)}")
print(f"export GIT_AUTHOR_NAME={shlex.quote(os.environ.get('GIT_AUTHOR_NAME') or 'Evolving Agent')}")
print(f"export GIT_AUTHOR_EMAIL={shlex.quote(os.environ.get('GIT_AUTHOR_EMAIL') or 'tmfynes@yahoo.com')}")
print("export GIT_COMMITTER_NAME=\"$GIT_AUTHOR_NAME\"")
print("export GIT_COMMITTER_EMAIL=\"$GIT_AUTHOR_EMAIL\"")
print("export GH_TOKEN=\"$GITHUB_TOKEN\"")
PY
)"
```

If `GITHUB_TOKEN` is empty, stop and tell the user the `evolving-agent-github` cups binding is missing. Do not invent credentials.

Clone with a token URL but **never echo that URL**:

```bash
WORKDIR="${TMPDIR:-/tmp}/evolving-agent-src"
rm -rf "$WORKDIR"
git clone --depth 20 -b "$SKILLS_REPO_BRANCH" \
  "https://x-access-token:${GITHUB_TOKEN}@github.com/tmfynes/evolving-agent.git" \
  "$WORKDIR"
```

If `SKILLS_REPO_URL` is not `tmfynes/evolving-agent`, parse owner/repo from it and clone that instead.

## Modes

Ask once if unclear. Then:

| User intent | Action |
|---|---|
| default / “open a PR” / “for review” | branch → commit → `gh pr create` → **stop** |
| “merge it” / “merge the PR” | `gh pr merge --squash` on the PR you created (or the one they named) |
| “push to main” / “skip the PR” / “direct to main” | commit on `main` → `git push origin main` |

Never force-push `main`. Never push secrets.

## Skill file rules

- Path: `skills/<name>/SKILL.md` (lowercase, hyphens).
- YAML front matter **must** include `name` and `description`. `description` is the trigger (“Use when…”).
- Prefer editing `skills/taught-knowledge/SKILL.md` for facts/shortcuts.
- New skill only for a distinct procedure.
- Optionally patch `AGENTS.md` when persona or standing rules change.
- Also write the same file into `/home/vcap/app/skills/...` when that path is writable so this live process can load it. GitHub is still required; instance files die on restage/`cf push` from an old droplet.

## PR workflow

```bash
cd "$WORKDIR"
git checkout -b "skill-update-$(date -u +%Y%m%d-%H%M%S)"
# write files, then:
git add AGENTS.md skills
git status
git commit -m "skill: <short description>"
git push -u origin HEAD
gh pr create --base "$SKILLS_REPO_BRANCH" --title "skill: <short description>" --body "$(cat <<EOF
## Why
Taught in conversation; persist so a later short request works.

## Files
- skills/<name>/SKILL.md

After merge, **cf push** this app from main (restart/restage alone is not enough).
EOF
)"
```

Reply with the PR URL. Wait for the user unless they already asked you to merge.

## Direct main

Same edits on `main`, then `git push origin main`. Reply with the commit URL. Tell them to `cf push` from main to bake it into the droplet.

## Merge an existing PR

```bash
gh pr merge <n> --squash --delete-branch
```

Then the same `cf push` reminder.

## After GitHub succeeds

1. Do not dump `git remote -v` (it may contain the token).
2. `git remote set-url origin https://github.com/tmfynes/evolving-agent.git` before any status you might show.
3. Stop. Do not open a second PR for the same change.
