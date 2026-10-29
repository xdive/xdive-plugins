# XDive plugins for Claude Code and Codex

[XDive](https://xdive.dev) gives AI agents web search, page reading as Markdown, site
crawling, data extraction and cited research. This repository is a plugin marketplace
for [Claude Code](https://claude.com/claude-code) and [Codex](https://developers.openai.com/codex)
(also the ChatGPT desktop app).

The plugin connects the hosted XDive MCP server (`https://xdive.dev/api/mcp`), so you
sign in once in the browser and never copy an API key. It adds two skills: `web-research`
(which tool to use, how to read long pages, how to cite sources) and `xdive-api` (for
agents that write code against the REST API).

## Install

Claude Code:

```bash
claude plugin marketplace add xdive/xdive-plugins
claude plugin install xdive@xdive-plugins
```

Then run `/mcp` in a session, select **xdive** and sign in.

Codex:

```bash
codex plugin marketplace add xdive/xdive-plugins
```

Then install **XDive** from the **XDive** marketplace (`/plugins` in the Codex CLI) and
sign in when asked.

## What you get

Eight tools: `search`, `fetch`, `research`, `scrape`, `map`, `crawl`, `crawl_status` and
`extract`. Long pages continue with an offset, and every research answer keeps its
numbered sources.

Docs: https://xdive.dev/docs/mcp · Docs for agents: https://xdive.dev/llms.txt

## Licence

MIT, see [LICENSE](LICENSE).
