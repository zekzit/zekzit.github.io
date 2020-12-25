---
layout:	post
title:	"Ways to build with Maven"
date:	2020-12-25
tags: [maven, java, build]
---

ให้เราเริ่มต้นโดยการสร้าง project ที่มีไฟล์ pom.xml ขึ้นมาแบบที่ไม่ได้ใส่ plugins อะไรให้กับ maven เลยตอน build การ build ก็จะมีวิธีต่าง ๆ ดังต่อไปนี้

## การ build ตามปกติ โดยไม่ได้ใส่อะไรเป็นพิเศษเลย

ปกติการ build โดยใช้ maven เราก็ใช้คำสั่ง `mvn package` ธรรมดาใช่ไหมครับ ผลลัพธ์ที่เราได้ก็จะเป็นไฟล์ JAR อยู่ใน folder `target/` แต่ทีนี้เมื่อเราลองเอาไปรันดูแล้ว จะเกิดปัญหาอย่างนี้ครับ

```
PS D:\projects\imed-notification-v2\target> java -jar .\notification-2.0.0-SNAPSHOT.jar
no main manifest attribute, in .\notification-2.0.0-SNAPSHOT.jar
```

### การเพิ่ม manifest file ใน JAR
ปัญหาก็คือตัว JAR ของเราจะไม่มีไฟล์ manifest อยู่ข้างใน ซึ่งเป็นตัว descriptor ให้ java runtime มาอ่านรายละเอียดของ JAR file ครับ ให้เราแก้ไขดังต่อไปนี้

```
...
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>2.6</version>
            <configuration>
                <archive>
                <manifest>
                    <addClasspath>true</addClasspath>
                    <mainClass>com.imed.notification.client.NotificationApplication</mainClass>
                </manifest>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

เมื่อใส่แล้วลอง build ใหม่ เมื่อเข้าไปดูข้างใน เราก็จะเห็นโฟลเดอร์ META-INF และข้อมูลข้างใน พอลองเรียกดู ปรากฏว่าเป็นอย่างนี้ครับ พบว่า project ไม่รู้จักคลาสที่อ้างถึงตามที่ระบุใน classpath

```
PS D:\projects\imed-notification-v2\target> java -jar .\notification-2.0.0-SNAPSHOT.jar
Exception in thread "main" java.lang.NoClassDefFoundError: org/springframework/boot/builder/SpringApplicationBuilder
        at com.imed.notification.client.NotificationApplication.main(NotificationApplication.java:17)
Caused by: java.lang.ClassNotFoundException: org.springframework.boot.builder.SpringApplicationBuilder
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        ... 1 more
```

วิธีการแก้ไขก็มีสองทางนะครับ 1 ก็คือการสร้าง fat-jar หรืออีกทางก็คือนำ library ออกมาใน classpath ให้เรียบร้อย

## การสร้าง Fat JAR

Fat jar ก็คือ JAR ที่ประกอบไปด้วย library เสร็จเรียบร้อยแล้ว แต่มีข้อเสียตรงที่ว่าไฟล์ artifact จะมีขนาดใหญ่ โดยหากต้องมีการ build บ่อย ๆ ก็จะทำให้เปลือง disk, เวลาการส่งข้อมูล, เวลาการสร้าง และอื่น ๆ แต่อย่างไรก็ดีถ้าเป็นโปรเจคที่ build ไม่บ่อย และไฟล์ที่ได้จะกระทัดรัด การสร้าง Fat JAR ก็ยังจำเป็นอยู่

การสร้างเราจะใช้ maven shade plugin โดยสิ่งที่มันทำให้คือการเรียกใช้ ManifestResourceTransformer เพื่อนำ dependencies ที่ระบุไว้ใน manifest มาใส่ไว้ใน JAR ให้เราเอง โดยให้เราเพิ่ม pom.xml ไฟล์ใน build section ตามนี้ครับ

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
                <execution>
                    <id>shaded-jar</id>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer
                                implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>
                                    com.imed.notification.client.NotificationApplication
                                </mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

^ ลองแล้วแต่รันไม่ได้ เอาเป็นว่ามันทำประมาณนี้ ให้ลองไปหาวิธีดูนะครับ

## การสร้าง Fat JAR สำหรับ Spring Boot

เมื่อเราใช้ Spring Boot Starter (เช่น สร้าง Project จาก [Spring Initializr](https://start.spring.io)) การ config ตรงส่วนนี้ก็จะติดมาให้แต่แรกแล้ว จริง ๆ สำหรับโปรเจค Spring Boot เราก็สามารถใช้ maven-shade-plugin ในการ build ได้ แต่ก็มักจะประสบปัญหาเมื่อรันขึ้นมา

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

เมื่อลองกับ project ตัวอย่างแล้ว พบว่าขนาดไฟล์เพิ่มจาก ~600 KB -> 43 MB เลยทีเดียว แต่ก็รันได้เนียนกริบอย่างไร้ที่ติ

## Skinny JAR

อย่างที่บอกไว้ตอนต้นเรื่อง ถ้าเกิดว่าเราต้องมีการ build แล้วต้องอัพโหลดไปที่อื่น เช่น ขึ้น cloud การที่เราจะอัพ Fat JAR ขึ้นไปก็จะเป็นการเปลือง Bandwidth มาก (ดังกล่าวไว้ที่นี่ [The Fault in Our JARs: Why We Stopped Building Fat JARs](https://product.hubspot.com/blog/the-fault-in-our-jars-why-we-stopped-building-fat-jars)) ดังนั้นทีมของ HubSpot จึงได้สร้าง Maven Plugin ชื่อว่า SlimFast แต่น่าเสียดายที่ตอนนี้ตัว Plugin ดังกล่าวทำงานผูกติดอยู่กับ Amazon S3 เพียงเท่านั้น แต่จริง ๆ ก็มีอีกตัวที่น่าศึกษา ไม่ได้เกี่ยวข้องกับ SlimFast หรือ Amazon S3 ก็คือ [Spring Boot Thin Launcher](https://github.com/spring-projects-experimental/spring-boot-thin-launcher)

## การสร้าง WAR file

ถ้าหากเราสร้างเป็น Application ที่เป็น Serverless Cloud หรือ ใส่ไฟล์ลงใน Application Server ต่าง ๆ ก็มี [Maven WAR Plugin](http://maven.apache.org/plugins/maven-war-plugin/) ให้ใช้นะครับ โดยหากเราใช้ Spring Boot เราก็ใช้ Spring Boot Maven Plugin ได้เช่นกัน แต่เราอาจจะต้องระบุว่าเราใช้ Servlet Container ตัวไหน เพื่อให้ตัว Plugin จะได้กำหนดค่าที่เหมาะสมกับ Servlet Container ที่เราใช้ได้

## การเตรียมขึ้น Cloud

บางทีเราก็อาจจะทำเป็น Package ให้ลงได้เลยบน OS ต่าง ๆ ที่อยู่บน Linux เช่น RPM, DEB ได้เลย โดยใช้ Plugin ต่าง ๆ ดังต่อไปนี้
 - [RPM Maven Plugin](http://www.mojohaus.org/rpm-maven-plugin/)
 - [Debian Maven Plugin](http://debian-maven.sourceforge.net/)
 - ตัวติดตั้ง [IzPack](http://izpack.org/), [Launch4j](http://launch4j.sourceforge.net/), [NSIS (Nullsoft Scriptable Install System)](https://nsis.sourceforge.io/Main_Page)
 - สร้างเป็น Machine Image [HashiCorp Packer](https://www.packer.io/), [Netflix aminator](https://github.com/Netflix/aminator)

## การสร้างเป็น Container

การสร้าง Docker image ก็สามารถทำได้ด้วยการสร้าง Dockerfile อย่างทั่วไป หรือจะใช้เครื่องมืออย่าง [fabric8 Docker Maven Plugin](https://github.com/fabric8io/docker-maven-plugin), [Spotify Docker Maven Project](https://github.com/spotify/dockerfile-maven) ก็ได้