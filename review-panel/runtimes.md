# Runtimes

`<dir>` is the member's working directory; `<brief-file>` holds the full brief. Never interpolate brief text into shell source: pass it on stdin or as `"$(cat <brief-file>)"`.

Run every command under the member's `timeout` in seconds, default 900. macOS has no `timeout`; use `perl -e 'alarm shift; exec @ARGV or exit 127' <seconds> <command>`, which exits 142 on timeout and 127 when the command cannot start. Prefix every command below with it. On expiry, report the member as timed out and proceed without it.

Record the exact model from each runtime's own evidence, as below. A `model` that is not a family is an alias or a full id the runtime accepts, passed as given.

A member with `web` reads the web by the runtime's means below, and no member may write outside `<dir>`. Where a runtime's own controls cannot guarantee that, run it under an OS write guard: on macOS, `sandbox-exec -f <profile> <command>` with a profile that allows network and denies file writes under the home directory except `<dir>` and the runtime's own state, for example:

```
(version 1)
(allow default)
(deny file-write* (subpath "<home>"))
(allow file-write* (subpath "<dir>") (subpath "<home>/.cursor") (subpath "<home>/.local/share/cursor-agent") (subpath "<home>/Library/Application Support/Cursor") (subpath "<home>/Library/Caches"))
```

## subagent

The host's own subagent facility, with the model named as the host accepts it (`sonnet`, `opus`).

- Use it only when the host enforces it as read-only and starts it without your transcript, in a separate copy of the repository. Otherwise use a CLI runtime.
- A Claude Code subagent loads CLAUDE.md, so it provides `checkout` only.
- It reports no model id; record the alias as unverified.

## claude

From `<dir>`:

```
claude -p --model <model> --setting-sources "" --strict-mcp-config --tools "<tools>" --output-format json [--effort <effort>] < <brief-file>
```

- `--setting-sources ""` suppresses user and project CLAUDE.md and settings; `--strict-mcp-config` suppresses configured MCP servers.
- For `checkout`, pass `--setting-sources "project,local"` instead, which loads the repository's CLAUDE.md but not the user's.
- `<tools>`: `""` for `sealed`, `Read,Grep,Glob` otherwise; add `WebSearch,WebFetch` for `web`.
- Always pass the brief on stdin. `--tools` takes several values and binds a trailing argument as one of them.
- The answer is the JSON `result`; the exact model is the key of `modelUsage`. Aliases `sonnet`, `opus`, `haiku` resolve to the newest version.
- Do not use `--bare`: it requires API-key authentication.

## cursor-agent

From `<dir>`:

```
cursor-agent -p --trust --mode ask --model <id> --output-format text "$(cat <brief-file>)"
```

- `--mode ask` is read-only.
- For `web`: cursor-agent 2026.09.26 or later, where WebSearch is a dynamic tool (`GetDynamicTools`, then `CallDynamicTool`, namespace `cursor`); add `--force`, without which print mode rejects the call. `--force` approves every tool call, ask mode forbids writes only by instruction, and cursor-agent's own `--sandbox` does not confine writes, so a `web` member always runs under the OS write guard above.
- It loads `AGENTS.md` and Cursor rules from `<dir>`; a `sealed` or `briefed` directory contains neither. It cannot suppress user rules set in Cursor's own settings. Unless the operator has stated that those are empty, report its context as `sealed, user rules unconfirmed` or `briefed, user rules unconfirmed`.
- Ids embed a version and end in an effort, such as `cursor-grok-4.6-xhigh`, `gpt-5.6-sol-high`, `glm-5.2-high`. A family (`grok`, `sol`, `glm`) requires an effort: resolve it to the highest version, comparing the one dot-numeric segment of each id numerically, among `cursor-agent --list-models` ids that contain the family as a hyphen-separated segment and whose last segment exactly equals the effort. A full id in the panel file is used as given; pin one whenever an id has more than one dot-numeric segment. The resolved id is the exact model; never pass `auto`.
- The list can lag behind what is accepted; an unknown id fails and prints the accepted list.
- Output arrives only when the member finishes; a silent member is not necessarily hung.

## codex

From `<dir>`:

```
codex exec --skip-git-repo-check --sandbox read-only --ephemeral -c model_reasoning_effort=<effort> [-m <model>] < <brief-file>
```

- Omit `-m` to run codex's current default. The exact model is the `model:` line of the output header; the answer follows the last `codex` line.
- Always pass `model_reasoning_effort`; the default may be `none`.
- For `web`: not yet verified. Check the installed codex for a web search option and confirm it with a checkable fetch before granting `web`.
- It loads `AGENTS.md` from `<dir>` and `~/.codex/AGENTS.md`. Before dispatch, check for `~/.codex/AGENTS.md`: while it exists, codex provides `checkout` only, and the report states that user instructions loaded.
