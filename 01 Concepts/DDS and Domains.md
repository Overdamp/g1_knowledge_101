---
tags: [concept, network]
aliases: [DDS, domain, CycloneDDS, multicast]
---
# DDS and Domains

## ภาพรวม
Unitree SDK และ ROS 2 ใช้ **DDS (CycloneDDS)** ส่งข้อความแบบ publish/subscribe เหมือนกัน แยกวงด้วย **domain id**

```
หุ่น G1 (192.168.123.161) ──DDS domain 0──► PC enp43s0 (192.168.123.123)
                                              ├─ g1_ctrl, monitors, sysid (Unitree SDK, domain 0)
                                              └─ g1pilot robot_state ──► ROS 2 domain 1 ──► RViz
```

| ระบบ | domain | ทำไม |
|---|---|---|
| Unitree SDK (หุ่น, g1_ctrl, สคริปต์ Python) | **0** | ค่าตายตัวของหุ่น |
| ROS 2 (g1pilot, RViz) | **1** | ถ้าใช้ 0 ใน process เดียวกับ SDK จะชนกัน |
| sim แยก (sysid test) | 1 | กันไม่ให้ชนกับ sim อื่น |

`robot_state` ของ g1pilot อ่าน `rt/lowstate` จาก domain 0 แล้วส่งต่อเป็น ROS 2 domain 1 (~1050 Hz) **ไม่ต้องตั้งเลขเดียวกัน**

## Gotcha ที่เจอบ่อย
| อาการ | สาเหตุ | แก้ |
|---|---|---|
| sim/g1_ctrl ไม่เจอกัน ไม่มี error | `lo` ไม่มี MULTICAST (หายทุกครั้งที่รีบูตคอมพ์) | `sudo ip link set lo multicast on` **ก่อน** เปิด sim |
| crash `dds_write.c ... iox_pub` | shell ที่ source ROS โหลด `libddsc` ของ ROS | `env -u CYCLONEDDS_URI LD_LIBRARY_PATH=/opt/unitree_robotics/lib ./...` |
| สคริปต์ Python error `does not match an available interface` | สาย LAN ไม่ได้เสียบ / หุ่นปิด | เช็ค `ip -br addr show enp43s0` |
| ROS ไม่ได้ข้อมูล | `ROS_DOMAIN_ID=0` จาก `.bashrc` | ตั้งเป็น 1 |

## ดูเพิ่ม
[[Network Setup]] · [[LowState and LowCmd]] · [[Troubleshooting]]
- ใน vault g1_sim2sim: [run_empty_simulation](obsidian://open?vault=g1_sim2sim&file=docs%2Frun_empty_simulation) (ทดสอบ DDS กับ sim เปล่า)
