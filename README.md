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
- **Skill `levels`** (`/owlmastery:levels NVDA`). Reads a ticker's call resistance, put support, HVL, gamma wall, net gamma, 1-day range and implied volatility against its close. Claude also loads it on its own when you ask about those levels.
- **Skill `market`** (`/owlmastery:market`). Summarizes breadth, indexes, movers, sectors and your watchlists.

The server has seven read-only tools: `search_tickers`, `get_ticker_levels`, `get_gamma_exposure`, `get_volatility`, `get_market_overview`, `get_sectors` and `list_watchlists`. The server describes their inputs itself, so the skills name the tools and leave the schemas out. Each skill's `allowed-tools` line lets Claude Code call the tools the skill uses without a prompt. A plugin's MCP tool is named `mcp__plugin_<plugin>_<server>__<tool>`, so `get_ticker_levels` of the `owlmastery` server in the `owlmastery` plugin is `mcp__plugin_owlmastery_owlmastery__get_ticker_levels`.

## Test against localhost

Start the Owl Mastery API on `http://localhost:8080` and create a key against it. Then point the plugin at it:

```bash
export OWLMASTERY_MCP_URL=http://localhost:8080/mcp
export OWLMASTERY_API_KEY=owl_...
claude
```

`OWLMASTERY_MCP_URL` is optional. When unset, the plugin uses `https://owlmastery.com/mcp`. Run `/mcp` in Claude Code to see whether `owlmastery` is connected. To validate the manifests, run `claude plugin validate .` and `claude plugin validate plugins/owlmastery`.

## Check the plugin

```bash
claude plugin validate .
claude plugin validate plugins/owlmastery
```

To see the token cost, install the plugin and run `claude plugin details owlmastery`. The two skills add about 180 tokens to every session. A skill's body loads only when it fires, about 900 tokens for `levels` and 500 for `market`.

The suite in `plugins/owlmastery/evals/` checks that each skill fires on the prompts it should and stays out of the ones it should not. It needs no server, because the cases stop at the skill call. It runs a model, so it costs a few cents:

```bash
cd plugins/owlmastery
claude plugin eval . --ablation none --runs 1 --model haiku --no-publish
```

Drop `--runs 1` for three runs per case. Results go to `evals/results/`, which git ignores.

## What the plugin leaves out

- **No `userConfig` key prompt.** Claude Code can ask for the key and keep it in the keychain. The variable stays because the Codex plugin reads the same one, so one setup covers both clients. A required prompt would also block installing the plugin before the user has a key. Revisit it when the server has OAuth.
- **No commands, agents or hooks.** A skill already runs as `/owlmastery:levels`, so a command adds nothing. The skills only read data, so no agent needs its own context. A hook cannot see a key's age, and Claude Code already reports a failed connection.
- **No MCPB bundle or MCP app widgets.** The server is remote HTTP, so there is nothing local to bundle. The answers are numbers and levels that read well as text.

## What works where

| Client | Works now | Notes |
| --- | --- | --- |
| Claude Code | Yes | Header with an environment variable. |
| Claude.ai and Claude Desktop connectors | Not yet | Connectors use OAuth, and the server has none yet. Claude.ai may accept a fixed request header as a beta option, which is untested. |
| ChatGPT and Codex | See the Codex plugin repository | |

## Keep the skills in sync

The skills in `plugins/owlmastery/skills/` also ship in the Codex plugin repository. After you edit a skill in either repository, copy its `SKILL.md` body and its `references/` folder to the other repository unchanged. The only difference allowed is the `allowed-tools` line in the frontmatter, which only the Claude Code copy has: the Claude Code copy keeps it and the Codex copy leaves it out. The skill text has no client-specific wording, so the body needs no edits. Run `diff -r` on the two `skills/` folders. It must show only the `allowed-tools` lines.

## Layout

```text
.claude-plugin/marketplace.json     the marketplace, named owlmastery
plugins/owlmastery/
  .claude-plugin/plugin.json        the plugin manifest
  .mcp.json                         the MCP server and its header
  skills/levels/                    SKILL.md and references/glossary.md
  skills/market/                    SKILL.md
  evals/                            skill-trigger cases for claude plugin eval
```

The data is educational information, not investment advice.
