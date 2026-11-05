---
tags: [troubleshooting]
aliases: [ปัญหา, แก้ปัญหา, gotchas]
---
# Troubleshooting

## เน็ตเวิร์ก / DDS
| อาการ | สาเหตุ | แก้ |
|---|---|---|
| sim/g1_ctrl ไม่เจอกัน ค้าง `Waiting for connection` | `lo` ไม่มี MULTICAST (รีบูตคอมพ์) | `sudo ip link set lo multicast on` ก่อนเปิด sim |
| crash `dds_write.c ... iox_pub` | libddsc ของ ROS | `env -u CYCLONEDDS_URI LD_LIBRARY_PATH=/opt/unitree_robotics/lib` |
| `does not match an available interface` | สาย LAN ไม่เสียบ/หุ่นปิด | `ip -br addr show enp43s0` |
| ROS ไม่ได้ข้อมูล | ไม่ได้ตั้ง domain/การ์ดของ ROS | `ros_lo` (5/11: `.bashrc` ไม่ export ทั่วไปแล้ว) |
| เครื่องมือไม่เห็นหุ่น บรรทัดแรก `DDS: SIM` | ลืม `--iface enp43s0` (ค่าเริ่มต้น = sim ตั้งแต่ 5/11) | ใส่ flag ([[DDS and Domains]]) |
| `deploy_byobu.sh`: `Set IFACE` / `unitree_mujoco is running` | ต้องระบุ IFACE / ปิด sim ก่อนโหมดหุ่น | `IFACE=enp43s0 ...` · ปิด sim |
| g1pilot sim ไม่เห็น sim | sim ย้ายไป domain 1 แต่ SDK ใน g1pilot ใช้ 0 | `G1_SIM_DOMAIN=0 g1sim` |

## หุ่นจริง
| อาการ | สาเหตุ | แก้ |
|---|---|---|
| L2 + R2 แล้วจอยสั่นอย่างเดียว / ยังเป็น `ai` | L2 + R2 ไม่ปลด motion_switcher | `motion_mode.py release` ([[Motion Switcher]]) |
| มอเตอร์ `vol 0`, `0x80000000` | ป้องกันค้าง / ฮาร์ดแวร์ | [[Incident Motors 25-26]] |
| lowstate ~350 Hz | เพิ่งบูต | รอแล้วอ่านซ้ำ |
| `kp x` เพี้ยนทุกข้อต่อทางเดียวกัน | `tau_est` คลาด | ไม่ใช่ gain ผิด ([[kp kd Verification]]) |
| byobu Alt + ลูกศรไม่ย้ายช่อง | Alt = สลับ window | คลิก หรือ Shift + ลูกศร |
| pd_check RAM เต็ม | log หลาย GB | ใช้รุ่นที่อ่านด้วย `np.loadtxt` + decimate หรือตัดด้วย `awk` |

## sim
| อาการ | สาเหตุ | แก้ |
|---|---|---|
| หุ่น sim ล้มเองทั้งที่ policy ปกติ | g1_ctrl สองตัวแย่ง lowcmd / CPU เต็ม (docker CVAT crash-loop เคยดัน load 9.6) | ปิดตัวเกิน, เช็ค `uptime`, `docker ps` |
| ล้มก่อนเข้า Velocity | ไม่ได้ Load key 0 | ทำตามลำดับ ([[Sim2Sim Evaluation]]) |
| แขนสั่นใน FixStand | kd 10 + PD explicit | ข้อจำกัด sim |
| ไฟล์ใน `/tmp` หาย | รีบูตคอมพ์ | สร้างใหม่ (sim สำเนา, `/tmp/g1_fixstand_train`) |

## เครื่องมือ / เอกสาร
| อาการ | สาเหตุ | แก้ |
|---|---|---|
| Obsidian เปิดไม่ขึ้น `ENOSPC` | inotify watches เต็ม (IDE ใช้ไป ~65k) | `/etc/sysctl.d/60-inotify.conf` = 524288 (ตั้งแล้ว 5/11) |
| `git status` เจอไฟล์ doc ที่ไม่ได้แก้ เนื้อหากลายเป็นเวอร์ชันเก่า + ตารางถูกจัดใหม่ | แท็บ IDE ที่เปิดเวอร์ชันเก่าค้างไว้ถูก save ทับ (5/11 เกิดกับ 2 ไฟล์ doc) | ดู diff ก่อน · ถ้าเป็นของเก่า `git restore` · IDE ถามว่าไฟล์เปลี่ยนจากภายนอก → เลือก reload |
| `!` นำหน้าคำสั่งใน terminal ปกติแล้ว `&&` ไม่ทำงาน | `!` ใน bash = กลับค่า exit status | `! cmd` ใช้ได้เฉพาะในช่องพิมพ์ของ Claude Code |

## Isaac
| อาการ | แก้ |
|---|---|
| ถาม EULA แล้ว EOF | `export OMNI_KIT_ACCEPT_EULA=YES` |
| `No module named isaaclab` | ลืม source env / ยังมี conda ซ้อน |
| `mjDSBL_MULTICCD` | newton 1.2 ต้องการ mujoco 3.8 |
| inference tensor error ตอน `env.reset()` ครั้งที่สอง | ห่อ reset ด้วย `torch.inference_mode()` |
| `uv sync` แล้ว isaacsim หาย | `uv sync --extra isaacsim` |

## ดูเพิ่ม
[[DDS and Domains]] · [[Environments Map]] · [[Real Robot Tools]]
