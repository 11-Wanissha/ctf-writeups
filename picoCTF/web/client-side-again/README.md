# Client-side Again

**Category:** Web Exploitation<br>
**Difficulty:** Medium

## Challenge Description

> Can you break into this super secure portal?

**Hint:** What is obfuscation?

---

## Initial Analysis

จาก Hint ของโจทย์ที่กล่าวถึง *obfuscation* จึงคาดว่า flag หรือ logic ที่ใช้ตรวจสอบรหัสผ่านอาจถูกซ่อนอยู่ใน JavaScript ฝั่ง Client

Obfuscation คือการทำให้โค้ดหรือข้อมูลอ่านได้ยากขึ้น แต่ยังสามารถทำงานได้ตามปกติ แตกต่างจาก Encryption ที่ต้องใช้ Key ในการถอดรหัส

เนื่องจากการตรวจสอบรหัสผ่านเกิดขึ้นฝั่ง Client จึงสามารถตรวจสอบ Source Code ได้โดยตรงผ่าน Developer Tools

---

## Investigation

เปิด Developer Tools (`F12`) และตรวจสอบ Source Code ภายในแท็ก `<script>`

<img width="1512" height="822" alt="img001" src="https://github.com/user-attachments/assets/b6072e0b-1845-4669-b81a-4bd9c6249a56" />

พบว่า JavaScript ถูก Obfuscate โดยใช้

* String Array
* Hexadecimal Index
* Variable Renaming

ตัวอย่างเช่น

```javascript
var _0x5a46 = [
  'daf93}',
  '_again_4',
  'this',
  'Password Verified',
  'Incorrect password',
  'getElementById',
  'value',
  'substring',
  'picoCTF{',
  'not_this'
]
```

เพื่อให้อ่านโค้ดได้ง่ายขึ้น จึงนำโค้ดไป Deobfuscate ที่ Website https://deobfuscate.relative.im/

ผลลัพธ์ที่ได้แสดงให้เห็นว่าโปรแกรมใช้การตรวจสอบรหัสผ่านผ่านฟังก์ชัน `substring()`

```javascript
if (checkpass.substring(7, 9) == '{n')
```

---

## Recovering the Flag

เริ่มจากพิจารณาค่าภายใน Array

```javascript
[
  'daf93}',
  '_again_4',
  'this',
  'Password Verified',
  'Incorrect password',
  'getElementById',
  'value',
  'substring',
  'picoCTF{',
  'not_this'
]
```

ค่าหลายตัวเป็นข้อความสำหรับโปรแกรม เช่น

```text
Password Verified
Incorrect password
getElementById
value
substring
this
```

จึงไม่น่าจะเป็นส่วนหนึ่งของ Flag

เหลือข้อมูลที่มีลักษณะคล้าย Flag

```text
picoCTF{
not_this
_again_4
daf93}
```

จากรูปแบบ Flag ของ picoCTF สามารถสันนิษฐานได้ว่า

```text
picoCTF{...daf93}
```

จากเงื่อนไข

```javascript
if (checkpass.substring(7, 9) == '{n')
```

ทำให้ทราบว่าตำแหน่งที่ 7-8 ของข้อความคือ

```text
{n
```

ดังนั้นข้อความหลัง `{` ต้องเริ่มต้นด้วยตัวอักษร `n`

เมื่อพิจารณาข้อมูลที่เหลือ พบว่า

```text
not_this
```

เป็นข้อความเดียวที่ตรงเงื่อนไข

จึงได้รูปแบบ

```text
picoCTF{not_this...
```

เมื่อรวมกับข้อความที่เหลือ

```text
_again_4
daf93}
```

จะได้ Flag สุดท้ายเป็น

```text
picoCTF{not_this_again_4daf93}
```

---

## Lessons Learned

* Obfuscation ไม่ใช่ Encryption
* Logic ที่อยู่ฝั่ง Client สามารถถูกตรวจสอบได้โดยผู้ใช้
* Developer Tools เป็นเครื่องมือสำคัญในการวิเคราะห์ Web Challenges
* การสังเกต String ที่ถูกซ่อนอยู่ในโค้ดสามารถช่วย Recover ข้อมูลสำคัญได้
