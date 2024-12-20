Firebase Authentication สามารถเพิ่มการยืนยันตัวตนผ่าน **LINE** ได้ แต่ต้องทำผ่าน **Custom Authentication** เนื่องจาก Firebase Authentication ไม่มีการรองรับ LINE OAuth โดยตรงเหมือนกับ Google, Facebook หรือ Apple 

ขั้นตอนการเพิ่ม LINE Authentication มีดังนี้:

1. **ตั้งค่า LINE Login ใน LINE Developers Console**  
   - ไปที่ [LINE Developers Console](https://developers.line.biz/)  
   - สร้าง Channel ใหม่ และเปิดใช้งาน LINE Login  
   - กำหนด Redirect URI ให้ตรงกับ URL ที่ Firebase ใช้ เช่น `https://your-app.firebaseapp.com/__/auth/handler`

2. **รับ Access Token จาก LINE**  
   - เมื่อผู้ใช้ล็อกอินผ่าน LINE สำเร็จ คุณจะได้รับ `access_token` และ `id_token` 

3. **ตรวจสอบและสร้าง Custom Token บน Firebase**  
   - ใช้ Firebase Admin SDK เพื่อสร้าง **Custom Token** สำหรับผู้ใช้ โดยตรวจสอบข้อมูลจาก `id_token` ของ LINE  
   - ตัวอย่างโค้ด Node.js:
     ```javascript
     const admin = require("firebase-admin");
     const createFirebaseToken = async (lineUserId) => {
       const firebaseToken = await admin.auth().createCustomToken(lineUserId);
       return firebaseToken;
     };
     ```

4. **ใช้ Custom Token เพื่อ Login บน Client-Side**  
   - ส่ง `custom_token` ที่สร้างขึ้นให้กับ Frontend และใช้ Firebase Client SDK เพื่อเข้าสู่ระบบ:  
     ```javascript
     firebase.auth().signInWithCustomToken(customToken)
       .then((userCredential) => {
         console.log("User signed in:", userCredential.user);
       })
       .catch((error) => {
         console.error("Error signing in:", error);
       });
     ```

5. **เก็บข้อมูลผู้ใช้ใน Firebase Database หรือ Firestore**  
   - หลังจากผู้ใช้ล็อกอินสำเร็จ คุณสามารถเก็บข้อมูลเพิ่มเติมของผู้ใช้ เช่น ชื่อ, อีเมล หรือโปรไฟล์ LINE ลงใน Firestore ได้  

หากคุณต้องการความช่วยเหลือในการตั้งค่าหรือเขียนโค้ดส่วนไหนเพิ่มเติม บอกได้เลยครับ! 😊