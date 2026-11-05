---
tags: [concept, deploy]
aliases: [deploy.yaml, policy.onnx, contract]
---
# Deploy Contract

"สัญญา" ระหว่างฝั่งเทรน (Python, mjlab) กับฝั่ง deploy (C++, [[g1_ctrl FSM|g1_ctrl]]) มี 2 ไฟล์ต่อ policy:

```
config/policy/velocity/<ชื่อ>/
├── exported/policy.onnx      ← เครือข่าย (normalizer + MLP) + metadata (joint_names)
└── params/deploy.yaml        ← ทุกอย่างที่ deploy ต้องทำให้ตรงกับตอนเทรน
```

## deploy.yaml มีอะไร
| ฟิลด์ | ความหมาย | ผิดแล้วเกิดอะไร |
|---|---|---|
| `joint_ids_map` | ลำดับข้อต่อของ policy → slot SDK ([[Joint Map and Slots]]) | ส่งคำสั่งผิดมอเตอร์ |
| `stiffness`, `damping` | kp/kd ([[Armature and Gains]]) | ข้อต่อแข็ง/อ่อนผิด policy พัง |
| `default_joint_pos` | ท่ายืนพื้นฐาน | ท่าเพี้ยน |
| `actions.JointPositionAction.scale` | [[Action Scale]] | ก้าวใหญ่/เล็กผิด |
| `observations` | term, scale, `history_length` ([[Observations]]) | policy เห็นข้อมูลผิดรูป |
| `step_dt` | 0.02 s (50 Hz) ([[Timing and Rates]]) | จังหวะผิด |

## ตรวจอย่างไร
```bash
python deploy/robots/g1_23dof/scripts/gain_check.py deploy/robots/g1_23dof/config/policy/velocity/v0
```
ต้องขึ้น `PASS` (ตรวจ joint map, kp, kd, default pose, action scale, offset และขนาด obs เทียบ metadata ใน ONNX)

## policy ที่มีอยู่
| โฟลเดอร์ | policy | ใช้ |
|---|---|---|
| `v0` | **P3b** (md5 `727da8a2…`) | ตัว deploy หลัก |
| `hist5_yawft` | yaw_ft | ทดลอง |
| `hist5_yawstand` | yaw_stand | ตัวที่สอง (เข้า Velocity เฉพาะเท้าแตะพื้น) |

ใน `config.yaml` ต้อง **pin `policy_dir`** ไว้ เพราะถ้าไม่ระบุ `exported/` ตรงๆ g1_ctrl จะเลือก "โฟลเดอร์ย่อยตัวสุดท้ายตามชื่อ" (เพิ่มโฟลเดอร์ใหม่อาจแทน policy โดยไม่รู้ตัว)

## ใช้ข้าม repo
contract นี้ทำให้เอา policy จาก mjlab ไปเล่นใน Isaac Lab ได้ ([[Cross-Sim Isaac Lab]]) ข้อต่าง: unitree_rl_lab export stiffness เรียงตาม SDK (27 ช่อง) ส่วน mjlab เรียงตาม policy (23 ช่อง)

## ดูเพิ่ม
[[Policy Lineage]] · [[Sim2Sim Evaluation]]
- ใน vault g1_sim2sim: [implementation_plan](obsidian://open?vault=g1_sim2sim&file=docs%2Fimplementation_plan) (robot/policy contract ในแผนเดิม)
