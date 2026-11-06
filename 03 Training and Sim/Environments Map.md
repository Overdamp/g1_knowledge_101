---
tags: [setup, env]
aliases: [env, conda, uv, venv]
---
# Environments Map

| งาน | env | ชนิด | activate |
|---|---|---|---|
| **unitree_rl_mjlab** (เทรน, sim2sim) | `unitree_rl_mjlab` | conda | `conda activate unitree_rl_mjlab` |
| สคริปต์ต่อหุ่น/sim (monitor, vel_eval, pd_check, sysid) | `g1_sdk` | conda | `conda activate g1_sdk` |
| unitree_rl_lab + Isaac (23dof task) | `~/nvidia/IsaacLab-unitree/env_isaaclab` | uv venv | `source .../bin/activate` |
| Isaac Lab v3.0.0 มาตรฐาน | `~/nvidia/IsaacLab/.venv` | uv | `uv run ...` |
| MicroDuck | `~/nvidia/IsaacLab-main/.venv` (ว่าง) | uv | – |

## รุ่นสำคัญ (2/11/2026)
| env | Isaac Sim | newton / warp | mujoco | torch |
|---|---|---|---|---|
| `IsaacLab/.venv` | 6.2.0 | 1.6.1 / 1.17 | 3.12 | 2.11 cu130 |
| `env_isaaclab` | 6.0.0 | **1.2.0** / 1.13 | **3.8.1** | 2.10 cu128 |
| conda `unitree_rl_mjlab` | – | – / 1.12 | 3.5 (mjlab 1.2.0) | 2.14 |

## กฎ
1. **`conda deactivate` ก่อน source uv venv** (PATH / LD_LIBRARY_PATH ชนกัน)
2. `source .../activate` = เอา `bin/` ของ env ไว้หน้า PATH ให้ `python` เป็นตัวของ env (ไม่ source ก็ได้ ถ้าเรียก path เต็ม)
3. ใน env_isaaclab ติดตั้งด้วย `uv pip install --no-deps` **ห้าม `uv sync`** (ลบ isaacsim)
4. ใน `~/nvidia/IsaacLab` ต้อง `uv sync --extra isaacsim` เสมอ
5. รัน Isaac: `export OMNI_KIT_ACCEPT_EULA=YES`
6. สคริปต์ Unitree SDK: `env -u CYCLONEDDS_URI` ([[DDS and Domains]])
7. ดิสก์ /home เหลือน้อย (~50 GB) uv ใช้ cache ร่วม conda ดาวน์โหลดซ้ำ

## ทางลัดใน `~/.bashrc` (5/11/2026)
| พิมพ์ | ทำอะไร |
|---|---|
| `g1net` | IP 192.168.123.123 บน `enp43s0` + multicast บน `lo` + ping หุ่น |
| `g1env` | repo + `conda activate g1_sdk` + `unset CYCLONEDDS_URI` + ตั้ง `$S` `$D` |
| `g1sim` | unitree_mujoco (lo, domain 1) · `G1_SIM_DOMAIN=0 g1sim` สำหรับ g1pilot |
| `g1sim_band` | sim ที่มีเชือก (`~/sim_band`, domain 1) · ปุ่ม 9 / 7 / 8 |
| `g1ctrl_sim` | g1_ctrl กับ sim · ไม่ยอมเปิดซ้ำ |
| `g1ctrl_real` | g1_ctrl หุ่นจริง · ป้ายแดง · ต้องพิมพ์ `real` · ไม่เปิดถ้า sim เปิดอยู่หรือยังไม่ตั้ง IP |
| `ros_lo` | ROS 2 บน lo + `ROS_DOMAIN_ID=1` เฉพาะ terminal นั้น |

- `CYCLONEDDS_URI` / `ROS_DOMAIN_ID` ไม่ export ทั่วไปแล้ว (comment ไว้) · `RMW_IMPLEMENTATION=rmw_cyclonedds_cpp` ยังเป็นค่าทั่วไป
- history: `HISTSIZE=100000`, `HISTFILESIZE=200000`, มีวันเวลา, `history -a` ทุกคำสั่ง (ช่อง byobu ไม่ปนกัน) · ค้นด้วย Ctrl + R
- สำรอง: `~/.bashrc.bak_20261105`, `~/.bashrc.bak_20261105b`

## ดูเพิ่ม
[[Cross-Sim Isaac Lab]] · [[Training Practice]] · [[Troubleshooting]]
