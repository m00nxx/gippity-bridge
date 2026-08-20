# Gippity Bridge OpenCode engineering notes

The short setup guide is
[`integrations/opencode/README.md`](../integrations/opencode/README.md). This
page records the integration contract and the main maintenance priorities.

## Current contract

- Gippity Bridge exposes `/v1/models` and `/v1/chat/completions`.
- OpenCode executes tools; the bridge only prepares and validates tool calls.
- `@optimized` compresses tool schemas and conversation context.
- `@opencode` preserves more agent context at the cost of larger, slower turns.
- GPT-5.6 Web supports selectable `instant`, `medium`, and `high` effort.
- OpenCode image attachments are converted to multimodal input and uploaded
  through the captured ChatGPT Web session.
- Account pools support sticky, failover, random, round-robin, weighted, and
  quota-aware routing.

## Compatibility identifiers

The display name is **Gippity Bridge**, but these older technical identifiers
remain intentionally stable:

- Python package: `chatgpt_api`
- environment variables: `CHATGPT_*`
- provider ID used by the setup helper: `chatgpt-web`
- legacy command alias: `chatgpt-api`
- local setup state: `~/.config/chatgpt-api/opencode-setup.json`

Keeping them avoids breaking existing captures, scripts, and OpenCode configs.

## Tool-call lifecycle

1. OpenCode sends the prompt, available tools, and any attachments.
2. The selected prompt profile serializes the agent context for ChatGPT Web.
3. The bridge parses the assistant output and accepts only valid tool-call JSON.
4. OpenCode asks for permission and executes the selected local tool.
5. The result returns as the next conversation message.

The bridge must never execute an OpenCode tool itself or silently bypass the
client's permission policy.

## Attachments

The OpenAI-compatible route accepts text plus image content blocks. Inputs may
be local files supplied by the client, data URLs, or reachable HTTP(S) URLs.
Keep the provider model declaration aligned with this contract:

```json
{
  "attachment": true,
  "modalities": {
    "input": ["text", "image"],
    "output": ["text"]
  }
}
```

Text files should normally be read by OpenCode tools so only relevant excerpts
enter the model context. Image generation remains a separate
`/v1/images/generations` operation.

## Limits and retries

- Prefer `auto` when browser-style fallback is desired.
- Use an explicit model when model-limit errors must remain visible.
- Treat unreported quota as unknown, not unlimited.
- Preserve reset timestamps and selected-account metadata in normalized errors.
- Keep concurrency conservative; repeated retries can create duplicate tool
  calls and consume Web quota.

## Maintenance priorities

1. Keep account capture parsing resilient without logging secrets.
2. Detect Web payload or endpoint changes with focused transport tests.
3. Preserve OpenCode tool-call validation across SDK updates.
4. Keep image attachments and effort variants covered by regression tests.
5. Make retries explicit in logs so duplicate actions can be diagnosed.
