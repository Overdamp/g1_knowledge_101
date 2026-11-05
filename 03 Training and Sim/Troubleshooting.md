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
| ROS ไม่ได้ข้อมูล | `ROS_DOMAIN_ID=0` | ตั้งเป็น 1 |

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
