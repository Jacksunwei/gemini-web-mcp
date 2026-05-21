# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex CLI, Gemini CLI, Cursor, etc.) working in this repo.

## Repository purpose

This repo is a **single MCP server** (`gemini-web`) packaged so it can be installed by any MCP client. It exposes three
tools — `web_search` (Gemini's `google_search` grounding), `summarize_pages` (Gemini's `url_context` tool), and
`generate_image` (Gemini's "Nano Banana" image model).

The repo is also packaged as a **one-plugin Claude Code marketplace** (via the `.claude-plugin/` directory at the root),
so Claude Code users can install it directly with `/plugin marketplace add Jacksunwei/gemini-web-mcp`. Other MCP clients
(Gemini CLI, Codex CLI, Antigravity) install the same `server/server.py` via their own config formats — see README.
It is also packaged as a **one-plugin Codex marketplace** via `.agents/plugins/marketplace.json`, `.codex-plugin/`, and
`.mcp.json`.

## Architecture

Five coordinating files at the repo root:

1. **`.claude-plugin/marketplace.json`** — mini-marketplace manifest. One entry, `"source": "./"`. Only consumed by
   Claude Code; other clients ignore it.
2. **`.claude-plugin/plugin.json`** — Claude Code plugin manifest. Declares `mcpServers` and `userConfig`. Use
   `${CLAUDE_PLUGIN_ROOT}` for paths into the plugin (e.g. `${CLAUDE_PLUGIN_ROOT}/server/server.py`) — never hardcode
   absolute paths. Only consumed by Claude Code.
3. **`.agents/plugins/marketplace.json`** — Codex marketplace manifest. One entry, `"source.path": "./"`, because the
   repo root is itself the plugin root.
4. **`.codex-plugin/plugin.json`** and **`.mcp.json`** — Codex plugin manifest and bundled MCP server config. Use
   `${PLUGIN_ROOT}` for paths into the installed plugin.
5. **`server/server.py`** — the MCP server itself. Uses **PEP 723 inline script metadata** (the `# /// script` block at
   the top) so `uv run --script` auto-installs Python deps on first launch. There is no `pyproject.toml` or
   `requirements.txt` — dependencies live inside the script.

Editing any plugin metadata file in isolation can break installation for that client. Keep marketplace descriptions,
versions, MCP server names, and server paths in sync. The server itself is client-agnostic.

## Auth model

Auth precedence (server-side):

1. **Plugin config `gemini_api_key`** (Claude Code `CLAUDE_PLUGIN_OPTION_gemini_api_key`, Codex
   `CODEX_PLUGIN_OPTION_gemini_api_key` / mapped `GEMINI_API_KEY`) — when set, the server constructs
   `genai.Client(api_key=..., vertexai=False)`, hard-overriding Vertex AI env vars.
2. **Otherwise the `google-genai` SDK auto-selects from the environment:**
   - `GOOGLE_API_KEY` set → Gemini API mode (individual users / AI Studio key).
   - `GOOGLE_GENAI_USE_VERTEXAI=true` + `GOOGLE_CLOUD_PROJECT` + ADC → Vertex AI mode (enterprise / Google-internal).

Models are also configured via plugin config or env vars (`search_model` / `GEMINI_SEARCH_MODEL`, `image_model` /
`GEMINI_IMAGE_MODEL`) — defaults
`gemini-flash-latest` and `gemini-3.1-flash-image-preview`. The search model **must support both `google_search`
grounding and the `url_context` tool** — not all Gemini models do.

## Common commands

```bash
# Run the MCP server standalone (smoke-test; serves stdio MCP protocol)
uv run --script server/server.py

# Install this plugin locally for end-to-end testing in Claude Code
# (substitute the absolute path to this repo on your machine):
/plugin marketplace add /path/to/gemini-web-mcp
/plugin install gemini-web@gemini-web-mcp
```

There is no test suite, linter config, or build step. Validate changes by:
1. Running the server standalone to catch import/syntax errors.
2. Installing the plugin locally (Claude Code) or pointing your client's MCP config at it, then exercising the tools.

## Conventions

- **Python indentation: 2 spaces** (see `server.py`). This is unusual for Python — match it.
- **Apache 2.0 header** on every Python source file (copyright `Wei (Jack) Sun`).
- Keep dependencies in the PEP 723 block, **not** in a separate requirements file. The whole point of the layout is
  single-file deployability via `uv`.
- Plugin description appears in three places — `marketplace.json`, `plugin.json`, and `README.md` — and they must stay
  in sync (hand-maintained).
