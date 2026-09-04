# บันทึกผลการทดสอบ Attack Scenario และรายงานการทดสอบ (สัปดาห์ที่ 4)
**รหัสนักศึกษา:** 67160028  
**รายวิชา:** การพัฒนาเว็บแอปพลิเคชันและเว็บเอพีไอ  
**บทเรียน:** สัปดาห์ที่ 4 (Security Headers, CORS, Logger, Centralized Error Handling & Attack Scenarios)

---

## 1. ผลการทดสอบ Security Headers (ขั้นตอนที่ 3.1)

เมื่อส่งคำขอ `GET /api/v1/students` ตรวจสอบ Response Headers ที่ส่งกลับมาจากเซิร์ฟเวอร์ (จัดการโดย `helmet()`):

- `X-Content-Type-Options: nosniff` — ป้องกันไม่ให้เบราว์เซอร์เดาชนิดไฟล์ (MIME-type Sniffing)
- `X-Frame-Options: SAMEORIGIN` — ป้องกันการฝังเว็บไซต์ใน iframe เพื่อป้องกันการโจมตีประเภท Clickjacking
- `X-Powered-By` — **ไม่ปรากฏ (ถูกซ่อนเรียบร้อยแล้ว)** ป้องกันไม่ให้ผู้โจมตีทราบว่าเซิร์ฟเวอร์พัฒนาด้วย Express.js

---

## 2. ผลการทดสอบ Error Format มาตรฐาน 4 กรณี (ขั้นตอนที่ 3.3)

| กรณีทดสอบ | คำขอ HTTP | Expected Status | Status จริง | โครงสร้าง Error Response |
|---|---|---|---|---|
| **ไม่พบ route** | `GET /api/v1/unknown-route` | 404 Not Found | 404 | `{"error":{"code":"ROUTE_NOT_FOUND","message":"ไม่พบเส้นทางที่ร้องขอ"}}` |
| **ไม่พบข้อมูลนักศึกษา** | `GET /api/v1/students/999` | 404 Not Found | 404 | `{"error":{"code":"NOT_FOUND","message":"ไม่พบข้อมูลนักศึกษา"}}` |
| **ข้อมูลไม่ครบถ้วน** | `POST /api/v1/students` (ไม่ระบุ name) | 400 Bad Request | 400 | `{"error":{"code":"VALIDATION_ERROR","message":"กรุณาระบุ name, major และ email ให้ครบถ้วน"}}` |
| **อีเมลซ้ำในระบบ** | `POST /api/v1/students` (ใช้อีเมลมีอยู่แล้ว) | 409 Conflict | 409 | `{"error":{"code":"DUPLICATE_EMAIL","message":"อีเมลนี้มีอยู่ในระบบแล้ว"}}` |

---

## 3. บันทึกผลการทดสอบ Attack Scenario (ขั้นตอนที่ 3.4 & 3.5)

### 3.1 Attack Scenario 1: Payload ขนาดใหญ่ผิดปกติ (ขั้นตอนที่ 3.4)
- **การทดสอบ**: ส่งคำขอ `POST /api/v1/students` ที่มีฟิลด์ `name` ความยาว 15,000 ตัวอักษร (`x` 15,000 ตัว ≈ 15KB) ซึ่งเกินกว่าขีดจำกัด 10KB ที่ตั้งไว้ใน `express.json({ limit: "10kb" })`
- **ผลลัพธ์**: เซิร์ฟเวอร์ตอบกลับด้วย **Status Code 413 Payload Too Large** (`{"error":{"code":"entity.too.large","message":"request entity too large"}}`)
- **ข้อสังเกตส่วนตัว**: การจำกัดขนาดของ Request Body มีความสำคัญมากในการป้องกันการโจมตีประเภท Denial of Service (DoS) ที่ผู้โจมตีพยายามส่งข้อมูลขนาดใหญ่เพื่อดึงทรัพยากรหน่วยความจำและ CPU ของเซิร์ฟเวอร์ ทั้งนี้ต้องระบุ `Content-Type: application/json` เพื่อให้ `express.json()` ทำงานร่วมกับขีดจำกัดขนาดดังกล่าวได้ถูกต้อง

### 3.2 Attack Scenario 2: การส่งข้อมูลประเภทผิด (Type Confusion) (ขั้นตอนที่ 3.5)
- **การทดสอบ**: ส่งคำขอ `POST /api/v1/students` โดยระบุ `name` เป็น Object แทน String:
  ```json
  {
    "name": { "malicious": "payload" },
    "major": "วิทยาการคอมพิวเตอร์",
    "email": "test2@example.com"
  }
  ```
- **ผลลัพธ์**: เซิร์ฟเวอร์ประมวลผลคำขอสำเร็จและตอบกลับ **Status Code 201 Created** พร้อมทั้งบันทึกข้อมูล `name` ที่เป็น Object เข้าสู่ระบบ
- **ข้อสังเกตส่วนตัวและการวิเคราะห์ช่องโหว่**:
  - ในปัจจุบัน ระบบตรวจสอบความครบถ้วนของข้อมูลเพียงแค่เงื่อนไข `!name` (Truthy Check) ซึ่งในภาษา JavaScript ค่าที่เป็น Object ถือเป็น Truthy เสมอ จึงผ่านเงื่อนไขไปได้
  - ช่องโหว่นี้คือ **Type Confusion Injection (OWASP Top 10:2025 - A05: Injection / Broken Access Control)** ซึ่งหากนำข้อมูลที่ส่งมาไปเรียกใช้เมธอดของ String ต่อไป (เช่น `.trim()` หรือ `.toLowerCase()`) จะเกิด Error หรือ Unhandled Exception ทำให้แอปพลิเคชันค้าง/ล่มได้
  - การแก้ไขช่องโหว่นี้อย่างยั่งยืน จำเป็นต้องใช้ Data Validation Library เช่น **Zod** (ที่จะได้เรียนในสัปดาห์ที่ 13) เพื่อตรวจทั้งความมีอยู่และชนิดข้อมูล (Data Type) อย่างเคร่งครัด

---

## 4. รายงานผลแบบฝึกหัดต่อยอด (Extra Exercises)

### แบบฝึกหัดที่ 1: Middleware ตรวจสอบ Content-Type
- **วิธีการ**: สร้าง Middleware `requireJson` เพื่อตรวจสอบคำขอที่มี Body (`POST`, `PUT`, `PATCH`) ว่ามี Header `Content-Type: application/json` หรือไม่
- **ผลการทดสอบ**: เมื่อส่งคำขอ `POST` แบบ `text/plain` เซิร์ฟเวอร์ตอบกลับ **Status Code 415 Unsupported Media Type** พร้อมข้อความอธิบาย

### แบบฝึกหัดที่ 2: Rate Limiting เบื้องต้น (`express-rate-limit`)
- **วิธีการ**: ตั้งค่า `createStudentLimiter` จำกัดให้แต่ละ IP ส่งคำขอ `POST /api/v1/students` ได้ไม่เกิน 5 ครั้งต่อนาที
- **ผลการทดสอบ**: เมื่อยิงคำขอถี่เกิน 5 ครั้ง/นาที คำขอถัดมาถูกปฏิเสธด้วย **Status Code 429 Too Many Requests**

### แบบฝึกหัดที่ 3: ตรวจสอบความยาวของค่าฟิลด์
- **วิธีการ**: เพิ่มเงื่อนไขตรวจสอบความยาวของ `name` ไม่ให้เกิน 100 ตัวอักษรใน `POST /api/v1/students`
- **ผลการทดสอบ**: ส่ง `name` ความยาว 105 ตัวอักษร ได้รับ **Status Code 400 Bad Request** (`ชื่อ (name) ต้องมีความยาวไม่เกิน 100 ตัวอักษร`)

### แบบฝึกหัดที่ 4: ทดสอบผลกระทบของลำดับ Middleware
- **การทดลอง**: ทดลองย้าย `app.use(helmet())` ไปวางไว้ *หลัง* Route ทั้งหมด
- **ผลลัพธ์**: Security Headers (`X-Content-Type-Options`, `X-Frame-Options`) หายไป และ `X-Powered-By: Express` แสดงขึ้นมาตามปกติ
- **ข้อสรุปเชิงทฤษฎี**: Express ประมวลผล Middleware แบบลำดับ (Pipeline) จากบนลงล่าง เมื่อคำขอตรงกับ Route Handler ใดๆ แล้ว Route Handler นั้นจะตอบกลับ (`res.json`) และจบ cycle ทันทีโดยไม่เรียก `next()` ทำให้ Middleware ที่วางไว้ด้านหลังไม่ถูกเรียกทำงาน Security Middleware จึงต้องวางไว้ก่อน Route ทั้งหมดเสมอ
