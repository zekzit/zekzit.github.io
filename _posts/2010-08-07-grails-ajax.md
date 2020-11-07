---
layout: post
title: สรุปคร่าว ๆ Grails + AJAX
tags: [ajax, grails, web development]
---

วันนี้ได้ไปทดลองทำตาม Tutorial ของ IBM ที่่ชื่อว่า 
[Mastering Grails: Many-to-many relationships with a dollop of Ajax](https://www.ibm.com/developerworks/java/library/j-grails04158/) มา หลังจากที่ได้ทำ ก็ไม่ได้จะเขียนโค้ด Grails + AJAX ได้เองทันทีหรอก แต่ว่ามีข้อสรุปเรื่องการทำงานของ AJAX คร่าว ๆ ตามนี้

* เตรียมข้อมูลฝั่ง Server ให้อยู่ในรูปแบบ XML หรือ JSON ในตัว Controller ของ Model ที่เราสนใจ ให้เราใส่ฟังก์ชัน getXml หรือ getJson เข้าไปตามนี้

```
def getXml = {
    render Airport.findByIata(params.iata) as XML
}

def getJson = {
    def airport = Airport.findByIata(params.iata)
    if(!airport) {
        airport = new Airport(iata:params.iata, name:"Not found")
    }
    render airport as JSON
}
```

จากนั้นให้เราทดลองเรียกข้อมูลออกมาผ่าน URL นี้ http://localhost:8080/TripPlanner/airport/getXml?iata=HDY กับ http://localhost:8080/TripPlanner/airport/getJson?iata=HDY ถ้ามีข้อมูลออกมาในรูปแบบ XML หรือ JSON ตามที่ได้ทำเอาไว้ ก็ถือว่าโอเคครับ

* เขียน Javascript ไว้ที่ฝั่งของ Client (หรือในหน้าเว็บเพจที่เราจะใช้งานมันนั่นเอง) ตามตัวอย่างมันจะแก้ไขในไฟล์ views/flight/create.gsp ให้เราใส่ Javascript อันนี้ลงไปนะครับ

```
<g:javascript library="prototype" />
<script type="text/javascript">

function get(airportField) {
    var baseUrl = "${createLink(controller:'airport', action:'getJson')}"
    var url = baseUrl + "?iata=" + $F(airportField + "Iata")
    new Ajax.Request(url, {
        method: 'get',
        asynchronous: true,
        onSuccess: function(req) { update(req.responseText, airportField) }
    })
}

function update(json, airportField) {
    var airport = eval( "(" + json + ")" )
    var output = $(airportField + "Text")
    output.innerHTML = airport.iata + " - " + airport.name
    var hiddenField = $(airportField + ".id")
    airport.id == null ? hiddenField.value = -1 : hiddenField.value = airport.id
}
</script>
```

สคริปตัวนี้ บรรทัดแรกจะเป็นการโหลด Javascript Library ที่ชื่อว่า Prototype ขึ้นมา (เป็น Library ที่สามารถเรียกใช้งาน AJAX ได้ในหลาย ๆ Browser แล้วไม่ค่อยมีปัญหา)

ฟังก์ชัน get() ก็จะเอาไว้ใช้ในการส่ง Request ไปที่ทาง Server (เรียกไป URL ตามที่เราทำในข้อ 1) โดยจะเรียกข้อมูลที่เป็น JSON มาเพราะว่า Javascript สามารถจัดการมันได้ง่ายกว่า สุดท้ายก็จะเป็นการบอกว่า ถ้าได้รับข้อมูลกลับมาแล้ว ให้เรียกฟังก์ชัน update() มาทำงาน (เป็นการกำหนด callback) นั่นเอง

ฟังก์ชัน update() จะทำการแก้ไขหน้าเว็บเพจในส่วนที่เราต้องการ (เอาข้อมูลที่ได้จาก Server มายัดลงไปในกล่องข้อความ ประมาณนี้)

* สุดท้ายก็จะเป็นการเรียกใช้จากตัวกล่องข้อความ โดยเมื่อเราป้อนข้อความไปนิดนึง แล้วก็ Find ไอ้ปุ่ม Find มันก็จะวิ่งมาเรียกฟังก์ชัน get ให้ทำงาน แล้วก็มาอัพเดตค่าอีกทีหนึ่ง การใช้งานก็ตามนี้ครับ

```
<div id="arrivalAirportText">[Type an Airport IATA Code]</div>
<input type="hidden" name="arrivalAirport.id" value="-1" id="arrivalAirport.id"/>
<input type="text" name="arrivalAirportIata" id="arrivalAirportIata"/>
<input type="button" value="Find" onClick="get('arrivalAirport')"/>
```

จากโค้ด จะเห็นว่าเวลากดปุ่มมันจะส่งชื่อของ 'arrivalAirport' ติดไปด้วย ทำให้มันรู้ว่าควรจะ update ลงไปในช่องอันไหน

อันนี้จดเอาไว้เป็นความรู้เบื้องต้น เอาไว้เวลาจะใช้ก็เอาพวกนี้เป็นหลักการพื้นฐาน แล้วก็พลิกแพลงไปใช้งานต่อไปครับ