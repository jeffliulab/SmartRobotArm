[![Language: English](https://img.shields.io/badge/Language-English-2f81f7?style=flat-square)](README.md) [![语言: 简体中文](https://img.shields.io/badge/语言-简体中文-e67e22?style=flat-square)](README_zh.md)

# Open Chess Robot

[![ROS 2](https://img.shields.io/badge/ROS_2-Jazzy-22314e?style=flat-square)](https://docs.ros.org/en/jazzy/) [![Gazebo](https://img.shields.io/badge/Gazebo-Harmonic-orange?style=flat-square)](https://gazebosim.org/) [![Status: Active](https://img.shields.io/badge/Status-Active-success?style=flat-square)]() [![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

> 🤖 If you are an AI agent, read [AGENTS.md](AGENTS.md) first.

A robot arm that plays chess against you — physically, in a Gazebo simulation, with a real
gripper picking real pieces. Any decision-making "brain" can drive the arm through a small
world protocol ([AWI](https://github.com/jeffliulab/anima-zero) over MCP); the arm side never
imports a brain framework. This repo is the body half of the chess line — the brain half is
[`anima-zero`](https://github.com/jeffliulab/anima-zero).

---

## Overview

Chess is a task that punishes sloppy manipulation: 64 squares, centimeter targets, long
horizons. This repo simulates that task end to end — a 6-DOF servo arm (Episode1) with a
servo gripper in Gazebo Harmonic, two cameras, a referee that knows the rules — and exposes
it as an MCP "world" server. A brain connects over HTTP, perceives the board, and issues
human-level commands ("move e2 to e4"); the world executes them with MoveIt motion planning
and reports back honestly.

The same body will move to the real Episode1 arm; the VLA data-and-training line for that
lives in [`episode-vla-pi`](https://github.com/jeffliulab/episode-vla-pi).

## Key features

- **Full-stack chess world**: ROS 2 Jazzy + MoveIt + Gazebo Harmonic, 32 pieces spawned from
  any FEN, real grasp physics (no link-attacher shortcuts), rule-aware referee.
- **Brain-agnostic protocol**: the world speaks AWI over MCP — observe / move / remove /
  place / status — so any MCP-host framework can play, not just ANIMA.
- **Vision pipeline**: overhead + oblique camera bridges with named streams, board-state
  perception designed for closed-loop recovery.
- **Failure injection**: grip-miss and other fault hooks for testing a brain's recovery
  behavior, not just its happy path.

## Prerequisites

- ROS 2 Jazzy + Gazebo Harmonic on Ubuntu 24.04
- The Episode1 ROS 2 simulation stack (URDF + Gazebo twin), colcon-built, with its path
  exported as `EPISODE_WS`
- Python 3.12

## Installation

```bash
source /opt/ros/jazzy/setup.bash
source "$EPISODE_WS/install/setup.bash"
cd sim/gazebo-chess
python3 -m venv --system-site-packages .venv
.venv/bin/pip install -e .
```

## Running a game

Three stages, in order (each in its own sourced terminal, or scripted):

```bash
ros2 launch episode1_gz_sim sim.launch.py        # 1. Gazebo stack
sim/gazebo-chess/scripts/start_camera_bridge.sh  # 2. camera bridge
cd sim/gazebo-chess && source .venv/bin/activate
uvicorn server:app --port 8106                   # 3. world server
```

Then point any AWI-speaking brain at `http://localhost:8106` — with `anima-zero`, add
`gazebo-chess=http://localhost:8106` to its `ANIMA_WORLDS` and play from its web UI.
Headless mode (`headless:=true rviz:=false`), custom FEN (`GZCHESS_SETUP_FEN`), and side
selection (`GZCHESS_BOT_SIDE`) are supported; see `sim/gazebo-chess/README.md` for the full
knob list.

## License

[MIT](LICENSE) © 2026 Jeff Liu
