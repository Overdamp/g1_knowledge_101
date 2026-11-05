---
tags: [concept, deploy]
aliases: [action, target position]
---
# Action Scale

## policy ส่งอะไรออกมา
policy ส่งตัวเลข 23 ตัว (หนึ่งต่อข้อต่อ) ซึ่ง**ไม่ใช่มุมโดยตรง** แปลงเป็นมุมเป้าหมาย:

```
q_des[i] = default_joint_pos[i] + action_scale[i] × action[i]
```

- `default_joint_pos` = ท่ายืนพื้นฐาน (เดียวกับ `qs` ของ FixStand)
- `action_scale = 0.25 × effort_limit / kp` → action = 1 ตอนห่างท่ายืน ≈ แรงบิด 25% ของ limit ถ้าข้อต่อนิ่ง

| กลุ่มมอเตอร์ | scale |
|---|---|
| hip pitch/yaw, waist | 0.548 |
| hip roll, knee | 0.351 |
| ankle, แขน | 0.439 |

## ทำไมสำคัญ
- scale ผูกกับ kp ([[Armature and Gains]]) เปลี่ยน kp แล้วต้องเปลี่ยน scale และเทรนใหม่
- `q_des` **ไม่ถูก clip** (`clip: null` ใน deploy.yaml) ใน sim เป้าเกิน limit ≤ 1% ของ step ข้อเสนอค้าง: ใส่ clip = limit MJCF

## ดูเพิ่ม
[[Deploy Contract]] · [[Observations]] (`last_action` คือ action รอบก่อน)
