---
name: xdive-api
description: Use when writing or running code that calls the XDive REST API (https://xdive.dev/api) to scrape pages to Markdown, crawl or map sites, search the web, research a question with citations, or extract structured data. Covers auth, choosing the endpoint, polling jobs, errors, retries and idempotency.
---

# Calling the XDive REST API

Reference, as Markdown: https://xdive.dev/llms.txt (index) and https://xdive.dev/docs/<page>.md. Machine-readable: https://xdive.dev/openapi.json. Look parameters up there rather than guessing.

## From a shell: the CLI

When you can run commands, the `xdive` CLI is the shortest path: `npx xdive scrape <url>` prints the page as Markdown on stdout, `npx xdive search "<query>"` and `npx xdive research "<question>"` likewise; add `--json` for the raw response. It reads `XDIVE_API_KEY`; if there is no key, ask the user to run `npx xdive login` (they approve once in a browser). Warnings and the next `--offset` for long pages go to stderr. Exit codes: `2` usage, `3` not signed in or key refused, `4` quota used up (tell the user, do not retry), `5` rate limited (wait, then retry). See https://xdive.dev/docs/cli.md.

## Auth and paths

- Base URL `https://xdive.dev/api`. Header `Authorization: Bearer $XDIVE_API_KEY` (or `X-API-Key`). Read the key from the environment; never print, log or commit it.
- Work endpoints are `/orgs/{orgId}/...`. Get `orgId` once: `GET /whoami` returns `{"orgId": ..., "scopes": [...]}`.
- Scopes: `scrape` (scrape, parse), `crawl` (crawl, map), `search` (search, research). A `403` naming a scope means the key lacks it; tell the user.

## Choose the endpoint

| Need | Call |
|---|---|
| One URL as Markdown | `POST /orgs/{id}/scrape` `{"url": "...", "formats": ["markdown"]}`; read `data.artifacts.markdown.content` |
| Find pages | `POST /orgs/{id}/search` `{"query": "...", "limit": 5}` |
| A cited answer | `POST /orgs/{id}/research` `{"query": "...", "depth": "quick"}` |
| Fields from up to 10 pages | `POST /orgs/{id}/extract` `{"urls": [...], "prompt": "...", "schema": {...}}` |
| A site's URLs (free) | `POST /orgs/{id}/map` `{"url": "...", "search": "..."}` |
| Many pages of a site | `POST /orgs/{id}/crawl` `{"url": "...", "max_pages": 50, "include_paths": [...]}` |
| A list of up to 100 URLs | `POST /orgs/{id}/batch/scrape` `{"urls": [...]}` |

Do not crawl to read one page: map with `search`, then scrape it.

Long pages: send `"max_chars": 8000` and continue with `"offset"` set to the returned `data.artifacts.markdown.next_offset`; `estimated_tokens` says what a part costs to read. A page whose content looks incomplete carries `data.warnings`: retry once with `"wait_for": 3000`, or tell the user.

## Jobs

`crawl` and `batch/scrape` answer `202` with `{"id": "crl_..."}`. Poll `GET /orgs/{id}/crawls/{crawlId}` until `status` is `success`, `error` or `cancelled`, waiting `Retry-After` seconds between polls and sending the last `ETag` as `If-None-Match` (a `304` means unchanged). Failed pages stay in `results` with `error` and sometimes `error_code`.

## Errors and retries

- Error bodies: `{"error": "...", "code": "..."}`. Branch on the status and `code`, never on `error` text. Quote `X-Request-ID` when reporting a failure.
- `429`, `503`: wait `Retry-After` seconds (1 if absent) and retry, at most a few times. Watch `RateLimit-Remaining` to slow down early.
- `402` (quota used up), `401`, `403`, other `4xx`: do not retry unchanged; fix the request or tell the user.
- `502`, `504`: one retry may succeed.
- On a POST you may retry, send `Idempotency-Key: <uuid>` and reuse the same key on the retry. Within 24 hours the same key and body return the stored response with `Idempotent-Replayed: true`; the same key with a different body fails with `422` `IDEMPOTENCY_KEY_REUSED`.
- Use HTTP timeouts of at least 120 s (180 s for `"depth": "deep"` research).

## Costs

Scrapes and crawled pages count pages, search and research count searches and pages read, browser actions count browser seconds; map and polling are free. See https://xdive.dev/docs/usage.md.
