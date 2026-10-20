# Runtimes

`<dir>` is the member's working directory; `<brief-file>` holds the full brief, passed on stdin, never interpolated into shell source. Every command runs under the member's `timeout` in seconds (default 900); macOS has no `timeout`, so `perl -e 'alarm shift; exec @ARGV or exit 127' <seconds> <command>` stands in, exiting 142 on expiry. A member that times out is reported and left out. The model each member ran, its session id and its cost are read from the runtime's own output, as below; a field a runtime does not give is `unreported`.

## claude

```
claude -p --model <model> --setting-sources "" --strict-mcp-config --tools "<tools>" --output-format json [--effort <effort>] < <brief-file>
```

`--setting-sources ""` suppresses the user's and the project's CLAUDE.md and settings; for `checkout`, `--setting-sources "project,local"` loads the repository's. `--strict-mcp-config` suppresses configured MCP servers. `<tools>` is `""` for `sealed`, `Read,Grep,Glob` otherwise, plus `WebSearch,WebFetch` for `web`. The answer is the JSON `result`; the model is the key of `modelUsage` (an alias such as `opus` or `sonnet` resolves to the newest version); `session_id`, `total_cost_usd` and `duration_ms` are the session, the cost and the time. A second delivery to the same session uses `--resume <session_id>`. Automatic memory is per working directory and therefore empty for a fresh directory.

## codex

```
codex exec --skip-git-repo-check --sandbox read-only --ephemeral -c model_reasoning_effort=<effort> [-m <model>] < <brief-file>
```

Without `-m` the runtime's default model runs. The header on stderr gives `model:`, `provider:` and `session id:`; the answer is on stdout, followed by a `tokens used` total. Codex loads `AGENTS.md` from `<dir>` and from `~/.codex/AGENTS.md`, so while the latter exists a Codex member's level carries `user instructions loaded`. Web access is not yet verified for Codex. A second delivery uses `codex exec resume <session id>`; `--ephemeral` is omitted when a session will be resumed.

## cursor-agent

```
cursor-agent -p --trust --mode ask --model <id> --output-format text "$(cat <brief-file>)"
```

`--mode ask` is read-only. Ids embed a version and end in an effort (`cursor-grok-4.6-xhigh`, `gpt-5.6-sol-high`); a family (`grok`, `sol`) with an effort resolves to the highest version among `cursor-agent --list-models` ids that contain the family as a hyphen-separated segment and end in that effort. It loads `AGENTS.md` and Cursor rules from `<dir>` and cannot suppress user rules set in Cursor's own settings, so its level carries `user instructions unconfirmed` unless the operator says those are empty. It reports no session id. For `web`, `--force` is needed and the member is then read-only by instruction only, which the report says.

## subagent

The host's own subagent facility, started without the driver's transcript and read-only by its tool list. A Claude Code subagent loads CLAUDE.md, so it provides `checkout` only, and reports no model id or session id; the alias is recorded as unverified and its cost as `unreported`.
