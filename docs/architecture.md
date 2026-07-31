# Architecture

## A body, with the brain outside the system boundary

Open Chess Robot is the **body** of the chess line: everything between the cameras and the
arm. The brain — whatever framework decides what move to play — sits *outside* this repo and
talks to the body over a wire protocol (AWI over MCP).

Today's working implementation of that body is [`sim/gazebo-chess/`](../sim/gazebo-chess/):
the chess task simulated end to end in Gazebo — Episode1 arm with a servo gripper, 32 pieces
from any FEN, two camera streams, a rule-aware referee — served as an MCP world on `:8106`.
The next implementation is the real Episode1 arm; its data-and-training line lives in
`episode-vla-pi` and its device plugin in `lerobot_robot_episode1`.

## Layering principles

- **The body executes one atomic action per command and reports honestly.** It never plans,
  never judges task success, never retries on its own. Retry, recovery, and verification
  belong to the brain — *any* brain. Execution self-checks (gripper closed? force plausible?)
  are included in the result as early-stop hints, not as verdicts.
- **The body holds physical reality, not task truth.** Perception reports what the camera
  sees; whether that constitutes "the move worked" is the brain's judgement.
- **The body core never imports a brain framework.** A brain attaches over the protocol, so
  the same body can be driven by different frameworks without code changes on either side.

## Why an independent repo

- **Different runtimes.** Real-time robot control (high-frequency, hardware-coupled) and a
  cognitive stack (slow, LLM-driven) want different processes, dependencies, and cadences.
- **One body, many brains.** Welding the body into any single brain's repo would defeat the
  point of the protocol split.
- **Independently showcased.** The chess body is a project in its own right.

The cost of the split — keeping the protocol copies in sync — is paid by hand:
`sim/gazebo-chess/awi_mcp.py` is a byte-identical copy of the adapter in `anima-zero`, with
no automated guard across the repo boundary. Change it on both sides together.

## Status

`sim/gazebo-chess/` is working and in active use with `anima-zero`. Real-arm chess is
upcoming, gated on the `episode-vla-pi` line (teleoperation → data → VLA policy).
