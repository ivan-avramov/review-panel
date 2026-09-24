# review-panel

An agent skill for adversarial review by a panel of independent models. The agent driving your session (the driver) writes a question, sends the same brief to every panel member in parallel, read-only, then adjudicates their findings as super-judge: it adopts what it agrees with, rejects what it doesn't, and needs no consensus. You see the result together with each member's position.

Two modes:

- **converge**: for artifacts the driver wrote and may revise (its own wording, its own code). Adopted critique is applied and the revision goes back to the whole panel, until a round adopts nothing of substance or three rounds have run.
- **opinion**: one adjudicated round, for anything the driver doesn't own, such as someone else's pull request.

The driver chooses how much context each member gets: `sealed` (the brief alone, no repository, no user or project instructions), `briefed` (plus chosen files), or `checkout` (a read-only copy of the repository).

## Install

Copy the `review-panel/` directory into a skills directory:

- user-wide: `~/.claude/skills/` (Claude Code), `~/.agents/skills/` (other clients)
- per repository: `<repo>/.claude/skills/`, `<repo>/.agents/skills/`

Claude Code resolves a skill present at both levels in favour of the user-level copy, silently; most other clients prefer the project copy. The roster line the driver prints before each review names the skill version that ran, so a shadowed copy is visible.

## Configure your panel

Copy `review-panel/panel.example.toml` to `~/.config/review-panel/panel.toml` and list the members this machine can run. Each member names a runtime (`subagent`, `claude`, `cursor-agent`, `codex`), a model, and optionally an effort. Prefer unversioned names (`sonnet`, a family such as `grok`, or the runtime's default) so new model releases are picked up; pin a full id when a runtime's model list lags behind what it accepts.

During a session, amend the panel in plain words ("drop glm for this one", "add sonnet"). With no panel file, the driver uses one fresh invocation of a model other than its own.

## Why the driver is the super-judge

The panel exists to generate adversarial pressure, not to reach agreement. Reviewers always find something, and requiring consensus would let the most contrarian member set the outcome. The driver weighs each finding, and you, the human, see both its conclusion and the members' raw output, so you can overrule it.
