---
name: docs-search
description: >-
  Search and query the Grounded Docs MCP Server documentation index.
  Covers listing indexed libraries, searching documentation content,
  and resolving library versions. Use when you need to look up API
  references, find code examples, or check which documentation is
  available in the local index.
compatibility: Requires Node.js 22+ and npx
metadata:
  author: grounded.tools
---

# Docs Search

Search the local Grounded Docs index for library documentation. These commands
return structured data (JSON by default in non-interactive sessions) and never
modify the index.

## When to use

- You need to answer a question about a library's API or behaviour.
- You want to check which libraries and versions are already indexed.
- You need to resolve which documentation version best matches a constraint.

Always run `list` first if you are unsure whether a library has been indexed.
If nothing is indexed yet, use the **docs-manage** skill to scrape documentation
before searching.

## Commands

### list

List every indexed library and its available versions.

```bash
npx @arabold/docs-mcp-server@latest list [--output yaml]
```

| Flag | Description |
|------|-------------|
| `--output json\|yaml\|toon` | Structured output format (default: JSON in non-interactive) |
| `--server-url <url>` | Connect to a remote pipeline worker instead of the local store |
| `--quiet` | Suppress non-error diagnostics |
| `--verbose` | Enable debug logging |

Example output (YAML):

```yaml
- library: react
  versions:
    - "19.0.0"
    - "18.3.1"
- library: typescript
  versions:
    - "5.7.0"
```

### search

Search documents in an indexed library by natural-language query.

```bash
npx @arabold/docs-mcp-server@latest search <library> "<query>" [options]
```

| Flag | Alias | Default | Description |
|------|-------|---------|-------------|
| `--version <ver>` | `-v` | latest | Version constraint (exact or range, e.g. `18.x`, `5.2.x`) |
| `--limit <n>` | `-l` | `5` | Maximum number of results |
| `--exact-match` | `-e` | `false` | Only match the exact version, no fallback |
| `--embedding-model <model>` | | | Embedding model (e.g. `openai:text-embedding-3-small`) |
| `--server-url <url>` | | | Remote pipeline worker URL |
| `--output json\|yaml\|toon` | | JSON | Structured output format |
| `--quiet` | | | Suppress non-error diagnostics |
| `--verbose` | | | Enable debug logging |

Example:

```bash
npx @arabold/docs-mcp-server@latest search react "useEffect cleanup" --version 18.x --limit 3 --output yaml
```

Each result contains a content snippet, source URL, and relevance score.

### find-version

Resolve the best matching documentation version for a library.

```bash
npx @arabold/docs-mcp-server@latest find-version <library> [--version <pattern>]
```

| Flag | Alias | Description |
|------|-------|-------------|
| `--version <pattern>` | `-v` | Version pattern to match (supports ranges) |
| `--server-url <url>` | | Remote pipeline worker URL |
| `--output json\|yaml\|toon` | | Structured output format |
| `--quiet` | | Suppress non-error diagnostics |
| `--verbose` | | Enable debug logging |

Example:

```bash
npx @arabold/docs-mcp-server@latest find-version react --version "18.x" --output yaml
```

Returns the resolved version string and library metadata.

## Interpreting output

All three commands emit structured data to **stdout**. Diagnostics and progress
messages go to **stderr** and are suppressed by default in non-interactive
sessions. Use `--verbose` (or set `LOG_LEVEL=INFO`) to re-enable them.
Use `--quiet` to suppress all non-error diagnostics regardless of session type.

To capture results programmatically, parse stdout as JSON (the default) or
request `--output yaml` for a more readable format.

Each search result contains these fields — map them to citations as follows:

| Search result field | Citation role |
|---------------------|---------------|
| `url` | Doc path (local) — the exact document URL recorded at scrape time; extract the path portion (everything after the domain) to show where in the docs the content lives. For local file indexes this will already be a file path. Content was served from the local index, not fetched live. |
| `content` | The snippet to quote verbatim |
| `score` | Relevance (do not surface to the user; use it to decide whether to trust the result) |
| `mimeType` | Content type — ignore for citations |

**Extracting the doc path from `url`:**
- Web URL `https://developer.hashicorp.com/vault/docs/auth/approle` → doc path `/vault/docs/auth/approle`
- Local file URL `file:///Users/cj/docs/vault/v1-21-x/content/docs/auth/approle/index.mdx` → doc path `vault/v1-21-x/content/docs/auth/approle/index.mdx`
- If the URL is only a root page (e.g. `https://pkg.go.dev/net/http@go1.15` with no deeper path), show the full URL as-is and note it is the root page.

## Fact-checking workflow

When the user asks you to fact-check a claim or document, use this process:

1. **Search** the index for the relevant claim using `search`.
2. **Locate** the exact sentence or passage in the result's `content` field that confirms, contradicts, or qualifies the claim.
3. **Classify** your finding as one of:
   - **`[DOCS SAY]`** — you are quoting or paraphrasing text that appears verbatim in the indexed source.
   - **`[INFERRED]`** — the source doesn't state this directly; you deduced it from surrounding context. Always name the source you inferred from.
   - **`[NOT FOUND]`** — no indexed source addresses the claim; say so explicitly rather than guessing.
4. **Report** using the citation block format below.

### Citation block format

For every claim you check, emit one citation block:

```
Doc path (local): <path portion of the url field — see extraction rules above>
Public URL:       <full url field value, so the reader can visit it>
Quote:            "<exact verbatim sentence(s) from content field>"
Finding:          [DOCS SAY] <your one-sentence finding>
                  — OR —
                  [INFERRED from <doc path>] <your one-sentence finding>
                  — OR —
                  [NOT FOUND] No indexed source addresses this claim.
```

**Rules:**
- The `Quote:` line must be a verbatim excerpt from the `content` field — never paraphrase it.
- If the content snippet is long, quote only the most directly relevant sentence(s) and use `…` for omitted text.
- Never use `[DOCS SAY]` unless you can point to an exact quote that supports it.
- The field label is `Doc path (local):` — this makes clear the content came from the local index. Never append `(local)` to the path value itself.
- If two results contradict each other, emit a citation block for each and note the conflict.

### Example citation block

```
Doc path (local): /vault/docs/auth/approle  ← path extracted from url field; served from local index
Public URL:       https://developer.hashicorp.com/vault/docs/auth/approle
Quote:            "AppRole is an authentication method in Vault that allows machines
                  or apps to authenticate with Vault-defined roles."
Finding:          [DOCS SAY] AppRole is designed for machine/app authentication, not human users.
```

## Typical workflow

```bash
# 1. Check what is indexed
npx @arabold/docs-mcp-server@latest list --output yaml

# 2. Search for relevant docs
npx @arabold/docs-mcp-server@latest search react "server components" --version 19.x --output yaml

# 3. If no results, index the docs first (see docs-manage skill), then retry

# 4. Fact-checking: run search, then emit citation blocks (see Fact-checking workflow above)
npx @arabold/docs-mcp-server@latest search vault "approle authentication" --version 1.21.x --limit 3 --output yaml
```
