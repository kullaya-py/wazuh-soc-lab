# 03 – ติดตั้ง Wazuh agent + Sysmon บน Windows

## 1. ติดตั้ง Wazuh agent ✅

1. ใน dashboard กด **Deploy new agent**
2. เลือก **Windows → MSI**, Server address: `192.168.38.130`, Agent name: `win-host`
3. เปิด **PowerShell แบบ Administrator** (คลิกขวาปุ่ม Start → Terminal (Admin)) แล้วติดตั้ง:

   ```powershell
   Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.10-1.msi -OutFile $env:tmp\wazuh-agent.msi
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:tmp\wazuh-agent.msi WAZUH_MANAGER=192.168.38.130 WAZUH_AGENT_NAME=win-host"
   NET START Wazuh
   ```

4. agent ลงทะเบียนสำเร็จเป็น ID `001` (สถานะ pending ช่วงแรก แล้วเปลี่ยนเป็น Active)

![Agent registered](../screenshots/06-windows-agent-registered.png)

### ปัญหาที่เจอและวิธีแก้

| อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|
| `NET START Wazuh` → *The service name is invalid* | `msiexec /q` รันเบื้องหลังและติดตั้งล้มเหลวแบบเงียบ เพราะ PowerShell ไม่ได้รันแบบ Administrator | ติดตั้งใหม่ด้วย `Start-Process ... -Wait` (ไม่ใช้ `/q`) เพื่อให้เห็นหน้าต่างติดตั้งและ error |
| `NET START Wazuh` → *System error 5 – Access is denied* | PowerShell ไม่ได้รันแบบ Administrator (path `C:\WINDOWS\system32` ไม่ได้แปลว่าเป็น admin) | เปิด Terminal (Admin) แล้วสั่ง `NET START Wazuh` ใหม่ |

บทเรียน: ตรวจแถบชื่อหน้าต่างว่ามีคำว่า **Administrator** ก่อนติดตั้งหรือจัดการ service เสมอ

## 2. ติดตั้ง Sysmon (TODO)

1. ดาวน์โหลด Sysmon จาก Microsoft Sysinternals แล้วแตกไฟล์
2. ดาวน์โหลด config ที่นิยมใช้ เช่น `sysmonconfig-export.xml` ของ SwiftOnSecurity
3. ติดตั้ง (PowerShell แบบ Administrator ในโฟลเดอร์ที่แตกไฟล์):

   ```powershell
   .\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
   ```

## 3. ให้ agent ส่ง log ของ Sysmon (TODO)

แก้ไฟล์ `C:\Program Files (x86)\ossec-agent\ossec.conf` (เปิด Notepad แบบ Administrator)
โดยเพิ่มเนื้อหาจาก [`configs/windows-agent-sysmon.xml`](../configs/windows-agent-sysmon.xml) ไว้ภายใน `<ossec_config>` แล้วรีสตาร์ต agent:

```powershell
Restart-Service -Name WazuhSvc
```

ตรวจสอบใน **Threat Hunting** ด้วยการค้นหา `data.win.system.channel: "Microsoft-Windows-Sysmon/Operational"`
