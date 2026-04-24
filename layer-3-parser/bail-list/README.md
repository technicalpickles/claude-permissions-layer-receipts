# Parser bail list: what takes a Bash command off the auto-allow path

**Layer:** 3, Bash parser (tree-sitter AST walker)
**Claim:** Inside the `autoAllowBashIfSandboxed` path, the Bash parser walks a tree-sitter AST and returns `simple`, `too-complex`, or `newline-hash`. Only `simple` stays silent. Specific node patterns bail: `simple_expansion` (bare `$VAR`), "Unhandled node type: string" (quoted string whose only child is an expansion), brace expansion, command substitution.
**Finding:** 8 commands, matching bail-list expectations. `echo $HOME` prompts. `echo "$HOME"` prompts (Unhandled string). `echo "item $HOME"` silent (string with literal content adjacent to the expansion, different AST node). `echo {1..5}` prompts. `echo $(whoami)` prompts. Compound + pipeline structurally allowed: `touch ... && echo done` silent, `echo hello | grep hello` silent.
**Claude Code:** 2.1.116 or 2.1.117
**Harness source:** `sandbox-13d-parser-boundary-lite`

## Setup

Sandbox on, `autoAllowBashIfSandboxed: true`. No allow/ask rules for Bash, all gating comes from the parser. Work dir: `/tmp/sandbox-test`.

## Outcome

See `results.md` for the full A1–A8 table. Key rows:

- **A1** `echo $HOME` → prompted (simple_expansion)
- **A2** `echo "$HOME"` → prompted (Unhandled string)
- **A3** `echo "item $HOME"` → silent (string_content adjacent to expansion)
- **A4** `echo {1..5}` → prompted (brace expansion)
- **A5** `echo $(whoami)` → prompted (command substitution)
- **A6** `touch /tmp/sandbox-test/foo && echo done` → silent (compound, all children safe)
- **A7** `echo hello | grep hello` → silent (pipeline, all children safe)

A2 vs A3 is the wild one: same variable, same quoting, one character of literal prefix flips the verdict.

## Files

- `config.yml`: settings under test
- `prompt.md`: prompt as given to Claude (A1–A8 command list)
- `results.md`: outcome table
- `raw/hook-events.jsonl`: `PreToolUse`, `PermissionRequest`, `PostToolUse` events
