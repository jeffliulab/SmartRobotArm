# AGENTS.md — agent entry point for SOMA Zero

Read this file first, then jump to the section your task needs. It is an index, not a manual:
`README.md` and `docs/architecture.md` are the authority.

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

**The body side of a robot — System 1.** SOMA Zero is about *embodied strategy*: how a physical
robot perceives its workspace and acts in it with a learned Vision-Language-Action policy. It is
**brain-agnostic**: any decision-making framework drives it through one small neutral contract.
MIT licensed. The repository is documentation-first; most folders are still early.

The flagship task is a real arm playing chess on a real board. Chess is the **first task
profile**, chosen because it is long-horizon and unforgiving of sloppy manipulation — it is not
the project's identity.

It is **not**:

- a brain — it does not plan, does not judge whether a task succeeded, and does not retry;
- tied to any one framework — the body core never imports a brain framework;
- a chess project — if you are hardcoding chess into `perception/` or `control/`, you have
  turned the first task profile into the architecture.

## Where a fact belongs (the layering rule)

Two questions decide it. **Would this still be true with a different brain attached?** If not,
it belongs in an adapter, not the core. **Is this physical reality, or task truth?** The body
holds the first; the brain judges the second.

| Layer | What may live here |
| --- | --- |
| `perception/` | What the camera sees, as structured scene state. Never *whether the move worked* |
| `control/` | Executing **one atomic action**, plus execution self-checks (gripper closed? force plausible?) reported as early-stop hints, never as verdicts |
| `interface/` | The neutral contract: observation, action intent, result, progress. Deliberately small and stable — the whole cost of the brain/body split is paid here |
| `adapters/<brain>/` | The only place a specific brain's protocol may appear. One thin translation layer per brain |
| `sim/` | Simulated stand-ins for the physical setup |

## Task → where to look

| If your task is… | Start at |
| --- | --- |
| Read the design before changing anything | `docs/architecture.md` § Layering principles |
| Turn camera images into scene state | `perception/` |
| Execute or self-check one action | `control/` |
| Change what brain and body say to each other | `interface/` — and expect it to ripple |
| Support a new brain | a new folder under `adapters/`; do not touch the core |
| Work on the Gazebo chess world | `sim/gazebo-chess/` — read the cross-repo rule below first |

## Repo map

| Path | What is in it |
| --- | --- |
| `perception/` | Eyes: workspace images to structured scene state |
| `control/` | Hands: VLA policy and arm execution for one atomic action |
| `interface/` | The neutral brain-to-body contract |
| `adapters/anima/` | The ANIMA adapter: SOMA mounts as one of ANIMA's worlds over AWI on MCP |
| `sim/gazebo-chess/` | Gazebo simulation of the arm playing chess, migrated here from `anima-zero` |
| `docs/` | `architecture.md`, plus `legacy/soma-arm-early/` — snapshots of where this started |

## Red lines

- ⛔ **Every command that moves the physical arm is run by a human operator.** Never from CI,
  never from a script, never autonomously by an agent. This arm has **no effective hardware
  e-stop** — cutting power is the only real stop, and the joints go limp when power is removed,
  so someone has to be there holding it. Simulation is not covered by this rule; the test is
  whether the command makes real hardware move or draw power.
- ⛔ **The body never decides.** No planning, no success judgement, no retry. The moment
  `control/` retries on its own, the contract with every brain is broken.
- **The core never imports a brain framework.** Framework-specific code lives in `adapters/`.
- **No hardcoding**: paths derived or from env, tunables in config, magic numbers named. A
  placeholder must be flagged to the maintainer, never buried.

## Cross-repo discipline: `awi_mcp.py`

⛔ `sim/gazebo-chess/awi_mcp.py` is a **byte-identical copy** of the AWI adapter that also lives
in `anima-zero`'s worlds. Over there a test holds every in-repo copy identical; this one is
across a repository boundary and has **no automated guard**. Changing the adapter on either side
means changing it here by hand, in the same commit-shaped unit of work. A silent divergence here
does not fail anyone's CI — it just stops working.

Details: `docs/architecture.md`, `README.md`, and the pinned upstream rules above.
