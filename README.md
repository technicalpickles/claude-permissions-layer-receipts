# claude-permissions-layer-receipts

Receipts for a blog post. If the [Permission Layers article on pickles.dev](https://pickles.dev/) (link forthcoming) says "I ran X and got Y," the X and the Y are here.

Each receipt is a small directory containing the Claude Code settings under test (`config.yml`), the prompt given to Claude (`prompt.md`), the outcome table where the harness produced one (`results.md`), and the raw `PreToolUse`/`PermissionRequest`/`PostToolUse` events logged by a hook (`raw/hook-events.jsonl`). A few probe-style receipts have `raw/prompt-events.log` instead; it's the same shape of data captured by a different runner.

The article cites ~10 receipts across six permission layers. Those receipts live here, one per article section, in a layer-oriented directory structure.

## Versions

Tested against **Claude Code 2.1.116 and 2.1.117**, April 2026. Claude Code ships fast. If you're on a newer build and something here contradicts what you observe, trust the build. A PR adding a fresh run for a later version is welcome, see [Contributing](#contributing).

## What's in each receipt

```
<layer>/<receipt>/
├── README.md              # layer, article claim, finding, version
├── config.yml             # sandbox + permissions + hooks config
├── prompt.md              # prompt given to Claude
├── results.md             # outcome table (most receipts)
└── raw/
    └── hook-events.jsonl  # PreToolUse/PermissionRequest/PostToolUse
                           # (or prompt-events.log for probe-format receipts)
```

A few receipts don't have `results.md` because the test was short enough that the outcome is readable directly off the hook events. Those cases are noted in the receipt's own README.

## Mapping: article section to receipt

| Layer | Article claim | Receipt |
|-------|---------------|---------|
| 1, Rules | Edit prompts in `default` mode (not in allow list) | [`layer-1-rules/edit-default`](layer-1-rules/edit-default/) |
| 1, Rules + mode | Same Edit runs silent in `acceptEdits` mode | [`layer-1-rules/edit-acceptedits`](layer-1-rules/edit-acceptedits/) |
| 2, Sandbox | Happy path: `npx cowsay` runs silent with sandbox + `autoAllowBashIfSandboxed` on | [`layer-2-sandbox/happy-path-cowsay`](layer-2-sandbox/happy-path-cowsay/) |
| 2, Sandbox | Write tool bypasses the sandbox; Bash doesn't | [`layer-2-sandbox/write-tool-bypass`](layer-2-sandbox/write-tool-bypass/) |
| 3, Bash parser | Bail-list spot checks (simple_expansion, Unhandled string, braces, command substitution, compound, pipeline) | [`layer-3-parser/bail-list`](layer-3-parser/bail-list/) |
| 3, Bash parser | `newline-hash` classification across Python and Node `-c`/`-e` probes | [`layer-3-parser/newline-hash`](layer-3-parser/newline-hash/) |
| 4, Escape hatch | Natural retry: model sets `dangerouslyDisableSandbox` after a sandbox block | [`layer-4-escape-hatch/natural-retry`](layer-4-escape-hatch/natural-retry/) |
| 4, Escape hatch | Bare `Bash` in `ask`: stays silent while sandboxed (workaround setup) | [`layer-4-escape-hatch/bare-bash-ask-sandboxed`](layer-4-escape-hatch/bare-bash-ask-sandboxed/) |
| 4, Escape hatch | Same rule fires the moment Claude goes unsandboxed (workaround payoff) | [`layer-4-escape-hatch/bare-bash-ask-unsandboxed`](layer-4-escape-hatch/bare-bash-ask-unsandboxed/) |
| 5, Hooks | `PreToolUse` hook with `^curl` blocks one command, silently allows `/usr/bin/curl` | [`layer-5-hooks/curl-block-anchor`](layer-5-hooks/curl-block-anchor/) |
| 6, Auto mode | Classifier carries what the parser bails on (some of the time), punts on bare `$(...)` | [`layer-6-auto-mode/classifier-behavior`](layer-6-auto-mode/classifier-behavior/) |

## About the test harness

The receipts came out of a local test harness that runs isolated Claude Code environments under different sandbox and permission configs and logs every tool call via `PreToolUse` hooks. The harness is built on `CLAUDE_CONFIG_DIR` (see [~/.claude/ Is Production](https://pickles.dev/dot-claude-is-production/) for how that works).

The harness itself isn't open-sourced yet. Publishing just the receipts first because they're what the article actually cites, and because they're reproducible on their own: each `config.yml` is a valid Claude Code settings file, each `prompt.md` is a real prompt you can paste into a fresh session, and the hook log gives you something concrete to compare against. The runner is a future announcement.

## What got sanitized

Absolute paths, usernames, and session UUIDs were rewritten:

- `/Users/josh.nichols/...` → `/Users/[you]/...`
- `josh.nichols` → `[you]`
- Session IDs (the UUIDs Claude Code generates per run) → `[session-id]`

Timing data is preserved. Tool-use IDs (`toolu_...`) are preserved because they're noisy but not identifying. If you spot something that should have been scrubbed and wasn't, open an issue.

## Contributing

Fresh runs on newer Claude Code versions are welcome. A useful PR:

- Reruns one receipt under a specific Claude Code version, keeps the config and prompt the same
- Adds the new `results.md` and `raw/hook-events.jsonl` alongside (or instead of) the existing ones
- Notes in the receipt's README what the version was and whether the outcome changed

If Claude Code's behavior has drifted enough that the original finding no longer holds, that's the interesting case. Say so in the PR, and I'll pull it in.

## License

MIT. See [LICENSE](LICENSE).
