[![Language: English](https://img.shields.io/badge/Language-English-2f81f7?style=flat-square)](README.md) [![语言: 简体中文](https://img.shields.io/badge/语言-简体中文-e67e22?style=flat-square)](README_zh.md)

# Open Chess Robot

[![ROS 2](https://img.shields.io/badge/ROS_2-Jazzy-22314e?style=flat-square)](https://docs.ros.org/en/jazzy/) [![Gazebo](https://img.shields.io/badge/Gazebo-Harmonic-orange?style=flat-square)](https://gazebosim.org/) [![Status: Active](https://img.shields.io/badge/Status-Active-success?style=flat-square)]() [![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

> 🤖 如果你是 AI agent，先读 [AGENTS.md](AGENTS.md)。

一只能在 Gazebo 物理仿真里跟你下真棋的机械臂——真实夹爪、真实棋子、真实抓取物理。
任何做决策的"大脑"都可以通过一个小巧的世界协议（AWI over MCP）驱动这条臂，
臂这一侧不 import 任何大脑框架。本仓是下棋线的"身体"半边，大脑半边是
[`anima-zero`](https://github.com/jeffliulab/anima-zero)。

---

## 概览

下棋是一种不放过粗糙操作的任务：64 个格子、厘米级目标、长程流程。本仓把这个任务
端到端仿真出来——一台 6 自由度舵机臂（Episode1）带舵机夹爪，跑在 Gazebo Harmonic
里，两路相机，还有一个懂棋规的裁判——并把它暴露成一个 MCP "world" 服务。大脑通过
HTTP 连上来，看清棋盘，发出人类级指令（"e2 走到 e4"），世界用 MoveIt 运动规划执行
并诚实回报结果。

同一副身体将来会迁到真实的 Episode1 臂上；对应的 VLA 数据与训练线在
[`episode-vla-pi`](https://github.com/jeffliulab/episode-vla-pi)。

## 关键特性

- **全栈下棋世界**：ROS 2 Jazzy + MoveIt + Gazebo Harmonic，任意 FEN 开局 spawn 整盘
  32 子，真实抓取物理（不用 link-attacher 取巧），懂规则的裁判。
- **大脑无关协议**：世界说 AWI over MCP——observe / move / remove / place / status——
  任何能做 MCP host 的框架都能来下，不限于 ANIMA。
- **视觉链路**：俯视 + 斜视两路命名相机流，棋盘感知为闭环纠错设计。
- **故障注入**：夹空等故障钩子，用来考大脑的补救能力，而不只是顺利路径。

## 前提

- Ubuntu 24.04 上的 ROS 2 Jazzy + Gazebo Harmonic
- Episode1 ROS 2 仿真栈（URDF + Gazebo 孪生），已 colcon 构建，路径导出为 `EPISODE_WS`
- Python 3.12

## 安装

```bash
source /opt/ros/jazzy/setup.bash
source "$EPISODE_WS/install/setup.bash"
cd sim/gazebo-chess
python3 -m venv --system-site-packages .venv
.venv/bin/pip install -e .
```

## 跑一局

三个阶段按顺序起（各自一个 source 过的终端，或写脚本串起来）：

```bash
ros2 launch episode1_gz_sim sim.launch.py        # 1. Gazebo 栈
sim/gazebo-chess/scripts/start_camera_bridge.sh  # 2. 相机桥
cd sim/gazebo-chess && source .venv/bin/activate
uvicorn server:app --port 8106                   # 3. 世界服务
```

然后把任何会说 AWI 的大脑指向 `http://localhost:8106`——用 `anima-zero` 的话，把
`gazebo-chess=http://localhost:8106` 加进它的 `ANIMA_WORLDS`，在它的网页界面里开局。
支持无头模式（`headless:=true rviz:=false`）、自定义 FEN（`GZCHESS_SETUP_FEN`）和
先后手选择（`GZCHESS_BOT_SIDE`）；完整开关清单见 `sim/gazebo-chess/README.md`。

## 许可

[MIT](LICENSE) © 2026 Jeff Liu
