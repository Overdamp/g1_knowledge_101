---
tags: [moc]
aliases: [หน้าแรก, Home]
updated: 2026-11-04
---
# G1 23-DoF: แผนที่ความรู้

โน้ตชุดนี้อธิบายงาน **velocity policy ของ Unitree G1 23-DoF** ตั้งแต่การเทรนใน sim จนถึง deploy บนหุ่นจริง เขียนให้กลับมาอ่านแล้วเข้าใจว่า "ทำไม" ไม่ใช่แค่ "ทำอะไร"
ทุกโน้ตบอกที่มา (ไฟล์ใน repo) ถ้าโน้ตกับโค้ดขัดกัน **ให้เชื่อโค้ด** แล้วแก้โน้ต

> [!tip] อ่านครั้งแรก เริ่มตามลำดับนี้
> 1. [[Deploy Contract]]: policy คุยกับหุ่นผ่านอะไร
> 2. [[PD Control]] → [[Armature and Gains]]: kp/kd คืออะไร มาจากไหน
> 3. [[g1_ctrl FSM]] → [[Motion Switcher]]: ใครคุมหุ่นตอนไหน
> 4. [[Test Day Procedure]]: วันทดสอบหุ่นจริงทำอะไรบ้าง
> 5. [[Policy Lineage]]: policy แต่ละตัวต่างกันอย่างไร

> [!info] ศึกษาโค้ด
> [[00 Code Map]]: ไฟล์ทุกตัวที่เราสร้าง/แก้ใน 4 repo พร้อมลำดับที่ควรอ่าน
> [[Study Plan]]: แผนฝึก 6 สัปดาห์ (เขียนเองใน `~/g1_study`)

## 01 แนวคิดพื้นฐาน
| โน้ต | คำถามที่ตอบ |
|---|---|
| [[PD Control]] | มอเตอร์คำนวณแรงบิดอย่างไร kp/kd อยู่ตรงไหน |
| [[Armature and Gains]] | ทำไม kp เข่า = 99.1 ไม่ใช่ 150 |
| [[Action Scale]] | output ของ policy กลายเป็นมุมข้อต่ออย่างไร |
| [[Deploy Contract]] | `policy.onnx` + `deploy.yaml` มีอะไร |
| [[Observations]] | policy เห็นอะไรบ้าง (blind) |
| [[Timing and Rates]] | 50 Hz / 200 Hz / 500 Hz / 1 kHz คืออะไร |
| [[Joint Map and Slots]] | ทำไม SDK มี 29 ช่องแต่หุ่นมี 23 ข้อต่อ |
| [[DDS and Domains]] | Unitree SDK กับ ROS 2 คุยกันอย่างไร |
| [[LowState and LowCmd]] | topic หลักสองตัว มีฟิลด์อะไร |
| [[g1_ctrl FSM]] | Passive / FixStand / Velocity / Handover |
| [[Motion Switcher]] | โหมด `ai` กับ debug ต่างกันอย่างไร |
| [[Observation History]] | Hist5 คืออะไร ทำไมโหลด P3b ไม่ได้ |
| [[Soft Start]] | ทำไมต้องค่อยๆ เข้า Velocity |

## 02 หุ่นจริงและความปลอดภัย
| โน้ต | ใช้เมื่อ |
|---|---|
| [[Test Day Procedure]] | ก่อนลงมือทุกครั้ง |
| [[Network Setup]] | ต่อสาย LAN แล้วคุยกับหุ่นไม่ได้ |
| [[Remote and LED]] | ลืมปุ่ม / ไม่แน่ใจสีไฟ |
| [[Byobu Guide]] | ใช้หน้าจอ T1–T4, ปุ่มลัด byobu, copy ข้อความ |
| [[Stop Mechanisms]] | อยากรู้ว่าหยุดหุ่นได้กี่ทาง เร็วแค่ไหน |
| [[Safe Shutdown]] | จะปิดเครื่อง |
| [[Real Robot Tools]] | สคริปต์ตัวไหนทำอะไร |
| [[kp kd Verification]] | ตรวจว่ามอเตอร์ใช้ gain ตามที่ส่ง |
| [[Sysid Motion]] | สั่งท่าทีละข้อต่อเพื่อเก็บข้อมูล |
| [[FixStand Train Gains]] | วัด gain ชุดที่เทรนตอนแขวน |
| [[Handover]] | คืนการควบคุมให้รีโมท Unitree |
| [[Offset Check]] | สงสัยว่าค่าศูนย์ข้อต่อเปลี่ยน |
| [[Incident Motors 25-26]] | มอเตอร์ศอก/ข้อมือขวาหลุด |
| [[Incident Fall 2026-10-30]] | บทเรียนจากหุ่นล้ม |
| [[Real Robot Results]] | ค่าที่วัดได้จริง ใช้เทียบครั้งหน้า |

## 03 เทรนและ sim
| โน้ต | เรื่อง |
|---|---|
| [[Policy Lineage]] | P1 → P3b → Hist5 → yaw_ft → yaw_stand |
| [[Domain Randomization]] | สุ่มอะไร ทำไม แต่ละ phase |
| [[Training Practice]] | curriculum, resume, DGX |
| [[Sim2Sim Evaluation]] | วัด policy ด้วย `vel_eval.py` และ `unitree_mujoco` |
| [[Cross-Sim Isaac Lab]] | ทดสอบ policy เดียวกันใน PhysX / Newton |
| [[Environments Map]] | env ไหนใช้กับงานไหน |
| [[Troubleshooting]] | อาการแปลกๆ ที่เคยเจอและวิธีแก้ |

## อื่นๆ
- [[Glossary]]: ศัพท์ทั้งหมด
- [[g1_sim2sim Vault]]: เอกสารแผนเดิมใน vault `g1_sim2sim` (ลิงก์สองทาง)
- repo หลัก: `~/nvidia/unitree/unitree_rl_mjlab` (branch `phase3-delay-dr`)
- เอกสารต้นฉบับละเอียดกว่า: `unitree_rl_mjlab/doc/g1_23dof_summary/`, `doc/g1_23dof_real_robot_setup_checklist.md`
