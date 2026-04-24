# Happy path: npm install cowsay under sandbox

**Layer:** 2, Sandbox (happy path)
**Claim:** With sandbox on, `autoAllowBashIfSandboxed: true`, npm cache dirs in `filesystem.allowWrite`, and `registry.npmjs.org` in allowed domains, `npx --yes cowsay@1.6.0 "sandbox test"` runs silently. The ASCII cow prints with no prompts.
**Finding:** `npx` completed in 30s, silent, exit 0. Same shape on a second run with `2>&1`.
**Claude Code:** 2.1.116 or 2.1.117
**Harness source:** `sandbox-11-package-registries`

## Setup

Sandbox on, `autoAllowBashIfSandboxed: true`. `filesystem.allowWrite` includes npm's cache paths; `network.allowedDomains` includes `registry.npmjs.org` (default). Work dir: `/tmp/sandbox-test`.

## Outcome

Four events in the hook log; no `PermissionRequest`. The package downloads, installs, and runs entirely inside the sandbox profile, and every hop is the OS saying yes because the profile was already open for it.

## Files

- `config.yml`: settings under test
- `prompt.md`: prompt as given to Claude
- `results.md`: outcome table (4 Bash calls, all silent)
- `raw/hook-events.jsonl`: `PreToolUse`, `PostToolUse` events
