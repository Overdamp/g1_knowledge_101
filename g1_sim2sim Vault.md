---
tags: [moc, external]
aliases: [g1_sim2sim, vault เก่า]
---
# g1_sim2sim Vault

vault อีกตัว (`~/nvidia/unitree/g1_sim2sim`) เป็น repo ช่วง**วางแผน** sim2sim (19–22/10/2026) ก่อนงานย้ายมาทำจริงใน unitree_rl_mjlab
ลิงก์ข้างล่างเป็นแบบ `obsidian://` กดแล้ว Obsidian สลับไปเปิดใน vault `g1_sim2sim` (vault ต้องเปิดใน Obsidian ไว้แล้วอย่างน้อยครั้งหนึ่ง และชื่อ vault ต้องยังเป็น `g1_sim2sim`)

> [!note] อ่านอย่างไร
> เอกสารใน g1_sim2sim เป็น**แผนและการศึกษา** บางข้อเสนอทำไปแล้ว บางข้อเปลี่ยนแนวทาง สถานะปัจจุบันให้ดูโน้ตใน vault นี้ (คอลัมน์ขวา)

| เอกสาร (g1_sim2sim) | เนื้อหา | โน้ตที่เกี่ยวข้องใน vault นี้ |
|---|---|---|
| [README](obsidian://open?vault=g1_sim2sim&file=README) | ภาพรวม repo (สถานะ: scaffold) + ลำดับงาน I01–I05 | [[00 Home]] |
| [implementation_plan](obsidian://open?vault=g1_sim2sim&file=docs%2Fimplementation_plan) | แผน Train → Sim2Sim ผ่าน Unitree SDK → Sim2Real แบ่ง phase, robot/policy contract | [[Deploy Contract]] |
| [environment_components](obsidian://open?vault=g1_sim2sim&file=docs%2Fenvironment_components) | ตาราง Scene → Commands → Actions → Observations → Rewards → Terminations → Events → Curriculum ของ env ฝั่งเทรน | [[Observations]] |
| [real_robot_components_unitree](obsidian://open?vault=g1_sim2sim&file=docs%2Freal_robot_components_unitree) | ตารางเดียวกันแต่ฝั่งหุ่นจริง: SDK messages, motor mapping, ขั้นทดสอบ hardware | [[LowState and LowCmd]] |
| [real_robot_safety_improvements](obsidian://open?vault=g1_sim2sim&file=docs%2Freal_robot_safety_improvements) | ข้อเสนอความปลอดภัยจากคลิป sentdex: bad_orientation, arm smoothing, กระตุกตอนเข้า Velocity, watchdog | [[Stop Mechanisms]] |
| [phase2_domain_randomization_plan](obsidian://open?vault=g1_sim2sim&file=docs%2Fphase2_domain_randomization_plan) | แผนเพิ่ม DR ขั้นที่ 2 (มวล/ข้อต่อ/มอเตอร์) และวิธีทดสอบเปรียบเทียบ | [[Domain Randomization]] |
| [run_empty_simulation](obsidian://open?vault=g1_sim2sim&file=docs%2Frun_empty_simulation) | เปิด G1 เปล่าใน unitree_mujoco, อ่าน LowState, ส่งคำสั่งผ่าน ROS 2, แก้ปัญหา DDS | [[DDS and Domains]] |
| [experiment_and_presentation_guide](obsidian://open?vault=g1_sim2sim&file=docs%2Fexperiment_and_presentation_guide) | ออกแบบการทดลอง 4 ชุด, KPI, โครงพรีเซนต์ 7 สไลด์ | [[Sim2Sim Evaluation]] |
| [setup](obsidian://open?vault=g1_sim2sim&file=docs%2Fsetup) | ติดตั้งและตรวจ baseline | [[Environments Map]] |
| [sim2sim](obsidian://open?vault=g1_sim2sim&file=docs%2Fsim2sim) | หัวข้อ sim2sim (สั้น) | [[Sim2Sim Evaluation]] |
| [training](obsidian://open?vault=g1_sim2sim&file=docs%2Ftraining) | หัวข้อ training (สั้น) | [[Training Practice]] |
| [unitree_sim2sim_plan](obsidian://open?vault=g1_sim2sim&file=docs%2Freference%2Funitree_sim2sim_plan) | แผน sim2sim ผ่าน Unitree API ทีละ phase (parity test, stress test, safety layer) | [[Sim2Sim Evaluation]] |
| [isaaclab_g1_locomotion](obsidian://open?vault=g1_sim2sim&file=docs%2Freference%2Fisaaclab_g1_locomotion) | คู่มือเทรน G1 ยืน/เดินด้วย Isaac Lab + Newton: MDP, PPO, ไฟล์ที่ต้องแก้ | [[Cross-Sim Isaac Lab]] |
| [training_from_scratch_mjlab](obsidian://open?vault=g1_sim2sim&file=docs%2Freference%2Ftraining_from_scratch_mjlab) | สร้างโปรเจกต์เทรน G1 ยืน/เดินเองบน mjlab ทีละขั้น (อังกฤษ) | [[Training Practice]] |
| [unitree_training_plan](obsidian://open?vault=g1_sim2sim&file=docs%2Freference%2Funitree_training_plan) | แผนโครงโปรเจกต์ทดลองฝึก G1 | [[Training Practice]] |

## ข้อเสนอในเอกสารเก่า → สถานะตอนนี้
| ข้อเสนอ ([[Stop Mechanisms]] / real_robot_safety_improvements) | สถานะ (3/11/2026) |
|---|---|
| เปิด `bad_orientation` ตัดเป็น Passive เมื่อเอียงเกิน 0.85 rad | ✅ ทำงานใน Velocity ([[Stop Mechanisms]]) |
| กันกระตุกตอนเข้า Velocity | ✅ ทำด้วย [[Soft Start]] 0.5 s ใน `State_RLBase.cpp` |
| Arm EMA filter | มีในโค้ดแต่ปิด (`arm_ema_alpha` 1.0) เพราะเปลี่ยน dynamics จากตอนเทรน |
| watchdog policy thread / DDS lag | LowState timeout 100 ms → Passive ([[Timing and Rates]]) |
| แผน DR ขั้นที่ 2 | ✅ เป็น Phase 2 → 3b ([[Domain Randomization]]) |

## โน้ตกลับฝั่งนั้น
ใน g1_sim2sim มีโน้ต [docs/knowledge_vault_g1](obsidian://open?vault=g1_sim2sim&file=docs%2Fknowledge_vault_g1) ลิงก์กลับมาที่ [[00 Home]]
