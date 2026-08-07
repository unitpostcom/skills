# Unitpost Agent Skills

Reusable [Agent Skills](https://github.com/vercel-labs/skills) that teach AI
coding agents how to use Unitpost correctly and safely.

## Install

With the [`skills`](https://skills.sh) CLI (works with Claude Code, Cursor,
Codex, Copilot, Gemini, Windsurf, and 60+ more):

```bash
# Install the Unitpost skill into the current project
npx skills add unitpostcom/skills --skill unitpost

# …or globally, for every project
npx skills add unitpostcom/skills --skill unitpost -g

# List what's available first
npx skills add unitpostcom/skills --list
```

The CLI auto-detects which agents you have installed and writes the skill into
each agent's skills directory.

## Skills

| Skill      | What it does                                                                                          |
| ---------- | ----------------------------------------------------------------------------------------------------- |
| `unitpost` | Send transactional/marketing email and manage contacts, segments, topics, templates, domains, inbound email, webhooks, and suppressions — via the Unitpost MCP server or REST API. Includes the ID scheme, scopes, pagination, error handling, and send-safety rules. |

## Pairs with the MCP server

The skill teaches the agent *how* to think about Unitpost; the **MCP server**
(`https://mcp.unitpost.com/mcp`) gives it the tools to act. Install both for
the best experience — see the [AI guide](https://www.unitpost.com/guides#ai).

## Publishing (maintainers)

This directory is the **source of truth**. `npx skills add` git-clones the repo
it points at, so the skill is distributed from the **public**
[`unitpostcom/skills`](https://github.com/unitpostcom/skills) repo (this product
monorepo is private). `unitpostcom/skills` is a **mirror** of this folder — never
edit it directly; edit `SKILL.md` here and let the mirror sync.

The `SKILL.md` is safe to publish: it's documentation only (endpoints, shapes,
usage), and the API key is supplied by the user at runtime — nothing secret ever
lives in it.

Syncing is automated: `.github/workflows/skills-mirror.yml` pushes this folder to
`unitpostcom/skills` whenever `skills/**` changes on `main`.

