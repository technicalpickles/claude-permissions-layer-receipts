# Edit in default mode

**Layer:** 1, Rules
**Claim:** In `default` permission mode, the Edit tool prompts when no rule auto-allows it. `allow: [Read, Write]` doesn't cover Edit.
**Finding:** Write (in allow list) ran silent. Edit (not in allow list) fired a `PermissionRequest`, confirming the rules layer treats each tool independently.
**Claude Code:** 2.1.116 or 2.1.117
**Harness source:** `permissions-01-edit-default`

## Setup

Sandbox on, `autoAllowBashIfSandboxed: true`, `permissions.allow: [Read, Write]`, no `defaultMode` set (falls back to `default`). Preamble: `lib/preambles/permissions.md`.

## Outcome

Two-step test:

1. `Write` creates `/tmp/sandbox-test/edit-target.txt` → silent (in allow list).
2. `Edit` changes `before` → `after` in the same file → `PermissionRequest` event fires, sidecar auto-accepts.

No `results.md` for this receipt. Outcome is reconstructible from `raw/hook-events.jsonl` (look for `hook_event_name: "PermissionRequest"` on the Edit call).

## Files

- `config.yml`: settings under test
- `prompt.md`: prompt as given to Claude
- `raw/hook-events.jsonl`: `PreToolUse`, `PermissionRequest`, `PostToolUse` events
