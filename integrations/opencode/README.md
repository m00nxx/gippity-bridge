# Gippity Bridge + OpenCode

Gippity Bridge exposes a local OpenAI-compatible endpoint. OpenCode connects to
that endpoint as a custom provider and remains responsible for reading files,
writing patches, and running shell commands.

> [!IMPORTANT]
> Start the bridge on `127.0.0.1` and keep account captures private. They contain
> live browser credentials.

## Quick setup

Start the API from the repository root:

```bash
export CHATGPT_API_KEY='choose-a-long-local-key'
gippity-bridge serve --host 127.0.0.1 --port 8000 --agent-mode optimized
```

PowerShell:

```powershell
$env:CHATGPT_API_KEY = 'choose-a-long-local-key'
gippity-bridge serve --host 127.0.0.1 --port 8000 --agent-mode optimized
```

Run the included setup helper:

```bash
bun integrations/opencode/chatgpt-opencode.mjs
```

It updates the normal OpenCode configuration and does not copy ChatGPT account
captures. Afterwards, use OpenCode normally:

```bash
opencode .
```

`chatgpt-api` remains available as a compatibility alias for
`gippity-bridge`.

## Manual provider configuration

Add a provider to `~/.config/opencode/opencode.jsonc`:

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

The provider ID (`gippity` above) is local and can be changed. The bundled
helper keeps the historical `chatgpt-web` ID so existing OpenCode settings keep
working while the UI displays **Gippity Bridge**.

## Models and effort

- `auto@optimized` uses ChatGPT Web's browser-style model selection with the
  compact agent prompt.
- `gpt-5-6@optimized` selects GPT-5.6 Web explicitly.
- OpenCode variants map `Instant`, `Medium`, and `High` to the Web account's
  reasoning-effort setting.
- `@opencode` preserves more of OpenCode's original context and tool
  documentation. It is more capable on complex jobs but sends more text and is
  usually slower.

Use `Ctrl+T` in OpenCode to cycle variants when the current model defines them.

## Images and pasted screenshots

This fork accepts OpenCode image attachments. A pasted screenshot is converted
to OpenAI multimodal content, uploaded through the authenticated ChatGPT Web
session, and attached to the conversation turn. The model declaration must
include:

```json
{
  "attachment": true,
  "modalities": {
    "input": ["text", "image"],
    "output": ["text"]
  }
}
```

Supported inputs include local image files, data URLs, and reachable HTTP(S)
image URLs. Up to ten images are accepted by the bridge for a single request.

## Tool-call flow

```text
OpenCode sends prompt + tool schemas
             ↓
Gippity Bridge compresses and validates the request
             ↓
ChatGPT Web returns a structured tool choice
             ↓
OpenCode executes the tool under its permission rules
             ↓
The tool result is sent through the bridge for the next turn
```

The bridge does not directly run `shell`, edit files, or bypass OpenCode
permissions. Repeated shell entries normally mean the model requested the same
tool more than once; inspect the OpenCode session and bridge logs to identify
retries.

## Helper commands

Inject or update the provider:

```bash
bun integrations/opencode/opencode-config.mjs --inject \
  --base-url http://127.0.0.1:8000/v1 \
  --api-key local-dev-key \
  --model chatgpt-web/gpt-5-6@optimized
```

Inspect the resolved configuration:

```bash
bun integrations/opencode/chatgpt-opencode.mjs --debug-config
```

Remove only the injected provider:

```bash
bun integrations/opencode/opencode-config.mjs --eject \
  --base-url http://127.0.0.1:8000/v1 \
  --api-key local-dev-key
```

## Troubleshooting

| Symptom | Check |
| --- | --- |
| OpenCode cannot list models | Bridge is running and the base URL ends in `/v1` |
| `401 Unauthorized` | `CHATGPT_API_KEY` matches the bridge key |
| ChatGPT authentication error | Refresh the saved account capture |
| Pasted image is ignored | Model has `attachment: true` and image input modality |
| Response is very slow | Use `@optimized` with `Instant`; avoid `@opencode` for small tasks |
| Duplicate tool calls | Check for retries or repeated tool-call JSON in the session logs |

For account capture instructions see
[`docs/ACCOUNT_CAPTURE.md`](../../docs/ACCOUNT_CAPTURE.md). For the complete API
surface see [`docs/OPENAI_COMPATIBILITY.md`](../../docs/OPENAI_COMPATIBILITY.md).
