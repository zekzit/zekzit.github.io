---
layout:	post
title:	"ทดสอบ ❤️ Ruby on Rails :: Model Test"
date:	2021-01-19
tags: [ruby on rails, test]
---

# อยากมีมาตรฐานในการเขียนโปรแกรมมากขึ้นรึเปล่า

การ Test เป็นอะไรที่ถือได้ว่าเป็นการยกระดับมาตรฐานโปรแกรมที่เราเขียนกันมาเลยทีเดียว ถ้าหากว่าเราเขียนโปรแกรมไก่กา รันเอาสนุก ทำงานได้แล้วก็จบกันไป ไม่น่าจะต้องมีอะไรเพิ่มเติมหรือแก้ไขในภายหลังอีก การ Test ก็จะไม่ค่อยจำเป็นมากนัก แต่สำหรับโปรแกรมที่เราต้องการจะใช้งานไปในระยะยาว (ก็เหมือนกับความรักแหละครับ) เราก็ต้องเขียน Test ควบคู่กันไปด้วย เพื่อทดสอบอยู่เสมอว่าโปรแกรมของเรายังทำงานได้ตามที่ต้องการหรือไม่ครับ

Post นี้ก็ถือเป็น Walkthrough ในวงการ Programming ก็ได้ครับ ทำไปพร้อมกับผมนี่แหละครับ ผมจดไปพลาง ๆ ระหว่างทำด้วย

# Test on Rails 101

สำหรับบทความนี้ ตั้งใจว่าจะทำเป็น Series เป็นขั้นเป็นตอนตามที่ได้ลองจากตัวเอกสารของ Ruby on Rails เองเลย [ที่นี่](https://guides.rubyonrails.org/testing.html) ถ้าไม่ติดอะไรอย่างอื่นที่ทำให้ไม่ได้เขียนซะก่อนนะครับ โดยในวันนี้เราจะมาเริ่มเขียนอะไรที่ไม่น่าจะต้องเทสต์เลย ซึ่งได้แก่ Model

ทำไมไม่น่าเทสต์ล่ะ? ก็ดูเหมือนจะไม่มีอะไรไงครับ โมเดลตามที่เข้าใจมันก็เป็นแค่ POJO เท่านั้น (เอ๊ะ ศัพท์ทาง Java มาได้ไง) แต่จริง ๆ ของ Rails ตัว Model จะมีอะไรมากกว่านั้นครับ อาจจะประกอบไปด้วย Logic เล็ก ๆ น้อย ๆ อยู่ได้บ้าง เอาเป็นว่ามาลองทำตาม Guide ดูแล้วกันนะครับว่าเค้าทำอะไรได้บ้าง

# เริ่มต้นล่ะนะ

## การสร้าง Test File สำหรับ Model

อันที่จริงถ้าหากเราใช้ตัว Generator ของ Rails ในการสร้าง Model อยู่แล้ว ไฟล์ต่าง ๆ ที่เกี่ยวข้องกับการเทสต์จะอยู่ในโฟลเดอร์ `test/` อยู่แล้วครับ โดยมันจะสร้างตัว `test/models/article_test.rb` แล้วก็ไฟล์ `test/fixtures/articles.yml` ซึ่งเป็น Unit Test ของ Model และข้อมูลตัวอย่าง (Fixture) ที่ใช้ในการ Test ครับ

หรือถ้าหากว่าใครสร้างมาแบบห่าม ๆ หน่อย ไม่มีไฟล์เหล่านี้ขึ้นมา ก็สร้างได้ง่าย ๆ ด้วยคำสั่ง `rails g test_unit:model <ชื่อโมเดล>` ได้เลยครับ

## ดูเนื้อหาในไฟล์ Test กัน

```
require "test_helper"

class ArticleTest < ActiveSupport::TestCase
  # test "the truth" do
  #   assert true
  # end
end
```

นี่คือเนื้อหาในไฟล์ Test ที่ Rails Generate ออกมาให้นะครับ (สาระอยู่ตรงไหน? มีแต่ไฟล์เปล่า ๆ) ก็คือมีแต่โครงนะครับ โดยเค้าว่า `require 'test_helper'` คำสั่งนี้ จะทำให้ Rails โหลดค่า default configuration มาให้เรานะครับ

`class ArticleTest < ActiveSupport::TestCase` ส่วนบรรทัดนี้ ก็คือจะทำให้ไฟล์เทสต์ของเรา ได้ความสามารถของ `ActiveSupport::TestCase` ซึ่งก็จะถูกสืบทอดมาอีกทีจาก `Minitest::Test` ไอ้ตัวนี้นี่เองที่ทำให้เราเขียน Test Case ได้สวย ๆ แบบ `test .... do .... end` แทนที่จะต้องเขียน `def .... do .... end` โดยแบบตัวอย่างที่คอมเมนต์ไว้ สรุปแล้วจริง ๆ มันจะสร้างเป็น Function ที่ชื่อว่า `test_the_truth` ให้เราครับ

## ปลดล็อกแล้วรันดู

เราต้องลองเขียนเล่น ๆ ดูสักฟังก์ชันหนึ่งนะครับ โดย Model ของผมมีชื่อว่า `Event` นะครับ โดยเงื่อนไขข้อหนึ่งถูกกำหนดไว้ว่า... (อะไรดี นึกก่อน) เอาเป็นว่า "เหตุการณ์หนึ่งจะต้องประกอบไปด้วยชื่อ" นะครับ เราลองเขียนแบบนี้

```
test 'should not save without a name' do
    event = Event.new
    assert_not event.save
end
```

จากนั้นก็ให้บันทึก แล้วรันด้วยคำสั่งนี้จ้า `rails test test/models/event_test.rb:4` (ฟังก์ชันเราอยู่บรรทัดที่ 4)

```
# Running:

.

Finished in 9.518733s, 0.1051 runs/s, 0.1051 assertions/s.
1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

สุดยอด รันได้ด้วย ไปนอนกันเถอะ.. อ้าว ในเอกสารเค้าบอกว่าให้ต้อง Fail ฉันก็ทำข้อที่ Test ผ่านซะอย่างนั้น เอาใหม่ ลองหาอะไรที่รันแล้วจะ Fail ดู

## จริงจังแล้วล่ะ

เอาเป็นว่าเราจะลองใส่ Fixture (หรือเรียกง่าย ๆ ว่า Sample Data) เป็นตัวที่มีข้อมูลเพียบพร้อมในไฟล์ `test/fixtures/events.yml` ดูนะครับ โดยเราตั้งชื่อ Fixture ตัวนี้ว่า `only_required` ก็คือเป็น Object ที่มีแต่ Required fields นะครับ ซึ่งก็ควรจะเซฟได้ถูกไหม มีฟิลด์ครบแล้ว

```
only_required: 
  name: "Test Event"
  location: "In your heart"
```

เรามาลองใหม่ดูครับ โดยคราวนี้เปลี่ยนเป็นว่าต้องเซฟได้สิ โดยในคราวนี้เราจะอ้างถึง Object ที่เราสร้างในไฟล์ Fixture ด้วย โดยการใช้คำสั่ง `event = events(:only_required)` แล้วเราก็ถอดตัวฟิลด์ `location` ออก โดยในฟังก์ชันนี้เราจะบอกด้วยว่าทำไมไม่ได้ ไม่งั้นมันจะบอกเราแค่ว่าประมาณว่า assertion failed อะไรพวกนี้ ซึ่งไม่มีประโยชน์ โดยโค้ดเป็นดังนี้ครับ

```
test 'should not save without location' do
    event = events(:only_required)
    event.location = nil
    assert_not event.save, "Saved without location"
end
```

ผลลัพธ์การรันที่ได้
```
Running via Spring preloader in process 550
Run options: --seed 3190

# Running:

.
( เทสต์อื่น ๆ ที่ลองเล่นอีก 4 tests)
.


F

Failure:
EventTest#test_should_not_save_without_location [/app/test/models/event_test.rb:7]:
Saved without location


rails test test/models/event_test.rb:4



Finished in 7.162568s, 0.6981 runs/s, 0.6981 assertions/s.
5 runs, 5 assertions, 3 failures, 0 errors, 0 skips
```

ก็พบว่ามันก็แจ้ง Error เราก็อย่าลืมใส่พวก `validates :location, presence: true` อันนี้ก็จะทำให้เราไม่สามารถเซฟได้ครับ เมื่อฟิลด์ที่เราต้องการไม่ครบ

ดูให้ชื่นใจก่อนจากกันไป

```
Running via Spring preloader in process 591
Run options: --seed 29124

# Running:

.....

Finished in 6.812863s, 0.5871 runs/s, 0.5871 assertions/s.
5 runs, 5 assertions, 0 failures, 0 errors, 0 skips
```

สำเร็จล่ะครับ คืนนี้ก็มีเวลาเขียนเท่านี้ ยังไงต่อไปก็จะเขียนต่อและลองอะไรที่ยาก ๆ ขึ้นด้วย แต่ยังไงก็ขอขอบคุณสำหรับการติดตามครับ ถ้าได้อ่านถึงบรรทัดนี้น่ะนะ :D