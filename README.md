# Gemini Web — MCP Server

![Search, summarize, and generate — three Gemini-powered tools for any MCP client](docs/hero.png)

**Real Google Search, multi-page summaries, and Nano Banana image generation — for any MCP client.**

Three Gemini-powered MCP tools:

- **`web_search`** — real Google Search via Gemini's grounding, with cited source URLs.
- **`summarize_pages`** — fetch and synthesize up to 20 URLs in a single call (HTML, PDF, JSON, images — up to 34 MB
  each).
- **`generate_image`** — text-to-image, image editing, and multi-image fusion via Gemini's "Nano Banana" model, saved to
  disk.

Drop-in upgrades to your client's built-in web tools — broader coverage, one-shot multi-URL synthesis — plus image
generation many clients don't ship at all. Especially useful on Claude Code with Bedrock or Vertex Anthropic backends
that don't ship a built-in WebSearch.

## Usage

Just ask the model. A few examples:

**`web_search`:**

> Use Gemini to research all the major image-generation models and compare pros and cons.

**`summarize_pages`:**

> Summarize key changes of the paper in \<url>.

**`generate_image`:**

> Generate an image of a retro 8-bit banana floating in space, save it as `/Users/me/proj/assets/hero.png`.

> Take `/Users/me/Downloads/sketch.png` and turn it into a watercolor painting.

## Install

The server is a single Python file with [PEP 723 inline script metadata](https://peps.python.org/pep-0723/), so
`uv run --script` auto-installs deps on first launch. [Install `uv`](https://docs.astral.sh/uv/getting-started/installation/)
once and you're set.

### Claude Code

Standalone (this repo is itself a one-plugin marketplace):

```bash
/plugin marketplace add Jacksunwei/gemini-web-mcp
/plugin install gemini-web@gemini-web-mcp
```

Or via the [`jacksunwei-marketplace`](https://github.com/Jacksunwei/jacksunwei-marketplace) index marketplace:

```bash
/plugin marketplace add jacksunwei/jacksunwei-marketplace
/plugin install gemini-web@jacksunwei-marketplace
```

### Gemini CLI, Codex CLI, Antigravity, and other MCP clients

Clone the repo somewhere stable:

```bash
git clone https://github.com/Jacksunwei/gemini-web-mcp.git ~/mcp/gemini-web-mcp
```

Then register the server with your client (substitute your absolute path).

**Gemini CLI** — edit `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "gemini-web": {
      "command": "uv",
      "args": ["run", "--script", "/Users/you/mcp/gemini-web-mcp/server/server.py"],
      "env": {
        "GOOGLE_API_KEY": "your-aistudio-key"
      }
    }
  }
}
```

Or install it as a Codex plugin marketplace:

```bash
codex plugin marketplace add Jacksunwei/gemini-web-mcp
```

Then open the Codex plugin directory and install **Gemini Web** from the `gemini-web-mcp` marketplace.

**Codex CLI** — edit `~/.codex/config.toml`:

```toml
[mcp_servers.gemini-web]
command = "uv"
args = ["run", "--script", "/Users/you/mcp/gemini-web-mcp/server/server.py"]

[mcp_servers.gemini-web.env]
GOOGLE_API_KEY = "your-aistudio-key"
```

**Antigravity** — open the Agent Manager → MCP store → "Add Custom Server", and use the same `command` / `args` / `env`
shape as above.

For clients without plugin install prompts, configure auth via env vars (the `GOOGLE_API_KEY` shown above, or Vertex ADC
— see [Advanced: env-var auth](#advanced-env-var-auth) below). Claude Code can use either env vars or the plugin's
install-time config fields.

## Configure

**First time:** Claude Code prompts you for the fields below right after `/plugin install`. Fill in the API key (the rest
can stay blank for defaults).

**Later:** to change any setting, run `/plugin`, select **gemini-web**, and edit its config.

| Field                            | Default                          | Notes                                                                                                                   |
| -------------------------------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Gemini API key**               | _none_                           | Your [AI Studio key](https://aistudio.google.com/apikey). Stored in your system keychain.                               |
| **Search / summarization model** | `gemini-flash-latest`            | Used by `web_search` and `summarize_pages`. Must support both `google_search` grounding and the `url_context` tool.     |
| **Image generation model**       | `gemini-3.1-flash-image-preview` | Nano Banana 2. Override to `gemini-2.5-flash-image` (GA Nano Banana) or `gemini-3-pro-image-preview` (Nano Banana Pro). |

> Need Vertex AI or env-var auth instead? See [Advanced: env-var auth](#advanced-env-var-auth) below.

## Advanced: env-var auth

If you can't (or don't want to) use the plugin UI for the API key — for example you're on Vertex AI, sharing settings
across machines, or scripting installs — leave the **Gemini API key** field blank and set env vars instead. The
`google-genai` SDK auto-selects the auth path from your environment:

**Gemini API key (individual users):**

```bash
export GOOGLE_API_KEY=your-key   # https://aistudio.google.com/apikey
export GOOGLE_GENAI_USE_VERTEXAI=false   # only if previously set to true
```

**Vertex AI + ADC (enterprise / Google-internal):**

```bash
gcloud auth application-default login
gcloud auth application-default set-quota-project YOUR_PROJECT_ID
export GOOGLE_GENAI_USE_VERTEXAI=true
export GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
export GOOGLE_CLOUD_LOCATION=us-central1
# Vertex AI API must be enabled on the project.
```

If both the **Gemini API key** plugin field and `GOOGLE_*` env vars are set, the plugin field wins.

## License

Apache 2.0 — see [LICENSE](LICENSE).
