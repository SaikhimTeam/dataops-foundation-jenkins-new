# 🚀 ETL Pipeline ง่ายๆ พร้อม Jenkins

## 📖 นี่คืออะไร?

โปรเจคนี้เป็น **ระบบประมวลผลข้อมูล** ที่ทำงานแบบอัตโนมัติ:
- อ่านข้อมูล Loan จากไฟล์ CSV
- ทำความสะอาดข้อมูล
- แปลงเป็นรูปแบบ Database 
- ส่งเข้า SQL Server อัตโนมัติ
- มี Jenkins ช่วยตรวจสอบและรันให้

## 🎯 ทำอะไรได้บ้าง?

### ✨ **ระบบทำงานอัตโนมัติ**
- ✅ อ่านไฟล์ข้อมูลขนาดใหญ่ได้
- ✅ ทำความสะอาดข้อมูลอัตโนมัติ  
- ✅ ตรวจสอบคุณภาพข้อมูลด้วย Unit Test
- ✅ ส่งข้อมูลเข้า Database ให้อัตโนมัติ
- ✅ แจ้งเตือนเมื่อมีปัญหา

### 📊 **จัดข้อมูลเป็นระเบียบ**
- สร้างตาราง Fact และ Dimension Tables
- กรองเฉพาะข้อมูลปี 2016-2019
- ลบคอลัมน์ที่มีข้อมูลขาดมากกว่า 30%

---

## 🚀 วิธีใช้งาน

### **ขั้นตอนที่ 1: เตรียม Docker**
```bash
# รัน Jenkins ใน Docker
docker run -d --name jenkins-python --restart unless-stopped -e TZ=Asia/Bangkok -p 8080:8080 -p 50000:50000 -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock amornpan/jenkins-python:latest
```

### **ขั้นตอนที่ 2: เข้า Jenkins**
1. เปิดเบราว์เซอร์ไป http://localhost:8080
2. ดู password จาก Docker logs:
   ```bash
   docker logs jenkins-python
   ```
3. ใส่ password และ setup Jenkins

### **ขั้นตอนที่ 3: สร้าง Job**
1. กด **New Item** → **Pipeline**
2. ใส่ Git URL ของโปรเจคนี้
3. Script Path: `Jenkinsfile`
4. Save และ Build Now

### **ขั้นตอนที่ 4: ดูผลลัพธ์**
- Jenkins จะรันทุกอย่างอัตโนมัติ
- ดูผลได้ใน Console Output
- ข้อมูลจะถูกส่งไป SQL Server: `34.10.241.203`

---

## 🏗️ โครงสร้างโปรเจค

```
📁 dataops-foundation-jenkins-new/
├── 📂 functions/           # ฟังก์ชันสำหรับประมวลผลข้อมูล
│   ├── guess_column_types.py      # เดาประเภทข้อมูล
│   ├── filter_issue_date_range.py # กรองตามวันที่
│   └── clean_missing_values.py    # ทำความสะอาดข้อมูล
├── 📂 tests/              # ทดสอบระบบ
├── 📂 data/               # ไฟล์ข้อมูล
├── etl_pipeline.py        # โปรแกรมหลัก
└── Jenkinsfile           # คำสั่ง Jenkins
```

---

## ⚙️ ระบบทำงานอย่างไร?

### **🔄 Jenkins จะทำให้อัตโนมัติ 6 ขั้นตอน:**

#### **1. 🔍 ตรวจสอบโปรเจค**
- เช็คว่าไฟล์ครบไหม
- เตรียม environment

#### **2. 🐍 ติดตั้ง Python**
- สร้าง virtual environment
- ติดตั้ง pandas, numpy, sqlalchemy

#### **3. 🧪 ทดสอบระบบ (รันพร้อมกัน 3 ตัว)**
- **Test A:** ทดสอบการเดาประเภทข้อมูล
- **Test B:** ทดสอบการกรองวันที่
- **Test C:** ทดสอบการทำความสะอาด

#### **4. ✅ ตรวจสอบความพร้อม**
- เช็คว่า import functions ได้ไหม
- เช็คว่าไฟล์ข้อมูลอ่านได้ไหม

#### **5. 🔄 ประมวลผลข้อมูล**
- อ่านไฟล์ CSV
- ทำความสะอาดข้อมูล
- สร้างตารางฐานข้อมูล

#### **6. 🚀 ส่งเข้า Database**
- ส่งข้อมูลไป SQL Server
- ตรวจสอบว่าส่งสำเร็จ

---

## 📊 ได้ข้อมูลอะไรออกมา?

### **ตาราง Dimension (ข้อมูลอ้างอิง):**
- **home_ownership_dim** - ประเภทการเป็นเจ้าของบ้าน
- **loan_status_dim** - สถานะเงินกู้
- **issue_d_dim** - ข้อมูลวันที่ (เดือน, ปี, ไตรมาส)

### **ตาราง Fact (ข้อมูลหลัก):**
- **loans_fact** - ข้อมูลเงินกู้ทั้งหมด
- มีข้อมูล: จำนวนเงิน, อัตราดอกเบิ้ย, งวดผ่อน

---

## 🛠️ ทดสอบเฉพาะส่วน (ถ้าอยากลอง)

### **รันแค่ประมวลผลข้อมูล:**
```bash
cd dataops-foundation-jenkins-new
python etl_pipeline.py
```

### **รันพร้อมส่งเข้า Database:**
```bash
python etl_pipeline.py --deploy
```

### **รันแค่ Test:**
```bash
cd tests
python guess_column_types_test.py
python filter_issue_date_range_test.py  
python clean_missing_values_test.py
```

---

## ❓ เกิดปัญหาแก้ไงบ้าง?

### **🔴 Jenkins ไม่ทำงาน**
```
ปัญหา: Jenkins container ไม่รัน
แก้: docker restart jenkins-python
```

### **🔴 ไฟล์ข้อมูลไม่เจอ**
```
ปัญหา: Data file not found
แก้: เช็คว่ามีไฟล์ data/LoanStats_web_small.csv ไหม
```

### **🔴 เชื่อมต่อ Database ไม่ได้**
```
ปัญหา: Database connection failed
แก้: เช็ค network และ password ใน Jenkins Credentials
```

### **🔴 Test ล้มเหลว**
```
ปัญหา: Unit tests failed
แก้: ดู Console Output ว่า error อะไร แล้วแก้โค้ด
```

---

## 🎯 สรุปง่ายๆ

โปรเจคนี้ทำให้คุณ:
1. **ประมวลผลข้อมูลขนาดใหญ่** ได้แบบอัตโนมัติ
2. **ตรวจสอบคุณภาพข้อมูล** ด้วย Unit Testing
3. **ส่งข้อมูลเข้า Database** โดยไม่ต้องทำเอง
4. **มั่นใจได้** ว่าระบบทำงานถูกต้อง

## 🎉 ข้อดี

- ✅ **ง่ายใช้**: แค่กด Build ใน Jenkins
- ✅ **ปลอดภัย**: มี Test ตรวจสอบก่อนส่งข้อมูล  
- ✅ **รวดเร็ว**: รันแบบ parallel testing
- ✅ **น่าเชื่อถือ**: มี error handling ครบถ้วน
- ✅ **ขยายได้**: เพิ่ม functions ใหม่ได้ง่าย

---

*💡 **เคล็ดลับ:** ถ้าต้องการดูรายละเอียดการทำงาน ไปดูใน Console Output ของ Jenkins หลังจากกด Build*