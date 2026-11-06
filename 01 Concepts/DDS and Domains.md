---
tags: [concept, network]
aliases: [DDS, domain, CycloneDDS, multicast]
---
# DDS and Domains

## ภาพรวม
Unitree SDK และ ROS 2 ใช้ **DDS (CycloneDDS)** ส่งข้อความแบบ publish/subscribe เหมือนกัน แยกวงด้วย **domain id**

```
หุ่น G1 (192.168.123.161) ──DDS domain 0──► PC enp43s0 (192.168.123.123)
                                              ├─ g1_ctrl / สคริปต์ --network enp43s0 (Unitree SDK, domain 0)
                                              └─ g1pilot robot_state ──► ROS 2 domain 1 ──► RViz
unitree_mujoco (sim) ─────DDS domain 1──► lo  ── g1_ctrl / สคริปต์ ค่าเริ่มต้น (ไม่ใส่ flag)
```

| ระบบ | การ์ด | domain | ทำไม |
|---|---|---|---|
| หุ่นจริง (g1_ctrl, สคริปต์ Python) | `enp43s0` | **0** | ค่าตายตัวของหุ่น ต้องระบุเองทุกครั้ง |
| sim ของเรา (`unitree_rl_mjlab/simulate`, `~/g1_study/sim`) | `lo` | **1** | ค่าเริ่มต้นของทุกโปรแกรม (5/11/2026) |
| ROS 2 (g1pilot, RViz) | `lo` (`ros_lo`) | **1** | ถ้าใช้ 0 ใน process เดียวกับ SDK ของ g1pilot จะชนกัน |
| g1pilot sim | `lo` | sim **0** | SDK ใน g1pilot เขียน domain 0 ตายตัว → `G1_SIM_DOMAIN=0 g1sim` |

### sim เป็นค่าเริ่มต้น หุ่นจริงต้องระบุ (5/11/2026, commit 4fc50cc, 4318176)
- **กั้นสองชั้น:** การ์ด (`lo` / `enp43s0`) + domain (1 / 0) ลืม flag ตัวเดียวก็ไม่ข้ามไปอีกฝั่ง ทดสอบแล้ว: sim เปิดอยู่ที่ domain 1 → ฟังที่ domain 0 ได้ 0 ข้อความ
- **flag ชื่อเดียวกัน:** `--network` ใช้ได้ทั้ง g1_ctrl และสคริปต์ python (6/11) · `--iface` = ชื่อเก่าของสคริปต์ python ยังใช้ได้
- **ไม่ใส่ flag = sim:** `g1_ctrl` (ไม่มี `--network` → `lo`, domain 1) และสคริปต์ Python ทุกตัว (`--network` ค่าเริ่มต้น `lo`) · หุ่นจริง = `--network enp43s0` (domain 0 ให้เอง) · `deploy_byobu.sh` **ต้องใส่** `IFACE=`
- **บรรทัดแรกบอกเป้าหมาย:** `DDS: SIM | network lo | domain 1` หรือ `DDS: REAL ROBOT | network enp43s0 | domain 0` (g1_ctrl: `DDS: domain 0 on enp43s0 (REAL ROBOT, keyboard off, remote only)`)
- **ผิด domain = รอเฉยๆ:** g1_ctrl ขึ้น warning แล้วค้าง `Waiting for connection` ไม่ส่งอะไร
- **ช่องโหว่ที่เหลือ:** ถ้า sim เปิดค้างในวันเทสแล้วลืม `--network enp43s0` จะได้ค่าจาก sim (PASS หลอก) → `deploy_byobu` (โหมดหุ่น) และ `g1ctrl_real` ไม่ยอมเปิดถ้า sim เปิดอยู่ + checklist ข้อ 0a
- ก่อนหน้านี้ g1_ctrl เขียน `Init(0, ...)` ตายตัว sim ต้องเป็น domain 0 และค่าเริ่มต้นสคริปต์คือ `enp43s0` (ลืม flag = ไปหุ่น)

### `CYCLONEDDS_URI` (ตัวแปรของ CycloneDDS)
บอก**ทุกโปรแกรม DDS** ในเครื่องว่าใช้การ์ดไหน ถ้าตั้งเป็น `lo` ไว้ทั่วไป Unitree SDK ที่สั่งให้ใช้ `enp43s0` จะได้ค่าขัดกัน → หาหุ่นไม่เจอ/ล้ม · ตั้งแต่ 5/11 `~/.bashrc` ไม่ export ทั่วไปแล้ว ใช้ `ros_lo` เฉพาะ terminal ROS · terminal หุ่นใช้ `g1env` (unset ให้) · ดู [[Environments Map]]

`robot_state` ของ g1pilot อ่าน `rt/lowstate` จาก domain 0 แล้วส่งต่อเป็น ROS 2 domain 1 (~1050 Hz) **ไม่ต้องตั้งเลขเดียวกัน**

## Gotcha ที่เจอบ่อย
| อาการ | สาเหตุ | แก้ |
|---|---|---|
| sim/g1_ctrl ไม่เจอกัน ไม่มี error | `lo` ไม่มี MULTICAST (หายทุกครั้งที่รีบูตคอมพ์) | `sudo ip link set lo multicast on` **ก่อน** เปิด sim |
| crash `dds_write.c ... iox_pub` | shell ที่ source ROS โหลด `libddsc` ของ ROS | `env -u CYCLONEDDS_URI LD_LIBRARY_PATH=/opt/unitree_robotics/lib ./...` |
| สคริปต์ Python error `does not match an available interface` | สาย LAN ไม่ได้เสียบ / หุ่นปิด | เช็ค `ip -br addr show enp43s0` |
| ROS ไม่ได้ข้อมูล | ไม่ได้ตั้ง domain/การ์ดของ ROS | `ros_lo` (ROS_DOMAIN_ID=1 + lo) |
| สคริปต์/g1_ctrl ไม่เห็นหุ่น บรรทัดแรกขึ้น `SIM` | ลืม `--network enp43s0` | ใส่ flag แล้วรันใหม่ |
| g1_ctrl บน lo ค้าง `Waiting for connection` + warning domain | sim อยู่ domain 1 แต่สั่ง `--domain 0` | ไม่ต้องใส่ `--domain` (หรือ `G1_SIM_DOMAIN` ให้ตรงกัน) |

## ดูเพิ่ม
[[Network Setup]] · [[LowState and LowCmd]] · [[Troubleshooting]]
- ใน vault g1_sim2sim: [run_empty_simulation](obsidian://open?vault=g1_sim2sim&file=docs%2Frun_empty_simulation) (ทดสอบ DDS กับ sim เปล่า)
