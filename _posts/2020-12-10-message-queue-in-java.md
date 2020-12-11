---
layout:	post
title:	"Research about Message Queue in Java"
date:	2020-12-10
tags: [spring, message queue, amqp]
---

# Message Queue ใช้ทำอะไร
ในลักษณะงานของ Web Server เมื่อมี Request มาจาก Client แล้ว Backend ก็จะต้องการ Resource และ Execute งานทันที ซึ่ง Server ต้องใช้เวลา

วิธีแรกคือการใช้ Thread ในการทำงาน Synchronus Execution ซึ่งอันนี้ก็จะธรรมดา ไม่ว่าจะมีเทคนิคแฟนซีเช่น Multi-thread, Multi-Processing ก็ว่าไป แต่การทำอย่างนี้ หากมีงานที่ทำงานนาน หรือเข้ามามากเกินไป ก็จะทำให้ User รอนาน ซึ่งไม่ดีต่อ User Experience ซึ่งอันนี้ก็อาจจะแก้ปัญหาได้โดยการทำ Horizontal Scale โดยที่ไม่จำเป็นต้องพึ่งพาระบบ Queue ก็ได้ 
แต่ถ้าเรารู้ว่ายังไงแต่ละ request ก็จะนานมาก ๆ อยู่ดี ตรงนี้ระบบ queue จะมาช่วยได้ ไม่งั้น request จะมาถมกันแบบ exponentially

การแก้ปัญหานี้คือการใช้ระบบคิว คือ แทนที่ Server จะตอบการ request ให้เลยทันที ตัว Server ก็จะนำ request ใส่ queue แล้วตอบกลับมาด้วย identifier บางอย่าง ที่การทำงานระดับ O(1) ซึ่งอันนี้ก็จะทำงานเร็วอยู่แล้ว ตัว User ก็จะได้รับคำตอบกลับที่เร็วว่า อ๋อ ฉันได้รับงานแล้วนะ กำลังทำงานอยู่ งานต้องใช้เวลา ทีนี้ User ก็จะรอได้อย่างมีความสุขขึ้นกว่ารอด้วยหน้าจอขาว ๆ เป็นเวลานานและทำอย่างอื่นไม่ได้ และตัว client ก็สามารถมา poll ถามได้ว่างานเสร็จรึยังก็ได้

แต่ระบบคิวอย่าง RabbitMQ ก็จะไม่จำเป็นต้องทำอย่างนั้น เพราะก็จะมีระบบ push มาให้อยู่แล้ว โดยเมื่องานดังกล่าวได้ถูก "Execute" หรือ "Dequeue" แล้ว

"ตัว web server ควรทำงานเกี่ยวกับ web traffic, อย่าให้มันต้องมาคิดเลขเลยครับ" - Nasser กล่าวเป็นภาษาตนเอง


https://www.ibm.com/cloud/learn/message-queues

นอกเหนือจากที่กล่าวมาแล้วเรื่องที่ว่าทำให้ response ไปยัง client ทำได้เร็วขึ้น ระบบ message queue ยังทำให้ระบบที่มีความซับซ้อน และไม่รวมศูนย์ทำการแชร์ข้อมูลกันได้ง่ายขึ้น โดยเราต้องมี single messaging backbone ที่ robust และ secure เพื่อทำงานนี้ ข้อดีคือป้องกันการสูญเสียข้อมูลและทำให้ระบบยังทำงานได้แม้ว่าการเชื่อมต่อระหว่างกันไม่ค่อยเสถียรอีกด้วย

# Java + Message Queue

## JMS
JMS API (Java Messaging Service API) เป็น Specification ที่กำหนดแค่ Interface เท่านั้นว่าในการที่จะส่ง Message ระหว่างกัน จะต้องมีการเรียกใช้เป็นอย่างไร แต่ก็ไม่ได้กำหนดถึงชั้น Transport ว่าจะส่งข้อมูลไปด้วยวิธีการใด ดังนั้นเมื่อเราได้ยินคำว่า "ใช้ JMS เป็น Message Queue" เราก็จะต้องดูอีกว่าเราใช้ JMS Provider เจ้าไหน เช่น เราบอกว่าเราใช้ HornetQ เราก็จะต้องใช้ฝั่ง Client เป็น Vendor เดียวกัน เพื่อที่จะให้สื่อสารกันรู้เรื่อง

## AMQP
จากที่เล่ามาข้างบน ก็คิดเอาเองว่านั่นมันคงเป็นปัญหาอยู่เหมือนกันถ้าหากว่าเราใช้ภาษาอื่นเพื่อติดต่อ Message Broker หรือเราก็จะฉีกไปใช้ Message Broker เจ้าอื่นก็ไม่ได้อีก เค้าก็เลยมี AMQP (Advance Message Queue Protocol) ขึ้นมาแก้ไขปัญหานี้ โดยไม่ว่าเราจะเขียนภาษาอะไรก็ตาม เราก็แค่คุยกับ Message Broker ด้วย Protocol นี้ ต่อไปก็ไม่ต้องจำกัดแล้วว่าเราจะใช้ Message Broker เจ้าไหน

## JMS vs. AMQP

## Redis?
- ถึงแม้ว่า Redis จะนำมาทำเป็น Message Broker ได้ แต่เราเลือกจะไม่ใช้ เพราะว่าไม่มีเรื่องการจัดการข้อความที่ดีเท่าระบบ Message Queue อื่น ๆ เราจึงขอข้ามไปดีกว่า

# Java Application with Message Queue implementation

[What is a Message Queue and When should you use Messaging Queue Systems Like RabbitMQ and Kafka - Hussein Nasser](https://www.youtube.com/watch?v=W4_aGb_MOls)

[Which protocol does JMS use to send and receive messages?](https://stackoverflow.com/questions/23882032/which-protocol-does-jms-use-to-send-and-receive-messages)

[Message Queues - IBM Cloud Learn Hub](https://www.ibm.com/cloud/learn/message-queues)