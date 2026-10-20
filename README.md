# review-panel

An agent skill for adversarial review by a panel of other models. The session driving your work (the driver) writes a question, sends the same brief to every panel member in parallel, each in a fresh read-only session, then judges their findings: it adopts what it agrees with, rejects what it does not, and needs no consensus. You see its conclusions beside each member's position, and which members ran the driver's own model or vendor.

Three modes:

- **converge**: for something the driver wrote and may revise (its own wording, its own code). Adopted critique is applied and the revision goes back to the panel, up to three rounds by default.
- **opinion**: one round, for anything the driver does not own, such as someone else's pull request.
- **research**: one round on a problem rather than an artifact; each member researches it with web access and proposes a design, citing a source for every claim about external behaviour. The driver verifies the load-bearing claims and synthesises.

Each member gets the least context its review needs: `sealed` (the brief alone), `briefed` (plus chosen files) or `checkout` (a read-only copy of the repository), with `web` added where the material makes claims about the outside world. `runtimes.md` says how each runtime is run and what it reports.

## Install

Copy the `review-panel/` directory into a skills directory:

- user-wide: `~/.claude/skills/` (Claude Code), `~/.agents/skills/` (other clients)
- per repository: `<repo>/.claude/skills/`, `<repo>/.agents/skills/`

Claude Code resolves a skill present at both levels in favour of the user-level copy; most other clients prefer the project copy. The roster line printed before each review names the version that ran.

## Configure your panel

Copy `review-panel/panel.example.toml` to `~/.config/review-panel/panel.toml` and list the members this machine can run: a runtime, a model (unversioned names such as `opus` or a family such as `sol` pick up new releases; a full id pins one), and optionally an effort and a timeout. During a session, amend the panel in plain words. With no file, the driver uses one fresh session of a model other than its own.

## Why the driver is the judge

The panel exists to generate adversarial pressure, not agreement. Reviewers always find something, and requiring consensus would let the most contrarian member set the outcome. The driver weighs each finding, and you see both its reasoning and the members' raw output, so you can overrule either.
