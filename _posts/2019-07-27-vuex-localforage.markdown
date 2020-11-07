---
layout:	post
title:	"บันทึกการใช้งาน Vuex + localForage"
date:	2019-07-27
tags: [vue, vuex, localforage]
---

บทความนี้สำหรับ Vue Developer ผู้ซึ่งผ่านการใช้งาน Vue มาในระดับหนึ่งนะครับ ซึ่งถ้าเราใช้งาน Vue มาสักพักแล้ว สิ่งที่เราหลีกเลี่ยงไม่ได้ก็คือการเก็บ state ของ application เรา ซึ่งจริง ๆ ตัว Vue มันก็เป็นตัวจัดการ view เท่านั้น ส่วนตัวที่เป็น state management เราก็สามารถเลือกใช้ได้หลาย ๆ ตัวตามใจเรา แต่สิ่งตัวที่มันค่อนข้างง่ายกับชีวิตเรามากที่สุด ก็คือตัว state management ของ Vue เอง ซึ่งได้แก่ Vuex นี่เอง

![](/assets/images/migrated/0_gSqupDxO1Al_7U5e.png)Vuex (from: <https://vuex.vuejs.org/>) ในบางกรณีที่เราต้องการเก็บค่า state ของ application ของเราด้วยเหตุผลบางประการ เช่น เรื่องของ performance หรือการที่เราเก็บค่าพื้นฐานที่จะใช้ใน application ของเรา หรือเราอยากเก็บ state เพื่อความสะดวกในการ dev เราก็จำเป็นจะต้องมีการ persist state เอาไว้ใน storage ของ browser สิ่งที่แนะนำก็จะเป็น package ชื่อ “vuex-persist” (<https://www.npmjs.com/package/vuex-persist>) ซึ่ง package นี้จะสะดวกมากในการที่จะ persist ค่าลงใน storage ต่าง ๆ ที่ browser รองรับเอาไว้ โดย storage หลัก ๆ ก็จะมีอยู่ตามนี้ครับ

* Cookie เป็นชิ้นข้อมูลเล็ก ๆ ที่เก็บอยู่ใน Browser โดยที่ข้อมูลนี้จะถูกส่งไปที่ server เพื่อเป็น hint ให้ server สามารถทำงานบางอย่างให้เราได้ ขนาดของ cookie ต้องไม่เกิน 4KB
* Window Storage API (Local Storage และ Session Storage) เป็นข้อมูลที่เก็บไว้ในฝั่ง client โดยข้อมูลชุดนี้จะไม่มีการส่งกลับไปที่ server เลย การเข้าถึงข้อมูลจะเข้าถึงผ่าน Javascript ได้เพียงอย่างเดียว โดยความต่างของ Local Storage และ Session Storage จะต่างกันที่ความถาวรของข้อมูล โดย Session Storage จะถูกทำลายเมื่อปิด Tab ไป ในขณะที่ Local Storage จะอยู่คงทนถาวรจนกว่าจะเคลียร์ site data หรือล้าง cache ของ browser ขนาดสูงสุดจะอยู่ที่ประมาณ 5MB
* IndexedDB เป็นความสามารถของ browser ที่จะเก็บข้อมูลที่มีขนาดใหญ่ขึ้น และมีโครงสร้าง โดย API ที่ใช้ในการติดต่อ IndexedDB จะเป็น API แบบ asynchronus เท่านั้น (non-blocking)
โดยส่วนตัวในงานที่ทำ การเริ่มต้นใช้งาน Vuex + vue-persist + Local Storage ก็เป็นอะไรที่แฮปปี้มาก ด้วยสามองค์ประกอบนี้ทำให้การ dev ทำได้ง่ายขึ้น คือเราสามารถ reload page ได้โดยที่ไม่ต้องที่จะต้องเล่นโปรแกรมเพื่อให้เราได้ข้อมูลตามที่เราต้องการตอนเรา dev อยู่ แต่พอผ่านช่วงกระบวนการ dev แล้วเปลี่ยนมาเป็นการทดสอบโปรแกรมกับข้อมูลจริงที่มีขนาดใหญ่ขึ้น เราก็พบกับ error บางอย่างที่ไม่น่าปรารถนาขึ้น

![](/assets/images/migrated/1_tCH9vhsCxBUDFBYkhdhoIw.png)

Storage เกินโควตาหลังจากพยายามใส่ข้อมูลมาก ๆ ซึ่งเป็นฝันร้ายของนักพัฒนาอย่างเราทีนี้ตัวเลือกเดียวที่เหลืออยู่จาก storage ข้างต้นที่ browser provided ให้ ก็จะเป็นตัว IndexedDB ซึ่งจะทำให้ความแฮปปี้ของเราหายไป เนื่องจากว่า API ที่ใช้ในการติดต่อกับ IndexedDB จะมีความซับซ้อนกว่า Window Storage API (แต่เอาเข้าจริงเราก็ให้ตัว vue-persist จัดการอยู่ดี) ทีนี้ localForage ของเราก็จะเข้ามาเป็นพระเอกในงานนี้ครับ

![](/assets/images/migrated/1_eJKWL3otVqhhjAxqzRd_pQ.png)

API ของ Window Storage ก็จะโง่ ๆ ง่าย ๆ แบบนี้localForage เป็นโปรเจคที่ถูกพัฒนาโดย Mozilla ซึ่งหลัก ๆ แล้ว localForage เป็น library ที่ทำให้เราใช้งาน IndexedDB หรือ WebSQL ได้ง่ายขึ้น โดยอารมณ์จะเป็นคล้าย ๆ wrapper ให้เราเรียกใช้ asynchronus storage ได้แบบเดียวกันกับการใช้ Window Storage เลย โดยเราไม่จำเป็นจะต้องจัดการการสร้าง database หรือเขียน query ใด ๆ ให้วุ่นวาย ถ้าหากเราจะใช้งานเป็น key-value ง่าย ๆ

![](/assets/images/migrated/1_gCAnOHwIqzfrFpK2n_od7w.png)

API ของ localForage หน้าตาคล้าย Window Storage API แต่ผิดกันตรงที่จะ return Promise และที่สำคัญคือรองรับ TypeScript ด้วยและด้วยความโชคดี vuex-perist รองรับ localForage ด้วย!

แต่ทีนี้ด้วยธรรมชาติที่ต่างกันของ Window Storage API กับ Asynchronus Storage ก็คือความเป็น Sync/Async API ซึ่งจริง ๆ สมัยที่ใช้ Local Storage นั้น ตัว vuex-persist จะ block การทำงานของการ initialize vue ซึ่งทำให้การ restore state เกิดขึ้นก่อนการที่ Vue application ของเราจะเริ่มทำงาน ทำให้ application ของเราสามารถที่จะทำงานได้โดยมั่นใจได้ว่า state ของ application เราได้กลับมา 100% แล้ว แต่ในกรณีของการใช้งานของ localForage ซึ่งมี API เป็นแบบ asynchronus ทำให้เราไม่สามารถแน่ใจได้ว่า state ของเราจะกลับมาชัวร์ ๆ ก่อนที่ Vue application ของเราจะเริ่มทำงาน

แต่อันที่จริงแล้วมันก็ไม่ได้มีอะไรน่าตื่นเต้นยกเว้น case แปลก ๆ อย่างที่ผมพยายามจะใช้ เพราะเมื่อ state ได้ถูก restore มาใน Vuex เรียบร้อยแล้ว ตัว data ที่เป็น reactive จริง ๆ มันก็จะทำให้ component ต่าง ๆ กลับคืนมาในสภาพเดิมตามที่ state ได้กำหนดเอาไว้อยู่ดี (อันนี้คิดเองนะ เห็นก็ทำงานได้อยู่)

แล้ว case แปลก ๆ ที่ว่าคืออะไรล่ะ? เนื่องจากในงานผมจะมีการใช้งาน Vue Router ร่วมด้วย โดยในกรณีของผมจะมีการอ่านค่าใน state บางอย่าง เพื่อที่จะประเมินว่าจะให้เข้า route ดังกล่าวหรือไม่ ปัญหาก็จะเกิดขึ้นทันทีเนื่องจากว่าเราไม่สามารถที่จะแน่ใจได้เลยว่า state จะพร้อมสำหรับการใช้งานใน Vue Router แล้วหรือยัง การใช้งานดังกล่าวก็จะต้องมีเทคนิคเล็กน้อย

ใน vuex-persist [issue #15](https://github.com/championswimmer/vuex-persist/issues/15) ก็ได้พูดถึงประเด็นนี้ โดยทางออกก็คือเราก็จะต้องหน่วงการทำงานของ Vue Router จนกว่าตัว vuex-persist จะ restore state ได้สำเร็จ สิ่งที่จะต้องทำก็คือการใส่ option strict mode ใหก้ับ vuex-persist โดยเมื่อใส่แล้ว ตัว vuex-persist จะมีการส่ง mutation ที่ชื่อว่า RESTORED\_STATE จากนั้นเค้าก็ให้เรา subscribe store เพื่อหา mutation ตัวนี้ จากนั้นก็ให้ยิง event ออกมา เช่น “storageReady” ผ่านทาง root element ของ Vue จากนั้นในฝั่งของ Vue Router ให้เราไปดักตรง hook ที่เราสนใจในการ protected route ซึ่งได้แก่ router.beforeEach โดยให้เรา hold request เอาไว้ จนกว่าจะเจอ event “storageReady” เท่านี้เราก็สามารถที่จะมั่นใจได้ว่า vuex store ของเรา พร้อมที่จะให้ vue router ทำงานได้แล้ว (เรื่องโค้ดลองตามลิงค์ไปดูนะครับ)

ตอนนี้ก็ขอจบแค่นี้ก่อน ขอให้สนุกกับการ coding ครับ

<https://scotch.io/@PratyushB/local-storage-vs-session-storage-vs-cookie>  
<https://arty.name/localstorage.html>  
<https://github.com/localForage/localForage>

  