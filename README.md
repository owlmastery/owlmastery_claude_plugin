# Owl Mastery for Claude Code

A Claude Code plugin and marketplace that connect Claude Code to the Owl Mastery MCP server. The server gives Claude read-only access to Owl Mastery's daily gamma levels, implied volatility and market data. The plugin adds three skills that teach Claude how to read that data.

**Status.** The server answers at `https://owlmastery.com/mcp` and signs you in with your Owl Mastery account through OAuth. To test against a local API, see [Test against localhost](#test-against-localhost).

## Install

In Claude Code, add the marketplace and install the plugin:

```text
/plugin marketplace add owlmastery/owlmastery_claude_plugin
/plugin install owlmastery@owlmastery
```

To install from a local checkout, run `/plugin marketplace add .` from the repository root instead.

## Sign in

You need an Owl Mastery account. After you install the plugin, sign in once:

1. Run `/mcp` in Claude Code, select `owlmastery`, and choose **Authenticate**. If you skip this step, the first tool call prompts you instead.
2. Your browser opens the Owl Mastery sign-in page. Sign in, then select **Allow** on the consent page.
3. Claude Code stores the tokens and refreshes them itself. Access tokens last 1 hour and are renewed without asking you. The connection stays until you disconnect it, however long it goes unused, so you sign in once.

You see and disconnect your connections in Owl Mastery under **Profile**, **Connected apps**.

## What the plugin adds

- **MCP server `owlmastery`**, streamable HTTP at `https://owlmastery.com/mcp`. The server answers 401 until you sign in, and Claude Code then runs the OAuth flow. The plugin holds no credential.
- **Skill `levels`** (`/owlmastery:levels NVDA`). Reads a ticker's call resistance, put support, HVL, gamma wall, net gamma, 1-day range and implied volatility against its close, and compares up to 10 tickers in one table. Claude also loads it on its own when you ask about those levels.
- **Skill `market`** (`/owlmastery:market`). Summarizes breadth, indexes, movers, sectors and your watchlists.
- **Skill `screen`** (`/owlmastery:screen`). Lists the stocks on one of the daily screens, such as negative gamma, below put support or RSI oversold, in the market or one sector, and counts how many stocks closed above or below their levels.

The server has nine read-only tools: `search_tickers`, `get_ticker_levels`, `get_gamma_exposure`, `get_volatility`, `get_market_overview`, `get_sectors`, `list_watchlists`, `screen_tickers` and `get_market_gamma`. The server describes their inputs itself, so the skills name the tools and leave the schemas out. Each skill's `allowed-tools` line lets Claude Code call the tools the skill uses without a prompt. A plugin's MCP tool is named `mcp__plugin_<plugin>_<server>__<tool>`, so `get_ticker_levels` of the `owlmastery` server in the `owlmastery` plugin is `mcp__plugin_owlmastery_owlmastery__get_ticker_levels`.

## Test against localhost

Start the Owl Mastery API on `http://localhost:8080` and the web app with `make frontend`, which proxies `/mcp` and `/oauth` on `http://localhost:5173`. Add the local server next to the plugin's, under another name:

```bash
claude mcp add --transport http owl-local http://localhost:5173/mcp
```

Run `/mcp`, select `owl-local` and choose **Authenticate**. The sign-in goes through the local app, so use a local account. Remove it when you are done:

```bash
claude mcp remove owl-local
```

## Check the plugin

```bash
claude plugin validate .
claude plugin validate plugins/owlmastery
```

To see the token cost, install the plugin and run `claude plugin details owlmastery`, or run `claude --plugin-dir plugins/owlmastery plugin details owlmastery` from a checkout. The three skills add about 780 tokens to every session. A skill's body loads only when it fires: about 2,200 tokens for `levels`, which includes its references, 1,000 for `market` and 1,600 for `screen`. These are the command's projected figures.

The suite in `plugins/owlmastery/evals/` checks that each skill fires on the prompts it should and stays out of the ones it should not. It needs no server, because the cases stop at the skill call. It runs a model, so it costs a few cents:

```bash
cd plugins/owlmastery
claude plugin eval . --ablation none --runs 1 --model haiku --no-publish
```

Drop `--runs 1` for three runs per case. Results go to `evals/results/`, which git ignores.

## What the plugin leaves out

- **No `userConfig` key prompt and no stored credential.** Sign-in is OAuth, so Claude Code holds the tokens and the plugin asks for nothing at install.
- **No commands, agents or hooks.** A skill already runs as `/owlmastery:levels`, so a command adds nothing. The skills only read data, so no agent needs its own context. Claude Code already reports a failed connection and offers to sign in again.
- **No MCPB bundle or MCP app widgets.** The server is remote HTTP, so there is nothing local to bundle. The answers are numbers and levels that read well as text.

## What works where

| Client | Works now | Notes |
| --- | --- | --- |
| Claude Code | Yes | OAuth sign-in in the browser. |
| Claude.ai and Claude Desktop | Expected, not yet confirmed | Add the plugin under **Customize**, **Plugins**, then select **Connect** on its **Connectors** tab and sign in. In a Team or Enterprise organization, Connect stays greyed out until an owner adds a custom connector with the URL `https://owlmastery.com/mcp` under **Organization settings**, **Connectors**. Needs Owl Mastery to let server-to-server traffic through Cloudflare. A directory listing is pending. |
| ChatGPT and Codex | See the Codex plugin repository | |

## Keep the skills in sync

The skills in `plugins/owlmastery/skills/` also ship in the Codex plugin repository. After you edit a skill in either repository, copy its `SKILL.md` body and its `references/` folder to the other repository unchanged. Each skill folder has its own copy of `references/connect.md`, the steps to connect each client, and every copy is the same file. The only difference allowed is the `allowed-tools` line in the frontmatter, which only the Claude Code copy has: the Claude Code copy keeps it and the Codex copy leaves it out. The skill text has no client-specific wording, so the body needs no edits. Run `diff -r` on the two `skills/` folders. It must show only the `allowed-tools` lines.

## Layout

```text
.claude-plugin/marketplace.json     the marketplace, named owlmastery
plugins/owlmastery/
  .claude-plugin/plugin.json        the plugin manifest
  .mcp.json                         the MCP server URL
  skills/levels/                    SKILL.md, references/glossary.md and references/connect.md
  skills/market/                    SKILL.md and references/connect.md
  skills/screen/                    SKILL.md and references/connect.md
  evals/                            skill-trigger cases for claude plugin eval
```

The data is educational information, not investment advice.
