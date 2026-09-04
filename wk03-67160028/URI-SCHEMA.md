# เอกสาร URI Schema - student-api
## รหัสนักศึกษา: 67160028 | ชื่อ-นามสกุล: ณัฐวุฒิ รู้การนา

เอกสารฉบับนี้จัดทำขึ้นเพื่อระบุและบันทึกข้อกำหนด **URI Schema**, **HTTP Methods**, และ **Status Codes** ของ API ระบบจัดการข้อมูลนักศึกษา (`student-api`) ตามมาตรฐาน RESTful Web API (สัปดาห์ที่ 3)

---

## 1. ตาราง URI Schema ทั้งหมด (URI Schema Table)

| Method | URI | คำอธิบาย | Status Code ที่เป็นไปได้ | Safe | Idempotent |
| :--- | :--- | :--- | :--- | :---: | :---: |
| **GET** | `/api/v1/students` | ดึงรายการนักศึกษาทั้งหมด | `200 OK` | Yes | Yes |
| **GET** | `/api/v1/students/{id}` | ดึงข้อมูลนักศึกษารายบุคคล (รองรับ `?include=courses`) | `200 OK`, `404 Not Found` | Yes | Yes |
| **POST** | `/api/v1/students` | เพิ่มนักศึกษาใหม่ | `201 Created`, `400 Bad Request`, `409 Conflict` | No | No |
| **PUT** | `/api/v1/students/{id}` | แก้ไขข้อมูลนักศึกษาทั้งระเบียน (Full Update) | `200 OK`, `400 Bad Request`, `404 Not Found` | No | Yes |
| **PATCH** | `/api/v1/students/{id}` | แก้ไขข้อมูลนักศึกษาบางส่วน (Partial Update) | `200 OK`, `404 Not Found` | No | No |
| **DELETE** | `/api/v1/students/{id}` | ลบข้อมูลนักศึกษา | `200 OK`, `404 Not Found` | No | Yes |

---

## 2. รายละเอียด Endpoint และตัวอย่าง Payload (Endpoint Specifications)

### 2.1 GET `/api/v1/students`
- **คำอธิบาย:** ดึงรายการข้อมูลนักศึกษาทั้งหมด
- **Status Code:** `200 OK`
- **Response Body:**
```json
{
  "message": "สำเร็จ",
  "data": [
    {
      "id": 1,
      "name": "สมชาย ใจดี",
      "major": "วิทยาการคอมพิวเตอร์",
      "email": "somchai@example.com",
      "phone": "080-000-0001",
      "courseIds": [101, 102]
    },
    {
      "id": 2,
      "name": "สมหญิง รักเรียน",
      "major": "เทคโนโลยีสารสนเทศ",
      "email": "somying@example.com",
      "phone": "080-000-0002",
      "courseIds": [102]
    }
  ]
}
```

---

### 2.2 GET `/api/v1/students/{id}`
- **คำอธิบาย:** ดึงข้อมูลนักศึกษารายบุคคล รองรับ Query Parameter `?include=courses`
- **Query Parameter:** `include=courses` (Optional)
- **Status Codes:**
  - `200 OK`: ดึงข้อมูลสำเร็จ
  - `404 Not Found`: ไม่พบ ID นักศึกษาที่ระบุ
- **Request Example:** `GET /api/v1/students/1?include=courses`
- **Response Body (200 OK - มี `include=courses`):**
```json
{
  "message": "สำเร็จ",
  "data": {
    "id": 1,
    "name": "สมชาย ใจดี",
    "major": "วิทยาการคอมพิวเตอร์",
    "email": "somchai@example.com",
    "phone": "080-000-0001",
    "courseIds": [101, 102],
    "courses": [
      {
        "id": 101,
        "courseName": "การเขียนโปรแกรมเบื้องต้น",
        "credit": 3
      },
      {
        "id": 102,
        "courseName": "โครงสร้างข้อมูล",
        "credit": 3
      }
    ]
  }
}
```
- **Response Body (404 Not Found):**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "ไม่พบข้อมูลนักศึกษา"
  }
}
```

---

### 2.3 POST `/api/v1/students`
- **คำอธิบาย:** เพิ่มข้อมูลนักศึกษาใหม่เข้าสู่ระบบ
- **Headers:** `Content-Type: application/json`
- **Request Body Example:**
```json
{
  "name": "วิชัย ตั้งใจเรียน",
  "major": "วิศวกรรมซอฟต์แวร์",
  "email": "wichai@example.com",
  "phone": "080-000-0003"
}
```
- **Status Codes:**
  - `201 Created`: เพิ่มนักศึกษาสำเร็จ
  - `400 Bad Request`: ข้อมูลไม่สมบูรณ์ (ขาด `name`, `major` หรือ `email`)
  - `409 Conflict`: อีเมลนี้มีอยู่ในระบบแล้ว (Email Duplication)
- **Response Body (201 Created):**
```json
{
  "message": "เพิ่มข้อมูลสำเร็จ",
  "data": {
    "id": 3,
    "name": "วิชัย ตั้งใจเรียน",
    "major": "วิศวกรรมซอฟต์แวร์",
    "email": "wichai@example.com",
    "phone": "080-000-0003",
    "courseIds": []
  }
}
```
- **Response Body (400 Bad Request):**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "กรุณาระบุ name, major และ email ให้ครบถ้วน"
  }
}
```
- **Response Body (409 Conflict):**
```json
{
  "error": {
    "code": "DUPLICATE_EMAIL",
    "message": "อีเมลนี้มีอยู่ในระบบแล้ว"
  }
}
```

---

### 2.4 PUT `/api/v1/students/{id}`
- **คำอธิบาย:** แก้ไขข้อมูลนักศึกษาทั้งระเบียน (Full Update)
- **Headers:** `Content-Type: application/json`
- **Request Body Example:**
```json
{
  "name": "สมชาย ใจดีมาก",
  "major": "วิทยาการคอมพิวเตอร์",
  "email": "somchai@example.com"
}
```
- **Status Codes:**
  - `200 OK`: แก้ไขสำเร็จ
  - `400 Bad Request`: ระบุข้อมูลไม่ครบถ้วน
  - `404 Not Found`: ไม่พบ ID นักศึกษาที่ระบุ

---

### 2.5 PATCH `/api/v1/students/{id}`
- **คำอธิบาย:** แก้ไขข้อมูลนักศึกษาบางส่วน (Partial Update)
- **Headers:** `Content-Type: application/json`
- **Request Body Example:**
```json
{
  "major": "ปัญญาประดิษฐ์"
}
```
- **Status Codes:**
  - `200 OK`: แก้ไขเฉพาะฟิลด์ที่ส่งมาสำเร็จ (ฟิลด์อื่นคงค่าเดิม)
  - `404 Not Found`: ไม่พบ ID นักศึกษาที่ระบุ
- **Response Body (200 OK):**
```json
{
  "message": "แก้ไขข้อมูลสำเร็จ",
  "data": {
    "id": 1,
    "name": "สมชาย ใจดี",
    "major": "ปัญญาประดิษฐ์",
    "email": "somchai@example.com",
    "phone": "080-000-0001",
    "courseIds": [101, 102]
  }
}
```

---

### 2.6 DELETE `/api/v1/students/{id}`
- **คำอธิบาย:** ลบข้อมูลนักศึกษาตาม ID
- **Status Codes:**
  - `200 OK`: ลบข้อมูลสำเร็จ
  - `404 Not Found`: ไม่พบ ID นักศึกษาที่ระบุ
- **Response Body (200 OK):**
```json
{
  "message": "ลบข้อมูลสำเร็จ"
}
```

---

## 3. หลักการตั้งชื่อ RESTful API ที่ได้รับการปรับปรุง (RESTful Improvements)

1. **ใช้ Plural Nouns:** กำหนด Resource Path เป็น `/students`
2. **ปรับปรุง Query Parameter:** เปลี่ยนจาก path `/students/:id/full` เป็น `GET /students/:id?include=courses` เพื่อแสดงโครงสร้าง resource และการแนบข้อมูลย่อยอย่างเหมาะสม
3. **ปรับปรุง Status Code สอดคล้องกับสถานการณ์:**
   - ใช้ `201 Created` เมื่อสร้าง resource ใหม่
   - ใช้ `409 Conflict` เมื่อเกิดข้อมูลซ้ำ (เช่น duplicate email)
   - ใช้ `400 Bad Request` เมื่อ validation ล้มเหลว
   - ใช้ `404 Not Found` เมื่อค้นหาข้อมูลไม่พบ
4. **เพิ่ม HTTP PATCH:** รองรับ partial update โดยไม่กระทบกับข้อมูลฟิลด์ที่ไม่ได้ส่งมา
