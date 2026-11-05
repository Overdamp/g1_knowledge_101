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

## ดูเพิ่ม
[[Cross-Sim Isaac Lab]] · [[Training Practice]] · [[Troubleshooting]]
