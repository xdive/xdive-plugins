---
name: web-research
description: Use when a task needs current or external information from the public web - finding pages, reading a page or a whole site, pulling structured data out of pages, or answering a question with cited sources. Covers the XDive MCP tools search, fetch, research, scrape, map, crawl, crawl_status and extract.
---

# Web research with XDive

All XDive tools only read the public web. Pick the cheapest tool that answers the need.

## Choose the tool

| Need | Tool |
|---|---|
| Find pages about something | `search` (add `sources: ["news"]` and `freshness` such as `pw` for recent events) |
| Read one known URL | `fetch` with the URL as `id` (a search result's `id` works as is) |
| A synthesized answer from several sources, with citations | `research` (`depth: "quick"` by default; `"deep"` for broad or contested questions) |
| A page in another format: links, html, summary, images, product | `scrape` with `formats` |
| Find the right page on a site before reading it | `map` with `search` (lists URLs only, reads nothing) |
| Read many pages of one site | `crawl` with `include_paths` and a small `max_pages`; then `crawl_status` |
| Fields or tables from up to 10 pages | `extract` with a `prompt` and, for a fixed shape, a JSON `schema` |

Prefer `search` then `fetch` when you need to read precisely and quote; prefer `research` when the user wants an answer, not documents. Do not crawl a site to read one page: `map` with `search`, then `fetch`.

## Read long pages in parts

`fetch` and `scrape` return about 24,000 characters at a time.

- `fetch` returns JSON; when `metadata.next_offset` is present the page continues. Call `fetch` again with the same `id` and `offset` set to that value.
- `scrape` ends a partial page with `[Page continues: … Call scrape again with offset=N …]`. Call it again with `offset=N`.
- Read on only while the missing part matters. Raise `max_chars` (up to 60,000) only when you need a large section at once.

## Research that is still running

`research` waits about 40 seconds. If it answers that the research is still running, call `research` again with exactly the same arguments after the suggested wait. The run continues on the server and the repeat returns its answer instead of starting over. Do not change the query or depth between calls, and do not switch to another tool in the meantime unless the user asks.

## Cite what you use

- `research` returns an answer whose claims carry markers such as [1], followed by a numbered `Sources:` list (title and URL). Keep the markers and list the sources you relied on, with their URLs.
- When you answer from `search` or `fetch` results, link each claim to the page it came from. Never cite a URL you did not read or that no tool returned.
- Report the `Confidence:` line when research marks the answer low or medium confidence.

## Crawls

`crawl` waits about 40 seconds. A crawl still running returns its id (`crl_…`): call `crawl_status` with that id a little later. Each crawled page appears as an excerpt; `fetch` any listed URL to read it in full (crawled pages are cached, so this is fast).

## Errors

A failed call returns text such as `Error 403 (SITE_RESTRICTED): message`. Read it and act on it; do not retry the same call unchanged. A site that refuses access (403) or has nothing readable (404) is a reason to try another source. A used-up quota (402), a missing scope or an account problem (403), or too many requests (429) is something to tell the user, not to work around.
