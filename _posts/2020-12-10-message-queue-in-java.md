---
layout:	post
title:	"มาดูเรื่อง Message Queue + การใช้งานใน Java กันเถอะ"
date:	2020-12-10
tags: [spring, message queue, amqp]
---

เนื่องจากชีวิตเราก็ต้องก้าวไปข้างหน้าเรื่อย ๆ การเจอโจทย์ยากและท้าทายขึ้น เราก็ต้องหาวิถีทางที่มาแก้ปัญหาที่มันเปลี่ยนไปเช่นกัน และวันนี้โจทย์ที่เข้ามาก็คือการที่เราจะทำอย่างไรให้คำสั่งยังอยู่แม้ว่า Service นั้นจะ down ลงไปสักแป๊ป พอตื่นขึ้นมาเราก็ควรจะรับการทำงานต่อได้ ไม่ใช่ว่าให้คำสั่งนั้นหายไปกับสายลมและตามต่อไม่ได้ สิ่งนี้จึงเป็น Solution ให้กับโจทย์ในเรื่องนี้

# ทำไมต้อง Message Queue

![](/assets/images/queue-number.jpg)

ในลักษณะงานของการทำเว็บหลังบ้าน เมื่อมี request มาจาก client แล้วปกติหลังบ้านก็จะต้องการทรัพยากร และ execute request ที่เข้ามาทันที ซึ่งเป็นงานแบบ synchronus คือ request ที่เข้ามาจะต้องรอ server ทำงานให้เสร็จก่อนที่จะตอบกลับไป ตรงนี้ก็จะมีเทคนิคที่ทำให้เร็วขึ้น ไม่ว่าจะเป็นการทำ multi-threading หรือ multi-processing ก็ว่าไป 

แต่การทำอย่างนี้ หากมีงานที่ทำงานนาน หรือเข้ามามากเกินไป ก็จะทำให้ user รอนาน ซึ่งผู้ใช้ก็จะหงุดหงิด ซึ่งอันนี้ก็อาจจะแก้ปัญหาได้โดยการทำ horizontal scale เพิ่มเติมโดยที่ไม่จำเป็นต้องพึ่งพาระบบ queue ก่อนก็ได้ แต่ถ้าเรารู้ว่ายังไงแต่ละ request ก็จะนานมาก ๆ อยู่ดี ตรงนี้ระบบ queue จะมาช่วยได้ ไม่งั้น request จะมาถมกันแบบ exponentially ซึ่งจะพาให้ระบบล่มไปในที่สุด

การแก้ปัญหานี้คือการใช้ระบบคิวเข้ามาช่วย คือแทนที่ server จะตอบการ request ให้เลยทันที ตัว server ก็จะนำ request ใส่ queue แล้วตอบกลับมาด้วย identifier บางอย่าง ที่การทำงานระดับ O(1) ซึ่งจะทำให้ user ก็จะได้รับคำตอบกลับที่เร็วว่า "อ๋อ server ได้รับคำสั่งของฉันแล้วนะ กำลังทำงานอยู่ งานต้องใช้เวลา" ทีนี้ user ก็จะไม่ได้กังวลมากว่างานที่สั่งไปสั่งสำเร็จแล้วรึยัง 

และเมื่อตัว client อยากรู้ว่างานเสร็จรึยังก็สามารถมา poll ถามได้ว่างานเสร็จรึยังก็ได้ แต่ระบบคิวอย่าง RabbitMQ ก็จะไม่จำเป็นต้องทำอย่างนั้น เพราะก็จะมีระบบ push มาให้อยู่แล้ว โดยเมื่องานดังกล่าวได้ถูก "Execute" หรือ "Dequeue" แล้ว เราก็ไม่ต้องมานั่งดึงอยู่

```
"ตัว web server ควรทำงานเกี่ยวกับ web traffic, อย่าให้มันต้องมาคิดคำนวณมากเลยครับ" - Nasser กล่าวเป็นภาษาตนเอง พี่หมีมาแปลให้
```

นอกเหนือจากที่กล่าวมาแล้วเรื่องที่ว่าทำให้ response ไปยัง client ทำได้ดีขึ้นโดยการรับงานมาทำแล้วแจ้งเตือนไปยัง client แล้ว ระบบ message queue ยังช่วยทำให้ระบบที่มีความซับซ้อน และไม่รวมศูนย์ทำการแชร์ข้อมูลกันได้ง่ายขึ้น โดยที่เราต้องมี single messaging backbone ที่ robust และ secure เพื่อทำงานนี้ ข้อดีคือป้องกันการสูญเสียข้อมูลและทำให้ระบบยังทำงานได้แม้ว่าการเชื่อมต่อระหว่างกันไม่ค่อยเสถียรอีกด้วย *<< ซึ่งเราสนใจจะใช้ตรงนี้แหละ*

# Java + Message Queue

เมื่อเราเห็นถึงข้อดี(ที่ไม่ได้มองหาข้อเสียเล๊ย)แล้ว ทีนี้เราก็ลองมาพยายาม make it happens ใน application ของเรากันเถอะครับ ซึ่งวันนี้จะมาโฟกัสกันที่ application ที่พัฒนากันในโลกของภาษา Java นะครับ

ตัวที่คิดว่าจะนำมาพัฒนาก็จะใช้เฟรมเวิร์ก Spring ซึ่งหลัก ๆ แล้วการทำ Messaging จะทำได้หลัก ๆ ที่หามาอยู่ประมาณ 3 วิธี คือการใช้ Redis, JMS แล้วก็ใช้ AMQP ครับ

## Java Messaging Service API (JMS)
JMS API (Java Messaging Service API) เป็น Specification ที่กำหนดแค่ Interface เท่านั้นว่าในการที่จะส่ง Message ระหว่างกัน จะต้องมีการเรียกใช้เป็นอย่างไร แต่ก็ไม่ได้กำหนดถึงชั้น Transport ว่าจะส่งข้อมูลไปด้วยวิธีการใด ดังนั้นเมื่อเราได้ยินคำว่า "ใช้ JMS เป็น Message Queue" เราก็จะต้องดูอีกว่าเราใช้ JMS Provider เจ้าไหน เช่น เราบอกว่าเราใช้ HornetQ เราก็จะต้องใช้ฝั่ง Client เป็น Vendor เดียวกัน เพื่อที่จะให้สื่อสารกันรู้เรื่อง

## Advance Message Queue Protocol (AMQP)
จากที่เล่ามาข้างบน ก็คิดเอาเองว่านั่นมันคงเป็นปัญหาอยู่เหมือนกันถ้าหากว่าเราใช้ภาษาอื่นเพื่อติดต่อ Message Broker หรือเราก็จะฉีกไปใช้ Message Broker เจ้าอื่นก็ไม่ได้อีก เค้าก็เลยมี AMQP (Advance Message Queue Protocol) ขึ้นมาแก้ไขปัญหานี้ โดยไม่ว่าเราจะเขียนภาษาอะไรก็ตาม เราก็แค่คุยกับ Message Broker ด้วย Protocol นี้ ต่อไปก็ไม่ต้องจำกัดแล้วว่าเราจะใช้ Message Broker เจ้าไหน

## JMS vs. AMQP
ก็อย่างที่อธิบายเรื่องของ JMS กับ AMQP นะครับ ดูแล้วการเลือกใช้ AMQP จะเป็นทางเลือกที่ดีกว่าในแง่ที่เราไม่จำเป็นต้องผูกติดกันเรื่องภาษาที่ใช้เขียน และ Vendor ที่พัฒนาระบบคิวมากจนเกินไปครับ

## Redis?
ถึงแม้ว่า Redis จะนำมาทำเป็น Message Broker ได้ แต่เราเลือกจะไม่ใช้ เพราะว่าไม่มีเรื่องการจัดการข้อความที่ดีเท่าระบบ Message Queue อื่น ๆ เราจึงขอข้ามไป และไม่สนใจที่จะนำมาใช้ครับ

# Java Application with Message Queue implementation

ตรงนี้ก็ยังไม่ได้ศึกษามาก แต่ก็ขอแปะลิงค์เพื่อไว้ให้ศึกษาต่อนะครับ ในกรณีที่ใช้ Spring Framework ทางฝั่งของ Java หลังจากที่ทดลองกับบทความ *Messaging with RabbitMQ* แล้ว (+ศึกษาเพิ่มจาก *Spring AMQP*) ก็ถือว่าทำตามได้จริง และพอจะเข้าใจได้บ้างครับ (ส่วนตัวเพิ่งมาลองจับ Spring Framework เลยจะงง ๆ นิดหน่อยเพราะตัว Framework ซ่อนรายละเอียดเยอะอยู่)

- [Spring AMQP](https://docs.spring.io/spring-amqp/reference/html/)
- [Messaging with JMS](https://spring.io/guides/gs/messaging-jms/)
- [Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)

# References

- [JMS vs AMQP](https://www.linkedin.com/pulse/jms-vs-amqp-eran-shaham/)

- [What is a Message Queue and When should you use Messaging Queue Systems Like RabbitMQ and Kafka - Hussein Nasser](https://www.youtube.com/watch?v=W4_aGb_MOls)

- [Which protocol does JMS use to send and receive messages?](https://stackoverflow.com/questions/23882032/which-protocol-does-jms-use-to-send-and-receive-messages)

- [Message Queues - IBM Cloud Learn Hub](https://www.ibm.com/cloud/learn/message-queues)