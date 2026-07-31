# AGENTS.md — agent entry point for Open Chess Robot

Read this file first, then jump to the section your task needs. It is an index, not a manual.

> **This file is public.** Local working notes — dev logs, in-flight calibration numbers,
> unreleased hardware findings, absolute paths — belong in `CLAUDE.md`, which is not in git.

## Inherited rules

This repository inherits shared rules from `agent-rules`.

- `agent-rules` version: `v0.4.0`
- upstream machine entry: [agent-rules/AGENTS.md](https://github.com/jeffliulab/agent-rules/blob/v0.4.0/AGENTS.md)
- upstream manifest: [reading-order.yaml](https://github.com/jeffliulab/agent-rules/blob/v0.4.0/manifests/reading-order.yaml)

Read this file first for project-local overrides, then the pinned upstream entry, then follow
exactly one matching upstream path — do not sweep the whole upstream repository. Never pin
`main`. Where a rule here conflicts with `agent-rules`, this file wins.

## What this is (and is not)

**A robot arm that plays chess — currently in Gazebo simulation, next on the real Episode1
arm.** The repo's working content is `sim/gazebo-chess/`: a full-stack chess "world"
(ROS 2 + MoveIt + Gazebo, real grasp physics, rule-aware referee) exposed as an MCP server
that any brain can drive over AWI. MIT licensed.

It is **not**:

- a brain — the world executes commands and reports honestly; it does not plan, does not
  judge task success, does not retry (those belong to the brain);
- the VLA data/training line — that lives in `episode-vla-pi` (with the device plugin in
  `lerobot_robot_episode1`);
- tied to any one framework — the world core never imports a brain framework.

## Task → where to look

| If your task is… | Start at |
| --- | --- |
| Work on the Gazebo chess world | `sim/gazebo-chess/` — read the cross-repo rule below first |
| Understand the body/brain split | `docs/architecture.md` |
| Change the AWI world protocol | coordinate with `anima-zero`; see cross-repo rule below |
| Run a game locally | `README.md` → Running a game |
| Project history | `docs/legacy/soma-arm-early/` (archived early attempts) |

## Repo map

| Path | What is in it |
| --- | --- |
| `sim/gazebo-chess/` | The chess world: server, referee, grasp/spawn/vision, camera bridge, web UI |
| `docs/` | `architecture.md`, plus `legacy/soma-arm-early/` — snapshots of where this started |

## Red lines

- ⛔ **Every command that moves the physical arm is run by a human operator.** Never from CI,
  never from a script, never autonomously by an agent. This arm has **no effective hardware
  e-stop** — cutting power is the only real stop, and the joints go limp when power is removed,
  so someone has to be there holding it. Simulation is not covered by this rule; the test is
  whether the command makes real hardware move or draw power.
- ⛔ **The body never decides.** No planning, no success judgement, no retry.
- **The core never imports a brain framework.** Brains attach over the wire protocol only.
- **No hardcoding**: paths derived or from env, tunables in config, magic numbers named. A
  placeholder must be flagged to the maintainer, never buried.

## Cross-repo discipline: `awi_mcp.py`

⛔ `sim/gazebo-chess/awi_mcp.py` is a **byte-identical copy** of the AWI adapter that also lives
in `anima-zero`'s worlds. Over there a test holds every in-repo copy identical; this one is
across a repository boundary and has **no automated guard**. Changing the adapter on either side
means changing it here by hand, in the same commit-shaped unit of work. A silent divergence here
does not fail anyone's CI — it just stops working.

Details: `docs/architecture.md`, `README.md`, and the pinned upstream rules above.
