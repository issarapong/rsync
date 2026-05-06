# 📦 คู่มือการใช้งาน rsync ภาษาไทย

> **rsync** (Remote Sync) คือเครื่องมือสำหรับการซิงก์และคัดลอกไฟล์/โฟลเดอร์ ทั้งบนเครื่องเดียวกันและระหว่างเครื่องผ่านเครือข่าย โดยมีความสามารถในการส่งเฉพาะส่วนที่เปลี่ยนแปลง ทำให้รวดเร็วและประหยัดแบนด์วิดท์

---

## 📋 สารบัญ

1. [การติดตั้ง](#การติดตั้ง)
2. [ไวยากรณ์พื้นฐาน](#ไวยากรณ์พื้นฐาน)
3. [Options ที่ใช้บ่อย](#options-ที่ใช้บ่อย)
4. [Use Cases](#use-cases)
   - [1. คัดลอกไฟล์ในเครื่องเดียวกัน](#1-คัดลอกไฟล์ในเครื่องเดียวกัน)
   - [2. ซิงก์โฟลเดอร์ในเครื่องเดียวกัน](#2-ซิงก์โฟลเดอร์ในเครื่องเดียวกัน)
   - [3. อัปโหลดไฟล์ไปยัง Remote Server](#3-อัปโหลดไฟล์ไปยัง-remote-server)
   - [4. ดาวน์โหลดไฟล์จาก Remote Server](#4-ดาวน์โหลดไฟล์จาก-remote-server)
   - [5. สำรองข้อมูล (Backup)](#5-สำรองข้อมูล-backup)
   - [6. Mirror / Sync แบบสมบูรณ์](#6-mirror--sync-แบบสมบูรณ์)
   - [7. ยกเว้นไฟล์หรือโฟลเดอร์ที่ไม่ต้องการ](#7-ยกเว้นไฟล์หรือโฟลเดอร์ที่ไม่ต้องการ)
   - [8. Dry Run (จำลองโดยไม่ทำจริง)](#8-dry-run-จำลองโดยไม่ทำจริง)
   - [9. ส่งผ่าน SSH Port อื่น](#9-ส่งผ่าน-ssh-port-อื่น)
   - [10. จำกัดความเร็วการถ่ายโอน](#10-จำกัดความเร็วการถ่ายโอน)
   - [11. ซิงก์และลบไฟล์ที่ปลายทางซึ่งไม่มีที่ต้นทาง](#11-ซิงก์และลบไฟล์ที่ปลายทางซึ่งไม่มีที่ต้นทาง)
   - [12. สำรองข้อมูลแบบ Incremental Backup พร้อม Hard Links](#12-สำรองข้อมูลแบบ-incremental-backup-พร้อม-hard-links)
   - [13. ซิงก์ผ่าน rsync Daemon](#13-ซิงก์ผ่าน-rsync-daemon)
   - [14. แสดง Progress และ Transfer Rate](#14-แสดง-progress-และ-transfer-rate)
   - [15. ใช้กับ Cron Job สำหรับ Backup อัตโนมัติ](#15-ใช้กับ-cron-job-สำหรับ-backup-อัตโนมัติ)
5. [ข้อควรระวัง](#ข้อควรระวัง)
6. [Tips & Tricks](#tips--tricks)

---

## การติดตั้ง

```bash
# Debian / Ubuntu
sudo apt install rsync

# RHEL / CentOS / Fedora
sudo dnf install rsync

# macOS (มักติดตั้งมาแล้ว หรือใช้ Homebrew)
brew install rsync

# ตรวจสอบเวอร์ชัน
rsync --version
```

---

## ไวยากรณ์พื้นฐาน

```
rsync [OPTIONS] SOURCE DESTINATION
```

| ส่วน | คำอธิบาย |
|---|---|
| `OPTIONS` | ตัวเลือกต่างๆ เช่น `-a`, `-v`, `--delete` |
| `SOURCE` | ต้นทาง (ไฟล์หรือโฟลเดอร์) |
| `DESTINATION` | ปลายทาง (local หรือ `user@host:/path`) |

> **หมายเหตุเรื่อง Trailing Slash (`/`)**
> - `rsync -a src/ dst/` → คัดลอก **เนื้อหาข้างใน** `src` ไปใส่ใน `dst`
> - `rsync -a src dst/` → คัดลอก **โฟลเดอร์ `src`** ไปไว้ใน `dst` (กลายเป็น `dst/src/`)

---

## Options ที่ใช้บ่อย

| Option | ความหมาย |
|---|---|
| `-a` | Archive mode (รวม `-rlptgoD`) เหมาะสำหรับ backup |
| `-r` | Recursive (รวมโฟลเดอร์ย่อย) |
| `-v` | Verbose (แสดงรายละเอียด) |
| `-z` | บีบอัดข้อมูลระหว่างส่ง |
| `-h` | Human-readable (แสดงขนาดไฟล์ให้อ่านง่าย) |
| `-P` | แสดง Progress + รองรับ Resume |
| `-n` | Dry run (จำลองโดยไม่ทำจริง) |
| `-e` | ระบุ remote shell เช่น `-e ssh` |
| `--delete` | ลบไฟล์ที่ปลายทางถ้าไม่มีที่ต้นทาง |
| `--exclude` | ยกเว้นไฟล์/โฟลเดอร์ที่ระบุ |
| `--include` | รวมไฟล์/โฟลเดอร์ที่ระบุ |
| `--bwlimit` | จำกัดความเร็ว (KB/s) |
| `--link-dest` | ใช้ Hard Link สำหรับ Incremental Backup |
| `--checksum` | เปรียบเทียบด้วย Checksum แทน Timestamp |
| `--progress` | แสดง Progress ระหว่างโอน |

---

## Use Cases

### 1. คัดลอกไฟล์ในเครื่องเดียวกัน

```bash
rsync -vh file.txt /backup/file.txt
```

คัดลอกไฟล์ `file.txt` ไปยัง `/backup/` พร้อมแสดงรายละเอียด

---

### 2. ซิงก์โฟลเดอร์ในเครื่องเดียวกัน

```bash
rsync -avh /home/user/documents/ /mnt/backup/documents/
```

- `-a` → archive mode (รักษา permissions, timestamps, symlinks ฯลฯ)
- `-v` → verbose
- `-h` → human-readable

> **Tip:** ใส่ `/` ท้าย source เพื่อคัดลอกเฉพาะเนื้อหาภายใน ไม่ใช่ตัวโฟลเดอร์

---

### 3. อัปโหลดไฟล์ไปยัง Remote Server

```bash
rsync -avzh /local/path/project/ user@192.168.1.100:/remote/path/project/
```

- `-z` → บีบอัดระหว่างส่ง ช่วยประหยัดแบนด์วิดท์

อัปโหลดโดยระบุ SSH Key:
```bash
rsync -avzh -e "ssh -i ~/.ssh/mykey.pem" /local/project/ user@server.example.com:/var/www/project/
```

---

### 4. ดาวน์โหลดไฟล์จาก Remote Server

```bash
rsync -avzh user@192.168.1.100:/remote/path/data/ /local/path/data/
```

เพียงสลับ source และ destination กับ Use Case ที่ 3

---

### 5. สำรองข้อมูล (Backup)

```bash
rsync -avhz --progress /home/user/ /mnt/usb-backup/home-backup/
```

สำรองข้อมูล `/home/user/` ทั้งหมดไปยัง USB Drive โดยแสดง Progress

สำรองไปยัง Remote Server:
```bash
rsync -avzh /home/user/ backup-user@backup-server.com:/backups/user/
```

---

### 6. Mirror / Sync แบบสมบูรณ์

```bash
rsync -avhz --delete /source/dir/ /destination/dir/
```

`--delete` จะ**ลบไฟล์ที่ปลายทาง** ถ้าไฟล์นั้นถูกลบออกจากต้นทางแล้ว ทำให้ปลายทางเหมือนต้นทางทุกประการ

> ⚠️ **ระวัง:** ใช้ `--delete` ร่วมกับ `-n` (dry run) ก่อนเสมอ เพื่อตรวจสอบว่าจะลบไฟล์อะไรบ้าง

---

### 7. ยกเว้นไฟล์หรือโฟลเดอร์ที่ไม่ต้องการ

```bash
# ยกเว้นไฟล์/โฟลเดอร์เดียว
rsync -avh --exclude='.git' /project/ /backup/project/

# ยกเว้นหลายรายการ
rsync -avh \
  --exclude='.git' \
  --exclude='node_modules' \
  --exclude='*.log' \
  --exclude='*.tmp' \
  /project/ /backup/project/

# ใช้ไฟล์ exclude list
rsync -avh --exclude-from='exclude-list.txt' /project/ /backup/project/
```

ตัวอย่าง `exclude-list.txt`:
```
.git
node_modules
*.log
*.tmp
__pycache__
.DS_Store
```

---

### 8. Dry Run (จำลองโดยไม่ทำจริง)

```bash
rsync -avhn --delete /source/ /destination/
```

`-n` หรือ `--dry-run` จะแสดงผลว่าจะทำอะไรบ้าง **โดยไม่ได้ทำจริง** เหมาะสำหรับทดสอบก่อนรันจริง

---

### 9. ส่งผ่าน SSH Port อื่น

```bash
rsync -avzh -e "ssh -p 2222" /local/data/ user@server.com:/remote/data/
```

ใช้เมื่อ Server ใช้ SSH Port ที่ไม่ใช่ 22 (default)

---

### 10. จำกัดความเร็วการถ่ายโอน

```bash
# จำกัดที่ 1 MB/s (1000 KB/s)
rsync -avzh --bwlimit=1000 /large-files/ user@server.com:/backup/

# จำกัดที่ 500 KB/s
rsync -avzh --bwlimit=500 /data/ /backup/
```

มีประโยชน์เมื่อไม่ต้องการให้ rsync กิน Bandwidth จนส่งผลกระทบต่อการใช้งานอื่น

---

### 11. ซิงก์และลบไฟล์ที่ปลายทางซึ่งไม่มีที่ต้นทาง

```bash
# ทดสอบก่อน (Dry Run)
rsync -avhn --delete /source/ /destination/

# รันจริง
rsync -avh --delete /source/ /destination/
```

---

### 12. สำรองข้อมูลแบบ Incremental Backup พร้อม Hard Links

เทคนิคนี้ช่วยประหยัดพื้นที่ดิสก์ โดยไฟล์ที่ไม่เปลี่ยนแปลงจะถูก Hard Link มาจาก Backup ครั้งก่อน

```bash
#!/bin/bash

DATE=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_DIR="/backups"
SOURCE="/home/user/"
LATEST="$BACKUP_DIR/latest"
DEST="$BACKUP_DIR/$DATE"

rsync -avh \
  --link-dest="$LATEST" \
  "$SOURCE" \
  "$DEST/"

# อัปเดต symlink ให้ชี้ไปที่ backup ล่าสุด
rm -f "$LATEST"
ln -s "$DEST" "$LATEST"

echo "✅ Backup สำเร็จ: $DEST"
```

โครงสร้างที่ได้:
```
/backups/
├── 2026-05-01_00-00-00/
├── 2026-05-02_00-00-00/
├── 2026-05-03_00-00-00/
└── latest -> /backups/2026-05-03_00-00-00  (symlink)
```

---

### 13. ซิงก์ผ่าน rsync Daemon

เมื่อ Server ตั้งค่า rsync daemon (`rsyncd`) ไว้:

```bash
# List modules ที่มี
rsync rsync://server.com/

# ดาวน์โหลดจาก module
rsync -avz rsync://server.com/modulename/ /local/path/

# อัปโหลดไปยัง module (ถ้ามีสิทธิ์)
rsync -avz /local/path/ rsync://server.com/modulename/
```

---

### 14. แสดง Progress และ Transfer Rate

```bash
rsync -avh --progress /large-file.iso user@server.com:/storage/

# หรือใช้ -P (เทียบเท่า --progress --partial)
rsync -avhP /large-file.iso user@server.com:/storage/
```

`--partial` ช่วยให้ resume การส่งต่อได้ถ้าการเชื่อมต่อขาดหาย

---

### 15. ใช้กับ Cron Job สำหรับ Backup อัตโนมัติ

สร้างสคริปต์ `/usr/local/bin/daily-backup.sh`:

```bash
#!/bin/bash

LOG="/var/log/rsync-backup.log"
SOURCE="/home/"
DEST="backup-user@backup-server.com:/backups/daily/"

echo "--- Backup เริ่มต้น: $(date) ---" >> "$LOG"

rsync -avzh \
  --delete \
  --exclude='.cache' \
  --exclude='*.tmp' \
  -e "ssh -i /root/.ssh/backup_key" \
  "$SOURCE" "$DEST" >> "$LOG" 2>&1

if [ $? -eq 0 ]; then
  echo "✅ Backup สำเร็จ: $(date)" >> "$LOG"
else
  echo "❌ Backup ล้มเหลว: $(date)" >> "$LOG"
fi
```

ตั้งค่า Cron (รันทุกคืนเวลา 02:00):
```bash
crontab -e
```
```
0 2 * * * /usr/local/bin/daily-backup.sh
```

---

## ข้อควรระวัง

| ⚠️ เรื่อง | คำแนะนำ |
|---|---|
| `--delete` | ทดสอบด้วย `-n` (dry run) ก่อนเสมอ |
| Trailing Slash | ระวัง `/` ท้าย path — มีผลต่อโครงสร้างโฟลเดอร์ที่ปลายทาง |
| Permissions | `-a` จะรักษา permissions ต้นทาง ถ้าปลายทางเป็น FAT/NTFS อาจเกิดปัญหา |
| Overwrite | rsync จะ overwrite ไฟล์ที่ปลายทางโดยไม่ถามถ้าต่างกัน |
| SSH Key | ควรใช้ SSH Key แทน Password โดยเฉพาะกับ Cron Job |

---

## Tips & Tricks

```bash
# ดู stat สรุปหลังรันเสร็จ
rsync -avh --stats /source/ /dest/

# ใช้ checksum แทน timestamp (ช้ากว่า แต่แม่นยำกว่า)
rsync -avh --checksum /source/ /dest/

# ส่งหลาย source ไปยัง destination เดียว
rsync -avh /dir1/ /dir2/ /dir3/ user@server:/backup/

# สำรองเฉพาะไฟล์ที่นามสกุลที่ต้องการ
rsync -avh --include='*.jpg' --include='*.png' --exclude='*' /photos/ /backup/photos/

# แสดงเฉพาะไฟล์ที่เปลี่ยนแปลง (ไม่แสดงที่เหมือนกัน)
rsync -avhi /source/ /dest/ | grep '^>'
```

---

## 📚 อ้างอิง

- [rsync man page](https://linux.die.net/man/1/rsync)
- [rsync Official Website](https://rsync.samba.org/)

---

> 📝 **จัดทำโดย:** [issarapong](https://github.com/issarapong) | ภาษา: ไทย 🇹🇭
