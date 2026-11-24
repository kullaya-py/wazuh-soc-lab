# 06 – สรุปผลและบทเรียน

## ผลรวม

| หัวข้อ | ผล |
|---|---|
| การทดสอบโจมตีจำลอง | 6 ชุด (8 รายการย่อย) |
| ตรวจจับได้ด้วย rule ที่มีอยู่ | ล็อกอินผิด, Windows brute force, สร้าง/ลบ user, เพิ่มสิทธิ์ admin, SSH brute force |
| Detection gap ที่พบ | 1 – discovery commands (`whoami`, `systeminfo`) จาก cmd → ปิดด้วย rule 100110 |
| False positive ที่พบ | 1 – rule 92213 level 15 ทุกครั้งที่เปิด PowerShell → tuning ด้วย rule 100100 |
| Custom rules | 2 (ทดสอบผ่านทั้งหมด) |

## บทเรียน

1. **Detection ไม่ได้ทำงานตามที่คาดเสมอ** – brute force ผ่าน `net use` ไม่ถึงเกณฑ์ของ correlation rule แต่ Sysmon ยังจับพฤติกรรมของ `net.exe` ได้ การมีหลายชั้น (defense in depth) จึงสำคัญ
2. **Correlation rule** – alert ล็อกอินผิดครั้งเดียว (level 5) ไม่น่ากังวล แต่เมื่อเกิดถี่ในเวลาสั้น Wazuh ยกระดับเป็น brute force (level 10)
3. **Sysmon เพิ่ม visibility มาก** – เห็น process, command line และ parent process ที่ log ปกติของ Windows ไม่มี
4. **Built-in rule มีช่องโหว่** – rule เดิมจับ discovery เฉพาะเมื่อ parent เป็น PowerShell ผู้โจมตีเลี่ยงได้ด้วย cmd
5. **ชื่อ rule ไม่ใช่ความจริงทั้งหมด** – 92039 เรียก `net user /add` ว่า "account discovery" ต้องอ่าน log จริงประกอบ
6. **Triage ด้วยหลักฐาน** – ไฟล์อะไร, process ไหนสร้าง, user ไหน, เวลาตรงกับเหตุการณ์อะไร → ตัดสิน alert level 15 ว่าเป็น false positive ได้อย่างมีเหตุผล
7. **Tuning ต้องแคบที่สุด** – ลดระดับเฉพาะกรณีที่ตรงทุกเงื่อนไข (ชื่อไฟล์ + โฟลเดอร์ + process) เพื่อไม่เปิดช่องให้ผู้โจมตีหลบ
8. **เขียน rule จากข้อมูลจริง** – Wazuh เก็บ path ของ Windows แบบ backslash คู่ ทำให้ rule รอบแรกไม่ match
9. **รายละเอียดของ Windows Security log** – `subStatus` แยก user enumeration (`0xC0000064`) ออกจาก password guessing (`0xC000006A`)
10. **ระวังข้อมูลส่วนตัวในภาพหน้าจอ** – ผลของ `systeminfo` มีอีเมลและ Product ID จึงไม่ได้เผยแพร่ใน repo

## งานต่อยอด

- [ ] Triage alert 92058 (Application Compatibility Database launched) ให้จบ
- [ ] เพิ่ม RAM เป็น 16 GB แล้วแยก endpoint เป็น Windows VM และเพิ่ม Kali เป็นเครื่องโจมตี
- [ ] ทดสอบ File Integrity Monitoring และ Vulnerability Detection
- [ ] ลองตั้ง Active Response บล็อก IP ที่ทำ SSH brute force
- [ ] เชื่อมแจ้งเตือนไปยัง Discord / Slack / อีเมล
