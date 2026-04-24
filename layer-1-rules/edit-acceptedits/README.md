# Edit in acceptEdits mode

**Layer:** 1, Rules (mode interaction)
**Claim:** In `acceptEdits` mode, the same Edit that prompted under `default` goes through silent. The mode auto-accepts file-edit tool calls without changing the rule set.
**Finding:** Edit ran silent. No `PermissionRequest` event in the hook log, despite Edit not being in `permissions.allow`.
**Claude Code:** 2.1.116 or 2.1.117
**Harness source:** `permissions-01b-edit-acceptedits`

## Setup

Same as `edit-default`, plus `defaultMode: acceptEdits`. Sandbox on, `autoAllowBashIfSandboxed: true`, `permissions.allow: [Read, Write]`.

## Outcome

Two-step test, identical shape to `edit-default`:

1. `Write` creates the target file → silent.
2. `Edit` changes the content → silent. Mode auto-accepted, no prompt, no sidecar action needed.

Companion to `edit-default/`: together they show that modes operate on a different axis from rules.

No `results.md` for this receipt. Outcome is reconstructible from `raw/hook-events.jsonl` (the absence of `PermissionRequest` on the Edit call is the finding).

## Files

- `config.yml`: settings under test
- `prompt.md`: prompt as given to Claude
- `raw/hook-events.jsonl`: `PreToolUse`, `PostToolUse` events (no `PermissionRequest`; that's the point)
