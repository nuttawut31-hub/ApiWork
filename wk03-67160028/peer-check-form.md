# แบบฟอร์มการตรวจประเมิน URI Schema (Peer Check Form)


## รหัสนักศึกษา: 67160028 | ชื่อ-นามสกุล: ณัฐวุฒิ รู้การนา



## 1. ตารางแบบฟอร์มตรวจสอบมาตรฐาน RESTful URI Schema (ส่วนที่ 3.1)

| หัวข้อตรวจสอบ | ผ่าน / ไม่ผ่าน | ข้อเสนอแนะ / ข้อสังเกต |
| :--- | :---: | :--- |
| **1. URI ใช้คำนามพหูพจน์ ไม่มีคำกริยาปน** | **ผ่าน** | ใช้ `/api/v1/students` เป็นคำนามพหูพจน์ สื่อถึง collection resource ชัดเจน ไม่มีคำกริยาเช่น `/getStudents` ปะปน |
| **2. ใช้ตัวพิมพ์เล็กและขีดกลาง (kebab-case) ไม่ใช้ camelCase หรือ underscore** | **ผ่าน** | Path ทั้งหมดใช้ตัวพิมพ์เล็ก (lowercase) และแยกคำด้วยขีดกลาง (`-`) ถูกต้องตามมาตรฐาน |
| **3. HTTP Method สอดคล้องกับการดำเนินการ** | **ผ่าน** | ใช้ `GET` สำหรับอ่านข้อมูล, `POST` สำหรับสร้างใหม่, `PUT` สำหรับแก้ไขทั้งระเบียน, `PATCH` สำหรับแก้ไขบางส่วน, และ `DELETE` สำหรับลบข้อมูล |
| **4. Status Code ครอบคลุมทั้งกรณีสำเร็จและกรณีผิดพลาด** | **ผ่าน** | ครอบคลุม `200 OK`, `201 Created`, `400 Bad Request` (Validation Error), `404 Not Found` และ `409 Conflict` (Duplicate Email) |
| **5. ความสัมพันธ์ระหว่าง resource แสดงด้วย nested path หรือ query parameter อย่างเหมาะสม** | **ผ่าน** | ใช้ Query Parameter `?include=courses` เพื่อแนบรายชื่อวิชาได้อย่างเหมาะสม หลีกเลี่ยงการใช้ path ที่ไม่เป็นมาตรฐานเช่น `/full` |

---

## 2. ผลการทดสอบ Endpoint ด้วย Postman และ Edge Cases (ส่วนที่ 3.2)

ได้นำ URI Schema ของโปรเจกต์เพื่อนมาทำการทดสอบจริงด้วย Postman ในกรณี Edge Cases จำนวน 3 Endpoint โดยมีผลการทดสอบดังนี้:

### 🔴 Edge Case 1: เรียกข้อมูลด้วย ID ที่ไม่มีอยู่จริง (GET `/api/v1/students/999`)
* **การดำเนินการ:** ส่งคำขอ `GET http://localhost:3000/api/v1/students/999`
* **ผลลัพธ์ที่คาดหวัง:** HTTP Status Code `404 Not Found` พร้อม Error Message ที่ชัดเจน
* **ผลลัพธ์ที่ได้จากการทดสอบ:**
  - **Status Code:** `404 Not Found`
  - **Response Payload:**
    ```json
    {
      "error": {
        "code": "NOT_FOUND",
        "message": "ไม่พบข้อมูลนักศึกษา"
      }
    }
    ```
* **สรุปผล:** ✅ **ผ่าน (PASS)** - ระบบตอบสนองด้วย Status Code และ Error Code ที่ถูกต้องตามมาตรฐาน RESTful

---

### 🔴 Edge Case 2: ส่งข้อมูลไม่ครบถ้วนในการสร้างนักศึกษาใหม่ (POST `/api/v1/students`)
* **การดำเนินการ:** ส่งคำขอ `POST http://localhost:3000/api/v1/students` โดยส่งเฉพาะ field `"name": "วิชัย ตั้งใจเรียน"` (ขาด `major` และ `email`)
* **ผลลัพธ์ที่คาดหวัง:** HTTP Status Code `400 Bad Request`
* **ผลลัพธ์ที่ได้จากการทดสอบ:**
  - **Status Code:** `400 Bad Request`
  - **Response Payload:**
    ```json
    {
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "กรุณาระบุ name, major และ email ให้ครบถ้วน"
      }
    }
    ```
* **สรุปผล:** ✅ **ผ่าน (PASS)** - ระบบมีการทำ Input Validation และแจ้งข้อผิดพลาด 400 Bad Request เหมาะสม

---

### 🔴 Edge Case 3: สร้างข้อมูลด้วยอีเมลที่ซ้ำกับที่มีอยู่ในระบบ (POST `/api/v1/students`)
* **การดำเนินการ:** ส่งคำขอ `POST http://localhost:3000/api/v1/students` โดยใช้อีเมล `somchai@example.com` ซึ่งมีอยู่ในระบบอยู่แล้ว
* **ผลลัพธ์ที่คาดหวัง:** HTTP Status Code `409 Conflict`
* **ผลลัพธ์ที่ได้จากการทดสอบ:**
  - **Status Code:** `409 Conflict`
  - **Response Payload:**
    ```json
    {
      "error": {
        "code": "DUPLICATE_EMAIL",
        "message": "อีเมลนี้มีอยู่ในระบบแล้ว"
      }
    }
    ```
* **สรุปผล:** ✅ **ผ่าน (PASS)** - ระบบใช้ Status Code 409 Conflict สื่อความหมายถึงข้อมูลซ้ำได้อย่างถูกต้องตามที่ระบุใน wk03.md

---

## 3. สรุปข้อเสนอแนะและการปรับปรุงโปรเจกต์ของตนเอง (ส่วนที่ 3.3)

### ข้อเสนอแนะที่ได้รับจาก Peer Review:
1. **การกำหนด Status Code เมื่อระบุ Body ไม่ครบ:** แนะนำให้ใช้ Status Code `400 Bad Request` ร่วมกับโครงสร้าง Error Object ที่มี `code` และ `message` ชัดเจน
2. **การอัปเดตข้อมูลบางส่วน (PATCH):** ในกรณีที่ใช้ `PATCH` ควรตรวจสอบเงื่อนไข `undefined` ของทุกฟิลด์เพื่อไม่ให้ทับข้อมูลเดิมด้วยค่าว่าง

### การนำข้อเสนอแนะกลับมาปรับปรุงในโปรเจกต์ของตนเอง:
1. ได้ปรับปรุงส่วนของ `POST /api/v1/students` และ `PUT /api/v1/students/:id` ในไฟล์ `index.js` ให้ตรวจสอบความสมบูรณ์ของข้อมูล (`name`, `major`, `email`) อย่างเข้มงวด หากไม่ครบจะตอบกลับ `400 Bad Request` พร้อมรูปแบบ Error JSON
2. ได้ปรับปรุง endpoint `PATCH /api/v1/students/:id` ในไฟล์ `index.js` ให้รองรับการอัปเดตเฉพาะฟิลด์ที่มีการส่งค่าเข้ามา (`!= undefined`) เท่านั้น และอัปเดตไฟล์ [URI-SCHEMA.md](file:///c:/Pre3/student-api/URI-SCHEMA.md) ให้สอดคล้องกับโค้ดจริง
