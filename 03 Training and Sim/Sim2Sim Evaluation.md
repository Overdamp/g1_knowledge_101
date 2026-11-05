---
tags: [sim, evaluation]
aliases: [sim2sim, vel_eval, unitree_mujoco]
---
# Sim2Sim Evaluation

## สองวิธี
| วิธี | เครื่องมือ | ใช้ |
|---|---|---|
| headless | `vel_eval.py` | เร็ว ทำซ้ำได้ obs/action ตาม deploy.yaml (รวม history) policy 50 Hz, PD 500 Hz ไม่มี DDS delay |
| ครบระบบ | `unitree_mujoco` + g1_ctrl (`sim2sim_policy.sh`) | ทดสอบโค้ด C++ จริง ต้องกดใน GUI |

```bash
D=deploy/robots/g1_23dof/config/policy/velocity      # env g1_sdk
python $S/vel_eval.py --policy $D/v0/exported/policy.onnx --deploy $D/v0/params/deploy.yaml
python $S/vel_eval.py ... --cases "0,0,0" --hang          # แขวนกลางอากาศ
```

## ผล (% ของคำสั่ง, พื้นเรียบ)
| คำสั่ง | P3b | Hist5 | yaw_ft | yaw_stand |
|---|---|---|---|---|
| vx 0.4 | **97** | 82 | 76 | 88 |
| vx 0.8 | 74 | 91 | 83 | **94** |
| vx 1.0 | 41 | 94 | 86 | **95** |
| wz ±0.5 | 69 | 30–41 | **78–79** | 72–79 |
| vy 0.3 | 59 | **72** | 56 | 56 |
| ยืน (joint vel std / yaw std) | 0 / 0 | ≈ 0 | 0.055 / 0.065 | **0 / 0** |
| แขวน (`--hang`) | นิ่ง | – | เตะ 87% limit | **เตะ 131%** |

## unitree_mujoco + g1_ctrl
```bash
$S/sim2sim_policy.sh v0          # tmux: mujoco / g1_ctrl / speed
```
1. MuJoCo: Pause → Load key 0 (`stand_ready`) 2. g1_ctrl: `u` แล้ว `v` 3. MuJoCo: Run 4. `w/s/a/d/q/e/x`

**แขวนแบบรอกใน sim:** คัดลอก `simulate/` แล้วตั้ง `enable_elastic_band: 1` + `robot_scene` เป็น path เต็ม (ใช้ทดสอบ soft start, FixStand gain เทรน, Handover)

## ข้อจำกัดของ sim
- armature ใน `scene_g1_23dof.xml` = 0.01 ทุกข้อต่อ (ตอนเทรน 0.0036–0.025)
- FixStand แขนสั่น ~160 ครั้ง/s เพราะ kd 10 + PD explicit dt 2 ms (หุ่นจริงไม่เป็น)
- `vol` = 0 ทุกมอเตอร์, ไม่มี motion_switcher, bmsstate ปลอม

## ดูเพิ่ม
[[Policy Lineage]] · [[Cross-Sim Isaac Lab]] · [[Troubleshooting]] · ที่มา `doc/g1_23dof_summary/02_sim2sim_eval.md`
- ใน vault g1_sim2sim: [unitree_sim2sim_plan](obsidian://open?vault=g1_sim2sim&file=docs%2Freference%2Funitree_sim2sim_plan) · [run_empty_simulation](obsidian://open?vault=g1_sim2sim&file=docs%2Frun_empty_simulation)
