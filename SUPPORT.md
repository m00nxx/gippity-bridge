# Gippity Bridge support policy

Gippity Bridge is an experimental, community-maintained fork distributed under
the MIT license. It is a developer tool and reference implementation, not a
hosted service or a product with guaranteed support.

## Before opening an issue

Please include:

- operating system, Python version, and installation method;
- the exact command and complete error message;
- whether the failure affects the API, OpenCode, Docker, or the console;
- a minimal reproduction using redacted or synthetic data;
- the result of `python -m pytest -q` when relevant.

Never include cookies, bearer tokens, copied browser requests, HAR files,
account captures, personal account names, or a populated `.env`.

## Useful reports

Issues and pull requests are most useful when they cover:

- reproducible setup or compatibility bugs;
- a ChatGPT Web endpoint change with redacted evidence;
- OpenCode tool-call or image-attachment regressions;
- security and secret-handling improvements;
- small documentation fixes with verified commands.

## Security

Do not open a public issue containing a live credential or an exploit against
another person's account or system. Revoke exposed credentials immediately.
This bridge is intended for your own authenticated session on a trusted local
machine.

## Upstream

The original project is
[`suphotP/chatgpt-api`](https://github.com/suphotP/chatgpt-api). Check both
projects before filing a report that may already be addressed upstream.
