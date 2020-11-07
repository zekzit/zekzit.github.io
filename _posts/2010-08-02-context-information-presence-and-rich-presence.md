---
layout: post
title: Context Information, Presence and Rich Presence
tags: [communication, context information, context-aware, presence, rich presence]
---

ตอนทำงานวิจัยก็มีความสงสัยว่า Context Information กับ Presence มีความแตกต่างกันยังไง ถ้าเกิดว่าเรามาใช้ในฟิลด์ของการสื่อสาร หลังจากได้ไปศึกษามาก็พบความแตกต่างดังนี้ครับ

ก่อนอื่นเลย เรามาดูก่อนว่า Presence คืออะไร Presence มันเกิดขึ้นมาพร้อมกับ Instant Messaging อย่างพวก MSN, Jabber อะไรพวกนี้นะครับ มันเอาไว้แค่บอกสถานะ Online, Busy, Offline เท่านั้น

ดังนั้นความหมายของ Presence แต่แรก = Willingness of being contacted ครับ

แต่ต่อมาได้มีมาตรฐานซึ่งออกมารองรับความสามารถเพิ่มเติมของ Presence กล่าวคือ เป็น Presence แบบใหม่ ที่บอกอะไรได้หลาย ๆ อย่างมากกว่าเดิม เราเรียก Presence แบบใหม่นี้ว่า Rich Presence ครับ

สรุปคือ Rich Presence = Emotion + Activity + Willingness of being contacted + อื่น ๆ อีกมากมาย ที่เกี่ยวข้องกับตัวบุคคลนั้น ๆ นะครับ (ดูได้ใน RFC4480)

แล้วทีนี้ Context Information ล่ะ? มันมีความเหมือนกับความต่างกับ Rich Presence ยังไง?

ก่อนอื่นก็ต้องมาดูความหมายของ Context Information ก่อน มันคือข้อมูลที่อธิบายถึงสภาพแวดล้อมรอบ ๆ ตัวของ Entity ที่เราสนใจครับ

ทีนี้จากความเห็นส่วนตัวนะครับ ความต่างก็คือว่า Context Information จะมีความหมายที่กว้างกว่า ผิดกับ Rich Presence ที่มันจะระบุข้อมูลของ Presentity เท่านั้น ซึ่ง Presentity อาจจะเป็นได้แค่ตัวบุคคล บริการ หรืออุปกรณ์เท่านั้นครับ