# Aikchol Staff Portal – ClearPass Guest Web Login

หน้า Login สำหรับพนักงาน (รูปแบบตาม `reference.webp`) ปรับให้ใช้กับ **Aruba ClearPass Guest › Web Login** ได้โดยตรง

| ไฟล์ | ใช้ทำอะไร |
|---|---|
| `header.html` | โค้ดทั้งหมด (HTML + CSS + JS) วางในช่อง **Header HTML** |
| `reference.webp` | ภาพต้นแบบ |

## สิ่งที่เปลี่ยนจากโค้ดต้นฉบับ

- ใส่ CSS/JS ไว้ใน `{literal}…{/literal}` เพราะ ClearPass ใช้ Smarty และจะตีความ `{ }` ใน CSS/JS ผิด (หน้าพังหรือขึ้น template error)
- ฟอร์มเป็นฟอร์มจริงแล้ว: `method="post"` กลับไปที่หน้าเดิม (`{$smarty.server.REQUEST_URI|escape}` เพื่อคง query string ที่ controller ส่งมา เช่น `mac`, `ip`, `url`) ใช้ฟิลด์ `user` / `password` ตามที่ ClearPass ต้องการ
- ถ้ากรอกครบ ฟอร์มจะส่งจริง (ไม่มี `preventDefault` แบบหน้าตัวอย่างแล้ว) และปิดปุ่มไว้กันกดซ้ำ
- เปลี่ยนป้ายเป็น **รหัสพนักงาน / Employee ID** และข้อความ "ใช้รหัสพนักงานและรหัสผ่านองค์กร" ตามภาพ
- แสดง error เมื่อ login ไม่ผ่าน (อ่านจากข้อความของ ClearPass หรือ `?errmsg=` ที่ controller ส่งกลับมา)
- ใส่ prefix `ac-` ให้ทุก class/id เพื่อไม่ให้ชนกับ CSS ของ skin
- path รูป/ฟอนต์ชี้ไปที่ `/public/…` (Content Manager ของ ClearPass)

## ขั้นตอนติดตั้ง

### 1. อัปโหลดไฟล์ประกอบ
**Administration › Content Manager › Public Files › Upload New Content**

| ไฟล์ต้นฉบับ | ตั้งชื่อเป็น |
|---|---|
| `assets/logo.png` | `aikchol-logo.png` |
| `assets/staff-background.png` | `aikchol-staff-bg.png` (แนะนำบีบอัดให้ต่ำกว่า ~500 KB เพื่อโหลดเร็วบนมือถือ) |
| `assets/noto-sans-thai.ttf` | `noto-sans-thai.ttf` (ถ้าอัปโหลดไม่ได้ หน้าจะใช้ Tahoma แทน) |

หลังอัปโหลด ให้คลิกลิงก์ของไฟล์ใน Content Manager เพื่อดู URL จริง ถ้าไม่ใช่ `https://<cppm>/public/<ชื่อไฟล์>` ให้แก้ path `/public/` ใน `header.html` (มี 3 จุด)

### 2. สร้าง/แก้ Web Login
**Configuration › Pages › Web Logins › Create new web login page**

| ส่วน | ค่า |
|---|---|
| Name / Page Name | เช่น `Aikchol Staff` / `aikchol_staff` |
| Vendor Settings | **Aruba Networks** |
| Login Method | **Controller-initiated** (หรือ Server-initiated + CoA ตามที่ออกแบบไว้) |
| IP Address | IP/FQDN ของ controller/gateway ตามใบรับรองของ captive portal (เช่น `securelogin.<domain>`) |
| Secure Login | Use HTTPS |
| Authentication | **Credentials – Require a username and password** |
| **Custom Form** | ✅ **Provide a custom login form** |
| Terms | ไม่ต้องให้ยอมรับ (ในดีไซน์ไม่มี checkbox เงื่อนไข) |
| Skin | **Blank Skin** (หรือ skin ที่ไม่มี header/footer ของตัวเอง) |
| Title | `ระบบสำหรับพนักงาน · โรงพยาบาลเอกชล` |
| **Header HTML** | วางเนื้อหาทั้งหมดของ `header.html` |
| Footer HTML | เว้นว่าง |
| Login Delay | 0–3 วินาที |

กด **Save Changes** แล้วเปิด URL ของหน้า (`https://<cppm>/guest/aikchol_staff.php`) เพื่อตรวจสอบ

### 3. ฝั่ง Controller / Gateway
- Captive portal profile ชี้ Login page ไปที่ `https://<cppm>/guest/aikchol_staff.php`
- Whitelist/walled garden ต้องอนุญาต CPPM (HTTPS 443) สำหรับ role ก่อนยืนยันตัวตน เพื่อให้โหลดรูปและฟอนต์จาก `/public/` ได้
- ClearPass service ที่รับ RADIUS จาก Web Login ต้องใช้ Authentication Source เป็น AD/LDAP ของพนักงาน (รหัสพนักงาน = ชื่อผู้ใช้ใน AD)

## ถ้าส่งฟอร์มแล้วไม่ login

ClearPass แต่ละเวอร์ชันอาจต้องการ hidden field เพิ่ม ให้ตรวจแบบนี้:
1. ปิด **Custom Form** ชั่วคราว บันทึก แล้วเปิดหน้า Web Login
2. คลิกขวา › View Page Source แล้วดู `<form>` ของ ClearPass: จด `action`, ชื่อช่อง user/password และ `<input type="hidden">` ทุกตัว
3. เปิด Custom Form กลับ แล้วเพิ่ม hidden field นั้นใน `<form id="ac-login-form">` ของ `header.html` (หรือปรับ `action`/`name` ให้ตรงกัน)
4. ดูผลที่ **Monitoring › Live Monitoring › Access Tracker** ว่ามี RADIUS request เข้ามาหรือไม่ และเป็น Reject เพราะอะไร
