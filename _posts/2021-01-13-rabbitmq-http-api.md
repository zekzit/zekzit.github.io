---
layout:	post
title:	"RabbitMQ with HTTP Protocol"
date:	2021-01-14
tags: [rabbitmq, message queue]
---

# หลักการและเหตุผล

เมื่อเรามี Legacy Application ที่ไม่สามารถรองรับอะไรใหม่ ๆ เพิ่มเติมได้มากนัก (แต่ก็ยังเพิ่มได้อยู่นะ) การที่เราจะเพิ่มระบบ Messaging เข้าไปมันก็สุดแสนจะลำบาก หากเราต้องการติดต่อกันด้วย Protocol รุ่นใหม่อย่าง AMQP แต่โชคดีที่ RabbitMQ สามารถ[เชื่อมต่อได้หลายทาง](https://www.rabbitmq.com/protocols.html) และในหลายทางเลือกนั้นก็มี HTTP API ที่ค่อนข้างจะเรียบง่ายที่สุด และเราสามารถที่จะเรียกใช้ได้แม้ใน Legacy Application ก็ตาม

# สำรวจ API

สิ่งที่เราต้องการให้กับ Application ของเรา คือ feed event พร้อม parameter บางอย่างเข้าไปใน Message Queue Broker เพื่อให้ตัว Worker ของเราสามารถทำงานตาม Event ที่เกิดขึ้นใน Application นั้น ๆ ได้

แล้ว API ที่เราสามารถเรียกใช้ได้ล่ะ มีอะไรบ้าง? ถ้าหากว่าเราลง RabbitMQ พร้อม Management Console แล้วล่ะก็ เราสามารถเข้าถึง [RabbitMQ Management HTTP API](http://localhost:15672/api/index.html) ได้เลยครับ

# ทดลองกันเถอะ

เมื่อเรามี Topic Exchange ที่เรียกว่า "test-exchange" และมี Queue ที่ชื่อว่า "test-queue" ที่ Binding กันอยู่แล้ว และมี Route ที่ชื่อว่า "foo.bar.#" การส่งข้อความ ก็คือเราจะเรียกเป็นภาษาอังกฤษว่า "Publish" นะครับ ให้เรามองหา API ในการ Publish ครับ โดย API ที่เขียนไว้ เป็นดังนี้ครับ

```
POST /api/exchanges/vhost/name/publish

Publish a message to a given exchange. You will need a body looking something like:

{"properties":{},"routing_key":"my key","payload":"my body","payload_encoding":"string"}

All keys are mandatory. The payload_encoding key should be either "string" (in which case the payload will be taken to be the UTF-8 encoding of the payload field) or "base64" (in which case the payload field is taken to be base64 encoded).
If the message is published successfully, the response will look like:

{"routed": true}

routed will be true if the message was sent to at least one queue.
Please note that the HTTP API is not ideal for high performance publishing; the need to create a new TCP connection for each message published can limit message throughput compared to AMQP or other protocols using long-lived connections.
```

เราก็ลองเรียกดูนะครับ

```
curl --location --request POST 'localhost:15672/api/exchanges/%2F/test-exchange/publish' \
--header 'Authorization: Basic Z3Vlc3Q6Z3Vlc3Q=' \
--header 'Content-Type: application/json' \
--header 'Cookie: JSESSIONID=HSl-p27O58pAWVZe7LndPjg5' \
--data-raw '{"properties":{},"routing_key":"foo.bar.#","payload":"my body","payload_encoding":"string"}'
```

ผลลัพธ์ที่ได้ก็คือ 

```
{"routed": true}
```

ก็แปลว่าสำเร็จนะครับ ฮูเร่ แต่ยังไงก็เป็นทางที่ไม่ค่อยแนะนำนะครับ ใช้โปรโตคอลอื่นเช่น AMQP 0-9-1, STOMP, MQTT เพราะทาง HTTP API แบบนี้จะไม่ค่อยมีประสิทธิภาพนะครับ คืออยากส่งก็เปิดท่อส่งแล้วปิด ในขณะที่ Protocol อื่น ๆ ก็จะเปิดท่อทิ้งไว้แล้วก็ส่งไปส่งกลับได้สองทางเลย ดีกว่าเยอะครับ

ขอบคุณที่ติดตามอ่านครับ