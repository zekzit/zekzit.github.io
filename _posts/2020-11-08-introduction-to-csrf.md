---
layout:	post
title:	"บทเรียนเกี่ยวกับ CSRF"
date:	2020-11-08
tags: [security, web development]
---

Cross Site Request Forgery (CSRF) คือ การจู่โจมที่ผู้โจมตีหลอกให้ผู้ใช้กระทำการใด ๆ โดยใช้ Identity ของผู้ใช้เว็บไซต์เป้าหมายเพื่อกระทำการอย่างใดอย่างหนึ่งโดยที่ผู้ใช้ไม่รู้ตัวเพื่อให้บรรลุเป้าหมายของผู้โจมตี ถ้าอ่านแล้วไม่ค่อยเข้าใจ ลองคิดภาพตามตัวอย่างนี้ครับ

เช่น สมมติว่าผู้โจมตีมีเว็บไซต์เป้าหมายเป็น (www.someweb.com) มี API ที่ใช้ในการเปลี่ยนพาสเวิร์ด และสมมติว่าผู้ถูกโจมตีเป็นผู้ใช้งานของ www.pwntip.com และผู้โจมตีสร้างลิงค์หลอกให้ผู้ถูกโจมตีลิงค์ที่ผู้โจมตีสร้างขึ้นในเว็บไซต์ www.pwntip.com เช่น URL = http://www.someweb.com/rpc?action=setpassword&u=admin&p=demo และเมื่อผู้ใช้กดลิงค์ดังกล่าว และถ้าผู้ถูกโจมตี Login ค้างเอาไว้ มันก็จะกลายเป็นว่ามันทำในนามของผู้ถูกโจมตี โดยหากว่าผู้โจมตีฟลุ๊คได้ผู้ถูกโจมตีเป็นคนที่มีสิทธิ์ในการเปลี่ยน Password ได้ ผู้โจมตีก็สามารถที่จะเข้าใช้ใน Username และ Password ที่ตั้งไว้ได้ทันที

## คุณลักษณะของ CSRF
 - It involves sites that rely on a user's identity.
 - It exploits the site's trust in that identity.
 - It tricks the user's browser into sending HTTP requests to a target site.
 - It involves HTTP requests that have side effects.

## CSRF และ HTTP Method
 - GET การโจมตีผ่าน HTTP Method GET จะไม่ค่อยส่งผลใด ๆ กับระบบสักเท่าไหร่ เพราะส่วนใหญ่ GET Method จะเป็น Safe Method คือ มักจะเป็นการอ่านข้อมูล และไม่ได้ส่งผลอะไรกับระบบมาก
 - POST 
    + คนร้ายอาจจะสร้าง HTML Form ตามที่ตัวเองต้องการ แล้วหลอกให้ผู้ใช้กด Submit เพื่อส่งค่าได้
    + หรือถ้าเป็น POST ที่รับอย่างอื่น เช่น JSON, XML ก็คือให้ใช้ Same-origin Policy (SOP) หรือ Cross Origin Resource Sharing (CORS) ช่วย หรือการเช็ค MIME Type ว่าถ้าไม่ใช่ application/json, application/xml ก็ไม่รับทำงาน ก็จะช่วยได้
- Method อื่น ๆ เช่น DELETE, PUT ก็ให้ใช้ SOP หรือ CORS มาช่วย เช่นเดียวกัน แต่ถ้าเซ็ต Access-Control-Allow-Origin: * ก็จะไม่ช่วย

## การป้องกันการโจมตีแบบ CSRF
 - Synchronizer token pattern (STP) ให้ Server generate token ใส่ไปใน form เลย
 - Cookie-to-header token ให้ Server generate token ใส่ cookie ไว้ จากนั้นให้ js ส่ง header X-CSRF-Token ด้วยทุก Request โดยจะต้องไม่ใช่ Cookie ที่เป็น httpOnly
 - Double Submit Cookie คล้ายวิธี Cookie-to-header แต่ให้ใส่มาในฟอร์มแทน
 - ลง Extensions ไม่ให้เรียกข้ามเว็บไซต์กันแต่แรก (แต่ยกเว้นได้) เช่น RequestPolicy ของ Firefox หรือ uMatrix สำหรับ Firefox

## Project สำหรับ Test

หากต้องการทดสอบหรือลองเล่นว่าเราจะสามารถโจมตีแบบ CSRF ได้อย่างไร หรือป้องกันการโจมตีชนิดนี้ได้อย่างไร โปรเจคที่น่าสนใจอยู่ตามลิงค์นี้ครับ <https://github.com/pich4ya/csrf-land>