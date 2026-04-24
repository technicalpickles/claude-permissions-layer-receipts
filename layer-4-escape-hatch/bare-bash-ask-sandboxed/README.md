# Bare Bash in ask list: sandboxed commands stay silent

**Layer:** 4, Escape hatch (workaround setup)
**Claim:** Adding `"Bash"` (bare, no specifier) to `permissions.ask` doesn't disrupt the auto-allow flow for sandboxed commands. The rule doesn't fire when the Bash call is sandboxed.
**Finding:** Three sandboxed Bash commands, all silent. No `PermissionRequest` events, confirming the bare `Bash` ask rule is transparent on the auto-allow path.
**Claude Code:** 2.1.117
**Harness source:** `permissions-02a-bare-bash-ask-sandboxed`

## Setup

Sandbox on, `autoAllowBashIfSandboxed: true`, `permissions.ask: ["Bash"]`. Paired with `bare-bash-ask-unsandboxed/` (same settings; that one runs commands that force `dangerouslyDisableSandbox` and shows the rule fires).

This is the [@GMNGeoffrey workaround](https://github.com/anthropics/claude-code/issues/34315): `Bash` in `ask` doesn't fire for sandboxed commands, fires for unsandboxed ones. Gives you a prompt exactly when Claude escalates.

## Outcome

| # | Command | Permission |
|---|---------|------------|
| A3 | `ls /tmp/sandbox-test` | silent |
| 2 | `touch /tmp/sandbox-test/ask-baseline-test.txt && echo done` | silent |
| 3 | `cat /tmp/sandbox-test/README.md 2>&1 \| head -3` | silent |

Zero prompts. The `Bash` in `ask` rule is inert while commands stay sandboxed.

## Files

- `config.yml`: settings under test
- `prompt.md`: prompt as given to Claude
- `results.md`: outcome table
- `raw/hook-events.jsonl`: `PreToolUse`, `PostToolUse` events (no `PermissionRequest`)
