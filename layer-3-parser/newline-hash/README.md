# newline-hash: the parser's third verdict

**Layer:** 3, Bash parser (newline-hash classification)
**Claim:** `newline-hash` fires when an argument contains a newline followed by optional whitespace and a `#`. The warning text is: "Newline followed by # inside a quoted argument can hide arguments from path validation."
**Finding:** A comment position probe across Python and Node `-c`/`-e` inline scripts. 18 of 19 probes prompted. The one that stayed silent (B1) had `#` only as trailing inline comments after code, never as the first non-whitespace character on any line. Indentation doesn't save you; the trigger is positional.
**Claude Code:** 2.1.116
**Harness source:** `sandbox-12c-comment-position-probe`

## Setup

Sandbox on, `autoAllowBashIfSandboxed: false` (so we see the raw classification, not the sandbox happy path). The prompts exercise `python3 -c '...'`, `python3 -c "..."`, `node -e '...'`, `node -e "..."` with varying positions of `#` and `//`.

This receipt is a probe format: individual prompt attempts rather than a single session. The raw log is `raw/prompt-events.log` (analog to `hook-events.jsonl` but captured from the probe runner).

## Outcome

See `results.md` for the full A1–N4 table and the analysis section. One-line: the parser's newline-hash detector keys on "newline → (whitespace)* → `#`". Trailing inline comments survive because the `#` is after non-whitespace on the line.

Examples that prompt:
- `python3 -c '# comment at start\nx = 42\nprint(x)'` (A1)
- `node -e 'const channel = "#general"'` (J1, even a string literal starting with `#` trips it when it follows a newline)

Example that stays silent:
- `python3 -c 'x = 42  # set value\ny = x * 2  # double it\nprint(y)  # output'` (B1)

## Files

- `config.yml`: settings under test
- `prompt.md`: prompt as given to the probe runner
- `results.md`: full outcome table plus analysis
- `raw/prompt-events.log`: per-probe events from the probe runner
