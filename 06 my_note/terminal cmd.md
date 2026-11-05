---
tags: [robot, commands, runbook]
aliases: [Command Runbook, คำสั่งเทอร์มินัล, runbook]
updated: 2026-11-05
---
# terminal cmd

คำสั่งที่รันจริงบนหุ่น (ดึงจาก `~/.bash_history`) + คำสั่งสำหรับเทสครั้งถัดไป คัดลอกไปวางได้เลย บรรทัดที่ขึ้นต้นด้วย `#` เป็นคำอธิบาย bash ไม่รัน วางทั้งก้อนได้
ขั้นตอนและเกณฑ์ผ่าน: [[Test Day Procedure]] · ผลที่วัดได้: [[Real Robot Results]] · สคริปต์ทั้งหมด: [[Real Robot Tools]]

> [!danger] ก่อนรันคำสั่งที่ส่งแรง
> หุ่นแขวนรอก · คนถือรีโมทพร้อม **L2 + B** · ไม่กด L2 + A · รัน `--dry-run` ก่อนเสมอ · ไม่ใช้ `--yes`

## สคริปต์แต่ละตัวทำอะไร (สรุปสั้น)
| สคริปต์ | ส่งแรงไหม | หน้าที่ |
|---|---|---|
| `robot_health.py` | ไม่ | ฟัง ~3 s แล้วสรุป PASS/WARN/FAIL: Hz ของ lowstate, mode_machine, มีใครส่ง lowcmd อยู่ไหม, motion mode, มอเตอร์ offline, อุณหภูมิ, IMU, แบต |
| `motion_mode.py` | เปลี่ยนโหมด | คุยกับ motion_switcher ของ Unitree: `check` ดูโหมด, `release` ปิด controller ของ Unitree (เข้า debug), `select ai` เปิดกลับ |
| `offset_check.py` | ไม่ | ให้คนหมุนข้อต่อจนชนสุด แล้วเทียบกับจุดชนของ 28/10 ถ้าต่างมาก = ค่าศูนย์ encoder เลื่อน |
| `gain_check.py` | ไม่ (ไม่ใช้หุ่น) | เทียบ kp/kd/scale ใน deploy.yaml กับที่ฝังใน ONNX ว่าตรงกัน |
| `lowstate_monitor.py` | ไม่ | อ่าน `rt/lowstate` (หุ่น → เรา): q, dq, tau_est, อุณหภูมิ ของทุก slot · `--log` เซฟ CSV |
| `lowcmd_monitor.py` | ไม่ | อ่าน `rt/lowcmd` (เรา → หุ่น): q_des, kp, kd, tau_ff ที่ถูกส่งจริง และบอกว่า gain ตรงกับ state ไหนของ g1_ctrl · `--log` เซฟ CSV |
| `deploy_byobu.sh` | ไม่ (แค่เปิดจอ) | เปิด byobu 4 ช่อง พิมพ์คำสั่งรอไว้ให้ ยังไม่กด Enter |
| `sysid_motion.py` | **ส่ง** | ส่ง lowcmd เองแทน g1_ctrl: ท่า step / chirp ทีละข้อต่อ, กด L2 + B แล้วเข้า damping |
| `fixstand_train_gains.sh` | ไม่ (แค่ build) | สร้างสำเนา g1_ctrl ที่ `/tmp/g1_fixstand_train` ให้ FixStand ใช้ gain ตอนเทรน |
| `g1_ctrl` | **ส่ง** | controller ของเรา (FSM: Passive → FixStand → Velocity/policy, Handover) |
| `pd_check.py` | ไม่ (อ่าน CSV) | เอา log state + cmd มา fit หาว่ามอเตอร์ให้แรงตาม kp/kd ที่สั่งกี่เท่า (`kp x`, `kd x`) และ delay |

## 0. เตรียมเครื่อง (ทุกครั้ง)
```bash
# ตั้ง IP ให้การ์ดแลนที่ต่อหุ่น (หุ่นอยู่วง 192.168.123.x) และเปิดการ์ด · หายเมื่อรีบูต/ถอดสาย
sudo ip addr add 192.168.123.123/24 dev enp43s0 && sudo ip link set enp43s0 up
# เปิด multicast บน lo · ถ้าไม่เปิด โปรแกรม DDS บนเครื่องเดียวกันจะหากันไม่เจอ (ไม่มี error)
sudo ip link set lo multicast on
# เช็คว่าเห็นหุ่น (161 = บอร์ด motion control) ต้อง 0% packet loss
ping -c 3 192.168.123.161
# เข้า repo + env ที่มี unitree_sdk2py · unset CYCLONEDDS_URI เพราะค่าจาก ROS ทำให้ SDK พัง
cd ~/nvidia/unitree/unitree_rl_mjlab && conda activate g1_sdk && unset CYCLONEDDS_URI
# ตัวแปรย่อ: S = โฟลเดอร์สคริปต์, D = วันที่ (ใช้ตั้งชื่อ log)
S=deploy/robots/g1_23dof/scripts
D=$(date +%Y%m%d)
```

## 1. เช็คหุ่น (อ่านอย่างเดียว)
```bash
# สรุปสุขภาพหุ่น ต้อง PASS ทุกบรรทัด และ "23 motors online" (ระวัง slot 25/26 แขนขวา)
python $S/robot_health.py
# ดูโหมดปัจจุบัน ตอนบูตต้องเป็น name='ai' (controller ของ Unitree คุมอยู่)
python $S/motion_mode.py check
# เช็คค่าศูนย์เฉพาะแขนขวา: หมุนข้อต่อที่ระบุจนชนทั้งสองฝั่ง ดูว่า OK (|d| เล็ก) · Ctrl-C เพื่อจบและเซฟผล
python $S/offset_check.py --joints right_elbow,right_wrist,right_shoulder
# เหมือนข้างบนแต่ทั้ง 23 ข้อ
python $S/offset_check.py
# (ไม่ใช้หุ่น) เช็คว่า deploy.yaml ของ policy v0 ตรงกับ ONNX · ทำก่อนวันเทสได้
python $S/gain_check.py deploy/robots/g1_23dof/config/policy/velocity/v0
```

## 2. เข้า / ออก debug
```bash
# ปิด controller ของ Unitree เพื่อให้เราส่ง lowcmd เองได้ · ต้องพิมพ์ yes · ไฟเปลี่ยนเป็นเหลือง
python $S/motion_mode.py release
# ยืนยันว่าเข้า debug แล้ว: name='' และไม่มี lowcmd จากใคร
python $S/motion_mode.py check
# กลับไปให้ Unitree คุม (zero torque, ไฟม่วง) · ใช้เมื่อ Handover จากรีโมทไม่สำเร็จ หรือก่อนปิดเครื่อง
python $S/motion_mode.py select ai
```

## 3. byobu (T1 lowstate · T2 lowcmd · T3 g1_ctrl · T4 คำสั่ง)
```bash
# เปิดจอ 4 ช่อง · RUN = เลขรอบ, TAG = ชื่องาน → log ชื่อ ~/g1_{state,cmd}_<วันที่>_<TAG><RUN>.csv
$S/deploy_byobu.sh <RUN> <TAG>
```
คำสั่งที่พิมพ์รอไว้ในแต่ละช่อง (กด Enter เอง):
```bash
# T1: บันทึกสิ่งที่หุ่นรายงาน (มุม, ความเร็ว, แรงประมาณจากกระแส) ~1000 Hz · เปิดก่อนส่งแรงเสมอ
python $S/lowstate_monitor.py --iface enp43s0 --log ~/g1_state_${D}_${TAG}${RUN}.csv
# T2: บันทึกคำสั่งที่ส่งไปหุ่น · ถ้าเห็น MISMATCH = gain ไม่ตรงกับที่คาด ให้หยุด
python $S/lowcmd_monitor.py --iface enp43s0 --log ~/g1_cmd_${D}_${TAG}${RUN}.csv
```
F6 = detach (โปรแกรมยังรันต่อ) · `byobu attach -t g1deploy` = กลับ · ดู [[Byobu Guide]]

## 4. sysid step (3/11)
```bash
# เปิด byobu ชื่องาน sysid · ช่อง T3 ไม่ต้องเปิด g1_ctrl เพราะ sysid_motion ส่ง lowcmd เอง (สองตัวพร้อมกัน = แย่งกัน)
$S/deploy_byobu.sh 1 sysid
# ดูแผนก่อน ไม่ส่งอะไร: จะขยับข้อไหน กี่ rad นานเท่าไร
python $S/sysid_motion.py --groups armL --joints elbow --kinds step --dry-run
# step ศอกซ้าย (กระโดดมุมเล็กๆ แล้วค้าง) · --segments เซฟเวลาเริ่ม/จบของแต่ละท่า เอาไว้ให้ pd_check fit ทีละช่วง
python $S/sysid_motion.py --groups armL --joints elbow --kinds step --segments ~/sysid_seg_20261103_light.csv
# step ศอกขวา (ดูมอเตอร์ 25 ด้วย)
python $S/sysid_motion.py --groups armR --joints elbow --kinds step --segments ~/sysid_seg_20261103_lightR.csv
# step เข่าซ้าย: ดูแผนก่อนเพราะขาแรงกว่าแขน
python $S/sysid_motion.py --groups legL --joints knee  --kinds step --dry-run
python $S/sysid_motion.py --groups legL --joints knee  --kinds step --segments ~/sysid_seg_20261103_kneeL.csv
# รอบเทส L2 + B: กดระหว่างท่า ต้องเข้า damping ทันทีและสคริปต์จบเอง
python $S/sysid_motion.py --groups legL --joints knee  --kinds step --segments ~/sysid_seg_20261103_kneeL_stop.csv
# วิเคราะห์: kp x ≈ 1 = มอเตอร์ให้แรงตาม kp ที่สั่ง (ใส่ --segments ไฟล์ข้างบนเพื่อ fit เฉพาะช่วง step)
python $S/pd_check.py ~/g1_state_20261103_sysid1.csv ~/g1_cmd_20261103_sysid1.csv
```
> [!warning] `--segments` ตั้งชื่อใหม่ทุกรอบ ไม่งั้นทับไฟล์เดิม (30/10)

## 5. FixStand ด้วย gain ที่เทรน (3/11)
```bash
# build สำเนา g1_ctrl ที่ FixStand ใช้ gain ตอนเทรน (แขน 14.3/0.9 ...) แทน gain Unitree (เข่า 150/4)
# อยู่ใน /tmp → หายเมื่อรีบูต ต้อง build ใหม่ทุกวันเทส
$S/fixstand_train_gains.sh
# เปิด byobu โดยให้ T3 ใช้ g1_ctrl สำเนานั้น (G1_CTRL_DIR) · ชื่องาน bm (BeyondMimic gains)
G1_CTRL_DIR=/tmp/g1_fixstand_train/build $S/deploy_byobu.sh 1 bm
# T3: รัน g1_ctrl บนการ์ดที่ต่อหุ่น · env -u / LD_LIBRARY_PATH = ใช้ DDS ของ Unitree ไม่ใช่ของ ROS
#     เริ่มที่ Passive (damping) → รีโมทเข้า FixStand → ค้างดูว่านิ่ง ไม่สั่น
cd /tmp/g1_fixstand_train/build && env -u CYCLONEDDS_URI LD_LIBRARY_PATH=/opt/unitree_robotics/lib ./g1_ctrl --network enp43s0
# วิเคราะห์: kp x 0.8–1.2, kd x 0.5–1.5
python $S/pd_check.py ~/g1_state_${D}_bm1.csv ~/g1_cmd_${D}_bm1.csv
```
ดู [[FixStand Train Gains]]

## 6. g1_ctrl ปกติ P3b (30/10)
```bash
# byobu ชื่องาน run · T3 เปิด g1_ctrl ตัวหลัก (FixStand = gain Unitree, Velocity = policy v0)
$S/deploy_byobu.sh 1 run
# วิเคราะห์ log รอบนั้น: kp x / kd x ของ Passive และ FixStand + delay
python $S/pd_check.py ~/g1_state_20261030_run1.csv ~/g1_cmd_20261030_run1.csv
```

## 7. Handover + ปิด
- รีโมท: Passive → **L1 + R1** ค้าง 1 s → **START** → g1_ctrl หยุดส่ง lowcmd, สลับเป็น `ai` แล้วปิดตัวเอง → ไฟม่วง ([[Handover]])
- เช็ค:
  ```bash
  # ต้องเห็น name='ai' และ 23 motors online
  python $S/robot_health.py
  ```
- ปิดเครื่อง: [[Safe Shutdown]]

## 8. ครั้งถัดไป: chirp วัด kd (ยังไม่เคยรันบนหุ่นจริง)
แผน: [แผนทดสอบ G1 23-DoF](https://claude.ai/artifact/1q3CrcyKgpS5AsJQY88S6T) · ค่าเทียบจาก sim ใน [[kp kd Verification]]
chirp = คลื่น sine ที่เร่งความถี่ 0.3 → 2.5 Hz ใน 12 s ข้อต่อขยับเร็วขึ้นเรื่อยๆ ทำให้เทอม D (kd × ความเร็ว) ใหญ่พอจะวัดได้
```bash
# byobu ชื่องาน chirp · T3 ไม่เปิด g1_ctrl
$S/deploy_byobu.sh 1 chirp
# ต้อง name='' และไม่มี lowcmd จากใคร ก่อนให้ sysid ส่ง
python $S/motion_mode.py check

# ดูแผน chirp แขนซ้ายทั้ง 5 ข้อ (ไม่ส่งอะไร)
python $S/sysid_motion.py --groups armL --kinds chirp --dry-run
# chirp แขนซ้าย ~71 s
python $S/sysid_motion.py --groups armL --kinds chirp --segments ~/sysid_seg_${D}_chirp_armL.csv
# วิเคราะห์ทีละท่า: เทียบ kp x / kd x กับ sim (0.95–0.99)
python $S/pd_check.py ~/g1_state_${D}_chirp1.csv ~/g1_cmd_${D}_chirp1.csv --segments ~/sysid_seg_${D}_chirp_armL.csv

# รอบเทส L2 + B: กดตอนแขนกำลังแกว่งเร็ว ต้องหยุดทันที · สังเกตแขนหลังหยุด (ค้างหรือตกช้าๆ)
python $S/sysid_motion.py --groups armL --kinds chirp --segments ~/sysid_seg_${D}_chirp_armL_stop.csv

# กลุ่มที่เหลือ ทีละกลุ่ม ผ่านค่อยไปต่อ (รัน --dry-run ก่อนทุกกลุ่ม)
python $S/sysid_motion.py --groups armR  --kinds chirp --segments ~/sysid_seg_${D}_chirp_armR.csv
python $S/sysid_motion.py --groups waist --kinds chirp --segments ~/sysid_seg_${D}_chirp_waist.csv  # ~19 s
python $S/sysid_motion.py --groups legL  --kinds chirp --segments ~/sysid_seg_${D}_chirp_legL.csv   # ~84 s
python $S/sysid_motion.py --groups legR  --kinds chirp --segments ~/sysid_seg_${D}_chirp_legR.csv
```
- เปลี่ยน RUN ของ byobu ทุกครั้งที่เริ่มกลุ่มใหม่ ถ้าอยากให้ log แยกไฟล์ (`deploy_byobu.sh 2 chirp` ...)
- ผ่านเมื่อ `kp x` / `kd x` ห่างจาก sim ไม่เกิน ±0.1 และ corr ≥ 0.9 · เข่าใน sim ได้แค่ 0.85–0.93 (P กับ D หักล้างกัน) อย่าตกใจถ้าต่ำ

## 9. ดูบนตัวหุ่น
```bash
# SSH เข้าบอร์ด 164 (PC บนหุ่น) เช่นดูว่ามีโปรเซสไหนส่ง lowcmd อยู่
ssh unitree@192.168.123.164
```

## ไฟล์ log ที่มีอยู่
| วันที่ | ไฟล์ |
|---|---|
| 28/10 | `~/g1_baseline_20261028_1826.csv`, `~/g1_joints_20261028_1848.csv` |
| 30/10 | `~/g1_{state,cmd}_20261030_run1*.csv`, `_sysid1` |
| 3/11 | `~/g1_{state,cmd}_20261103_{sysid1-4,bm1,run2}.csv`, `~/offset_20261103_right.txt` |

## ดูเพิ่ม
[[Real Robot Tools]] · [[Sysid Motion]] · [[Stop Mechanisms]] · [[Network Setup]]
