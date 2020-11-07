---
layout: post
title: ติดตั้ง Windows 7 จาก Bootable USB
tags: [windows, installation]
---

ขออนุญาตเอามาโน้ตเก็บไว้เป็นข้อมูลส่วนตัวสักเล็กน้อยนะครับ (คนอื่นเอาไปใช้ก็ได้นะ ^^) พอดีไปเจอวิธีทำตัวติดตั้งวินโดวส์บน USB Flash Drive จากเว็บนี้มา <http://www.maximumpc.com/article/howtos/how_to_install_windows_7_beta_a_usb_key>

**สิ่งที่เราต้องมีในการทำนะครับ**
* ไฟล์ Image ของ Windows 7 หรือว่าเป็น DVD ติดตั้งก็ได้
* USB Flash Drive (ขนาดอย่างน้อยก็น่าจะ 4GB นะครับ)

**ขั้นตอนการทำ**

* อย่างแรกเลยก็ต้องทำการฟอร์แมต USB Flash Drive ให้สามารถบูตได้ครับ โดยก่อนอื่น เราก็จะต้องเปิด Command Prompt (โหมด Administrator) จากนั้นให้เราพิมพ์ว่า DISKPART ครับ แล้วพิมพ์คำสั่งตามนี้

```
>LIST DISK << เพื่อดูว่าตัว Flash Drive เราอยู่ที่ไหน
**(ดูตรงนี้ให้แน่ใจนะครับ ไม่งั้นเสี่ยงข้อมูลหายได้ในขั้นตอนต่อ ๆ ไป)**
SELECT DISK # << # คือหมายเลขที่เราดูจากคำสั่งก่อนหน้า ว่า Flash Drive เราอยู่ที่ไหน

CLEAN << ลบทุก Partition ที่อยู่บน Flash Drive

CREATE PARTITION PRIMARY << เป็นการสร้าง Partition ใหม่ที่เป็น Primary

SELECT PARTITION 1 << เลือก Partition ที่เพิ่งสร้างขึ้นมาใหม่

ACTIVE << เซ็ตว่าให้ Partition นี้บูตได้

FORMAT FS=NTFS << ฟอร์แมตให้อยู่ในรูปแบบของ NTFS

ASSIGN << ให้ Windows กำหนดตัวอักษรให้กับไดรว์ที่เราเพิ่งสร้างขึ้นใหม่

EXIT << เพื่อออกจากโปรแกรม
```


* ทำให้ Flash Drive มันบูตได้ โดยเข้าไปที่โฟลเดอร์ boot จาก ISO หรือแผ่น DVD ของ Windows 7 แล้วพิมพ์คำสั่งว่า bootsect /nt60 X: (x: คือไดรว์ที่เป็นของ Flash Drive เรา ลองตรวจสอบให้แน่ใจว่ามันเปลี่ยนไปจากขั้นตอนแรกรึเปล่า เพราะว่ามันอาจจะเปลี่ยนได้) แล้วทีนี้ Flash Drive เราก็จะบูตได้แล้ว
* ก๊อปปี้ไฟล์จาก DVD มาลงใน Flash Drive ให้หมด
* บูตเครื่องจาก USB

ถ้าอยากเห็นภาพ Capture หน้าจอ ก็ลองไปดูในเว็บต้นฉบับนะครับ เค้าทำไว้ดีกว่ามาก