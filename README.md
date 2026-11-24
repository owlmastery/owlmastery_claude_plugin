# Owl Mastery for Claude Code

A Claude Code plugin and marketplace that connect Claude Code to the Owl Mastery MCP server. The server gives Claude read-only access to Owl Mastery's daily gamma levels, implied volatility and market data. The plugin adds two skills that teach Claude how to read that data.

**Status.** The server answers at `https://owlmastery.com/mcp` once Owl Mastery routes it. Until then, test against a local API (see [Test against localhost](#test-against-localhost)).

## Install

In Claude Code, add the marketplace and install the plugin:

```text
/plugin marketplace add owlmastery/owlmastery_claude_plugin
/plugin install owlmastery@owlmastery
```

To install from a local checkout, run `/plugin marketplace add .` from the repository root instead.

## Create and set an API key

The server accepts a bearer API key from a registered Owl Mastery user. A key starts with `owl_`. You create it while signed in, and it works for 14 days. After that, calls answer 401 and you create a new key.

Set the key in the `OWLMASTERY_API_KEY` environment variable of the shell that starts Claude Code:

```bash
export OWLMASTERY_API_KEY=owl_...
```

Never write the key into a file in this repository or into a Claude Code config file. Both plugins read the variable by name. After you change the variable, restart Claude Code.

## What the plugin adds

- **MCP server `owlmastery`**, streamable HTTP, with the header `Authorization: Bearer ${OWLMASTERY_API_KEY}`. Claude Code expands the variable when it connects, so the key stays out of the config.
- **Skill `levels`** (`/owlmastery:levels NVDA`). Reads a ticker's call resistance, put support, HVL, gamma wall, net gamma and implied volatility, in a fixed tool order. Claude also loads it on its own when you ask about those levels.
- **Skill `market`** (`/owlmastery:market`). Summarizes breadth, indexes, movers, sectors and your watchlists.

The server has seven read-only tools: `search_tickers`, `get_ticker_levels`, `get_gamma_exposure`, `get_volatility`, `get_market_overview`, `get_sectors` and `list_watchlists`. The server describes their inputs itself, so the skills name the tools and leave the schemas out.

## Test against localhost

Start the Owl Mastery API on `http://localhost:8080` and create a key against it. Then point the plugin at it:

```bash
export OWLMASTERY_MCP_URL=http://localhost:8080/mcp
export OWLMASTERY_API_KEY=owl_...
claude
```

`OWLMASTERY_MCP_URL` is optional. When unset, the plugin uses `https://owlmastery.com/mcp`. Run `/mcp` in Claude Code to see whether `owlmastery` is connected. To validate the manifests, run `claude plugin validate .` and `claude plugin validate plugins/owlmastery`.

## What works where

| Client | Works now | Notes |
| --- | --- | --- |
| Claude Code | Yes | Header with an environment variable. |
| Claude.ai and Claude Desktop connectors | Not yet | Connectors use OAuth, and the server has none yet. Claude.ai may accept a fixed request header as a beta option, which is untested. |
| ChatGPT and Codex | See the Codex plugin repository | |

## Keep the skills in sync

The skill files in `plugins/owlmastery/skills/` are the same text as in the Codex plugin repository. This repository is the source of truth. Edit the skills there first, then copy `skills/` to the other repository unchanged. The skill text has no client-specific wording, so the copy needs no edits.

## Layout

```text
.claude-plugin/marketplace.json     the marketplace, named owlmastery
plugins/owlmastery/
  .claude-plugin/plugin.json        the plugin manifest
  .mcp.json                         the MCP server and its header
  skills/levels/                    SKILL.md and references/glossary.md
  skills/market/                    SKILL.md
```

The data is educational information, not investment advice.
