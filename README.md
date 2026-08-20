<p align="center">
  <img src="docs/assets/brand/gippity-bridge-hero.png" alt="Gippity Bridge connecting a web browser to a coding terminal" width="100%">
</p>

<h1 align="center">Gippity Bridge</h1>

<p align="center">
  <strong>Your ChatGPT Web session, wearing a local API and walking into OpenCode.</strong>
</p>

<p align="center">
  <a href="LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-35d0ba"></a>
  <img alt="Python 3.11+" src="https://img.shields.io/badge/python-3.11%2B-51b7e8">
  <img alt="Local first" src="https://img.shields.io/badge/runtime-local--first-f4b942">
  <img alt="OpenCode ready" src="https://img.shields.io/badge/OpenCode-ready-8be9fd">
</p>

Gippity Bridge turns an authenticated ChatGPT Web session into a local,
OpenAI-shaped API for chat, tools, vision, images, research, and agent
workflows. It is designed for one developer on one trusted machine: your
credentials stay local, clients talk to `127.0.0.1`, and OpenCode remains in
charge of file and shell permissions.

> [!WARNING]
> This is an unofficial, experimental bridge. It is not affiliated with or
> supported by OpenAI. ChatGPT Web endpoints can change without notice. Never
> publish account captures, cookies, bearer tokens, or a bridge containing
> real credentials.

## Why Gippity?

- **One local endpoint** for ChatGPT Web capabilities.
- **OpenCode integration** with validated tool calls and pasted image support.
- **GPT-5.6 Web effort controls**: Instant, Medium, and High.
- **Vision and OCR** for up to ten input images.
- **Image generation, editing, and compositing** with saved artifacts.
- **Deep Research** exported as Markdown.
- **Account routing** with failover, round-robin, weighted, or quota-aware strategies.
- **Operator console** for accounts, limits, tests, and artifacts.

## What works

| Capability | Surface | Status |
| --- | --- | --- |
| Chat and streaming | `/v1/chat/completions` | Ready |
| OpenCode tool calls | OpenAI-style `tools` | Ready |
| Pasted images in OpenCode | Multimodal chat content | Ready in this fork |
| OCR and image understanding | `/v1/chatgpt/vision` | Ready |
| Image generation | `/v1/images/generations` | Ready |
| Image editing/compositing | `/v1/images/edits` | Ready |
| Deep Research | `chatgpt-deep-research` model alias | Ready |
| Artifact downloads | `/v1/chatgpt/files/{id}/{filename}` | Ready |
| Usage and capacity | `/v1/chatgpt/usage` | When reported by ChatGPT |

## How it works

```text
OpenCode / app
      │  OpenAI-shaped request
      ▼
Gippity Bridge  ── validates tools, routes accounts, uploads media
      │  authenticated local capture
      ▼
ChatGPT Web
```

For tool calls, ChatGPT chooses an action and returns validated JSON. OpenCode
executes the actual read, write, patch, or shell operation under its own
permission rules, then sends the result back for the next turn. The bridge
never executes OpenCode tools itself.

## Quick start

### 1. Install

```bash
git clone https://github.com/m00nxx/gippity-bridge.git
cd gippity-bridge
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install -e ".[dev]"
```

macOS/Linux:

```bash
source .venv/bin/activate
python -m pip install -e '.[dev]'
```

Both command names are available:

```text
gippity-bridge   preferred
chatgpt-api      compatibility alias
```

### 2. Capture one ChatGPT Web request

Open ChatGPT in a dedicated browser profile, open DevTools, send a harmless
message, and copy the conversation request as cURL. Import it locally:

```bash
gippity-bridge admin account add --paste --account main
```

Paste the complete cURL request and finish with `END_CAPTURE` on its own line.
The capture is stored locally under the Git-ignored `secrets/accounts/` tree.
Treat that directory like a password vault and protect it with your OS account.

See [Account capture](docs/ACCOUNT_CAPTURE.md) for browser-specific steps and safe handling.

### 3. Start the API

```bash
export CHATGPT_API_KEY='choose-a-long-local-key'
gippity-bridge serve \
  --host 127.0.0.1 \
  --port 8000 \
  --agent-mode optimized \
  --temporary-chat
```

PowerShell:

```powershell
$env:CHATGPT_API_KEY = 'choose-a-long-local-key'
gippity-bridge serve --host 127.0.0.1 --port 8000 --agent-mode optimized --temporary-chat
```

Check it:

```bash
curl http://127.0.0.1:8000/health
curl -H "Authorization: Bearer choose-a-long-local-key" http://127.0.0.1:8000/v1/models
```

### 4. Connect OpenCode

Add a custom OpenAI-compatible provider to `~/.config/opencode/opencode.jsonc`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "gippity/gpt-5-6@optimized",
  "provider": {
    "gippity": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Gippity Bridge",
      "options": {
        "baseURL": "http://127.0.0.1:8000/v1",
        "apiKey": "{env:CHATGPT_API_KEY}"
      },
      "models": {
        "gpt-5-6@optimized": {
          "name": "GPT-5.6 Sol Web",
          "reasoning": true,
          "attachment": true,
          "modalities": {
            "input": ["text", "image"],
            "output": ["text"]
          },
          "variants": {
            "instant": { "reasoningEffort": "instant" },
            "medium": { "reasoningEffort": "medium" },
            "high": { "reasoningEffort": "high" }
          }
        }
      }
    }
  }
}
```

OpenCode uses `Ctrl+T` to cycle model variants. The Web effort setting belongs
to the ChatGPT account, so the last selected Instant/Medium/High value is also
visible in ChatGPT Web.

The included helper can inject or remove a provider automatically:

```bash
bun integrations/opencode/chatgpt-opencode.mjs
```

## API in sixty seconds

Chat:

```bash
curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Authorization: Bearer $CHATGPT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5-6",
    "reasoning_effort": "instant",
    "messages": [{"role": "user", "content": "Reply with: bridge crossed"}]
  }'
```

Vision:

```bash
gippity-bridge api vision \
  --input-image ./screenshot.png \
  --mode describe \
  --prompt "Describe the visible error" \
  --base-url http://127.0.0.1:8000/v1 \
  --api-key "$CHATGPT_API_KEY"
```

Image generation:

```bash
gippity-bridge api image \
  --prompt "A tiny robot crossing a glowing data bridge" \
  --base-url http://127.0.0.1:8000/v1 \
  --api-key "$CHATGPT_API_KEY"
```

## Operator console

The Svelte console shows health, accounts, quotas, API documentation, stored
artifacts, and OpenCode setup.

```bash
bun --cwd apps/bridge-console install
bun --cwd apps/bridge-console dev
```

<p align="center">
  <img src="docs/assets/screenshots/oss-console-overview.png" alt="Gippity Console overview" width="49%">
  <img src="docs/assets/screenshots/oss-console-opencode.png" alt="Gippity OpenCode setup" width="49%">
</p>

## Docker

```bash
cp .env.example .env
docker compose up --build
```

| Service | Address |
| --- | --- |
| API | `http://127.0.0.1:8000/v1` |
| Console | `http://127.0.0.1:8080` |
| Character game demo | `http://127.0.0.1:3000` |

Keep Docker volumes containing `secrets/` and `outputs/` private.

## Documentation

Start with the short [documentation index](docs/README.md):

- [Account capture](docs/ACCOUNT_CAPTURE.md)
- [CLI reference](docs/CLI.md)
- [OpenAI-shaped API](docs/OPENAI_COMPATIBILITY.md)
- [OpenCode integration](integrations/opencode/README.md)
- [Docker](docs/DOCKER.md)
- [Architecture](docs/ARCHITECTURE.md)

## Validation

```bash
python -m pytest -q
bun --cwd apps/bridge-console check
bun --cwd apps/bridge-console build
```

On Windows, the upstream POSIX-mode assertion in
`test_load_secrets_key_creates_owner_only_key_file` can report `0666` instead
of `0600`; real capture protection should be verified with NTFS ACLs.

## Security and limits

- Bind to `127.0.0.1` unless you deliberately harden a LAN deployment.
- Treat captures like passwords and never commit `secrets/`, `.env`, HAR, or copied request files.
- Keep concurrency low; Web quotas and anti-abuse limits can be hidden.
- ChatGPT subscriptions and official OpenAI API billing are separate.
- This is a developer bridge, not a public multi-tenant proxy.
- It is not a complete clone of every OpenAI API endpoint.

## Lineage

Gippity Bridge is a friendly fork of
[`suphotP/chatgpt-api`](https://github.com/suphotP/chatgpt-api). The original
architecture, transport, console, demos, and MIT license remain credited in
the Git history and license. This fork adds GPT-5.6 Web support, selectable Web
effort, direct OpenCode image attachments, Windows-focused setup, and a lighter
project presentation.

## License

[MIT](LICENSE). Keep the unofficial-use warning and protect browser captures.
