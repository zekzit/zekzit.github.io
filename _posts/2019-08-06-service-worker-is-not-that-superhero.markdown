---
layout:	post
title:	"Service Worker is not that superhero!"
date:	2019-08-06
tags: [web development]
---

  ครั้งนี้เป็นบันทึกความพยายามในการ Hack เล็ก ๆ ในตัว Web Application ตัวหนึ่ง เมื่อเรามีโจทย์ว่าเราต้องการที่จะเปลี่ยนข้อมูลใน Cookie ในทุก ๆ Request ที่เกิดขึ้นให้เป็นไปตามที่เราต้องการ จากเดิมที่มันถูกทำงานอย่างอัตโนมัติโดย Browser ของเรา

![อารมณ์แบบว่าอยากจะดัดช้อนตามใจเราเลยครับ](/assets/images/migrated/0_EaBXD66UNdf-4EJR.jpg)

อารมณ์แบบว่าอยากจะดัด Request headers ตามใจเราต้องการ ภาพนี้ผุดขึ้นมาในหัวเลยครับทีนี้เราก็มามองหา Superhero ของเราครับ Superhero ของเราจะต้องมีพลังพิเศษอยู่ประมาณนี้ครับ

* ดักจับทุก Request ที่ Browser เรียกออกไปได้
* แก้ไข Request ที่กำลังจะบินผ่านหน้าเราไปได้
* สามารถเข้าถึง Storage ที่เรา Map ค่าเอาไว้ได้
* อ่านค่าบางอย่างใน DOM ได้

ด้วยความที่ศึกษามาบ้าง ก็พอจะรู้ว่า Service Worker จะทำหน้าที่เป็นคล้าย ๆ Network Proxy คือเราสามารถจะจับทุก Request ที่จะวิ่งผ่าน Browser ไปได้ที่นี่ (มันจะเกิด FetchEvent ขึ้นทุก ๆ ครั้งที่มีการเรียก Network ให้เราใส่ Listener เข้าไปได้) และนอกจากนั้น เรายังสามารถบิดงอ Request ดังกล่าวได้ด้วย (ใช้ FetchEvent.respondWith() และ Override ด้วยฟังก์ชันที่ Return Response ใหม่ที่เราปรุงแต่งแล้วกลับไป) เจ๋งป่าวล่ะครับ?

Use case ปกติของ Service Worker ในสองความสามารถนี้ ที่เห็นอย่างดาษดื่นคือเค้าจะนำ Cache API เข้ามาร่วมด้วย คือตัว Service Worker ก็จะสกรีนก่อนว่า อ๋อ เรียก URL นี้นะ มีในแคชรึเปล่า ถ้ามีก็เอาในแคชก่อน เราก็จะ Response หน้าเว็บเพจของเรากลับด้วย Response ปลอม ๆ ที่เราสร้างขึ้นมาจากในนี้นี่เองครับ

แต่ทีนี้พอมาเคสของเราคือการ “เปลี่ยนค่า Cookie” กันดีกว่าครับ เราใช้ Service Worker ในการเปลี่ยนค่า Cookie ด้วยโค้ดชุดนี้ครับ คือปกติ Cookie จะถูกส่งผ่าน HTTP Header ที่ชื่อว่า Cookie แล้วก็สุดแต่ใจว่า Server จะเอาข้อมูลตรงนี้ไปทำอะไร แต่ปกติแล้ว fetch() จะไม่ส่ง Cookie ไปด้วย เราจะต้องใส่ “credentials: ‘include’” ไปด้วย

สังเกตนะครับ ว่าเราใส่ Header เข้าไปสองตัว ได้แก่ “Cookie” และ “Cookies” ซึ่งเป็นอันที่เค้าใช้ส่ง Cookie อย่างจริงจัง และอีกอันเป็นคำคล้าย ๆ กันที่ผมใส่เข้าไปเองครับ

![](/assets/images/migrated/1_nXVPGE-0QeE1VwwTDMpNmg.png)

ทีนี้เราก็มาลองกันครับ ก็ตามเงื่อนไข Service Worker ก็คือต้องทำงานใน Trusted environment ซึ่งก็คือทำงานบน https หรือ localhost อะไรก็ตามครับ ซึ่งเราลองเล่นๆ เราก็จะรันอยู่บน localhost ครับ และแน่นอน.. มันทำงานได้แล้ว เย้~~

ผลลัพธ์ที่ได้ก็ออกมาตามรูปข้างล่างนี้นะครับ วิธีทดลองคือเราก็ลองเรียกโค้ด fetch ธรรมดา ๆ ไปยัง API ก็จะเกิด Request แบบในบรรทัดแรกทางซ้าย ซึ่งเมื่อ Request วิ่งไปถึง Service Worker แล้ว Request นี้ก็จะถูกแปลง แล้วก็เรียกอีกครั้งโดย Service Worker (ที่มีฟันเฟืองข้างหน้า) เราก็สามารถกดดู Request Headers ได้ครับ ว่าจริง ๆ แล้ว Browser ส่งอะไรออกไปได้บ้าง

![](/assets/images/migrated/1_8Du8qxy3pFyrnk29_549dg.png)

ผลลัพธ์การแก้ไข Request ที่ได้จาก Service Workerผลปรากฏว่า Header “Cookie” ของเราหายไปครับ ส่วนอีกอันที่ใส่ปลอม ๆ เอาไว้ว่า “Cookies” ยังปรากฏอยู่ อันนี้เคยไปอ่านเจอเอกสารฉบับนึงเค้าบอกว่า Header ที่ชื่อว่า “Set-Cookie” นี่ ตัว Browser จะไม่ให้อ่าน Header ตัวนี้เองได้ครับ แต่สุดท้าย Browser ก็จะจัดเตรียมไว้ให้อยู่ใน document.cookies แทน ซึ่งส่วนตัวผมคิดว่า Header “Cookie” นี่ ก็คงจะถูกส่งไปไม่ได้ด้วยเหตุผลเดียวกันครับ (เพื่อความปลอดภัย?)

เอาเป็นว่าวิธีนี้ก็ไม่เวิร์กไปนะครับ สำหรับความพยายามที่จะใช้ Service Worker เป็น Superhero ของเราในโจทย์นี้… แต่ด้วยความใคร่รู้ เราจึงเหลืออีก 2 ข้อที่ยังสงสัยครับ คือ การเข้าถึง Window Storage และการอ่านค่า DOM

และปรากฏว่าเราไม่สามารถเข้าถึงทั้ง Window Storage ต่าง ๆ และอ่านค่า DOM ได้เลยครับ เหตุผลอาจจะมาจากการที่ Service Worker มันจะรันแยก Process ไปเลย ทำให้ไม่สามารถเข้าถึง API เหล่านี้ได้ และอาจจะเพราะ Security concern ต่าง ๆ ด้วย

เอาเป็นว่าดับฝันกันไปครับ ก็เลยจดไว้เตือนความจำนิดหน่อย เจอกันโอกาสหน้าครับ

**References:**

[https://developers.google.com/web/updates/2015/03/introduction-to-fetch](https://developers.google.com/web/updates/2015/03/introduction-to-fetch)

  