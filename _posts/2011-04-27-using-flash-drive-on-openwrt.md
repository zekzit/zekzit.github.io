---
layout: post
title: ย้ายพื้นที่ใช้สอยบน OpenWRT มาอยู่บน Flash Drive
tags: [openwrt, linux, filesystem, mount]
---

หลังจากลง OpenWRT เสร็จใหม่ ๆ เครื่องก็จะไม่มีโปรแกรมอะไรเลย ไม่สามารถที่จะ Mount Flash Drive ได้ ให้เราลง Package ตามนี้

* kmod-usb-storage

ลงพวกนี้ตามที่ต้องใช้
* kmod-fs-ext3
* kmod-fs-ntfs
* kmod-fs-vfat

แพ็คเกจที่แนะนำข้างบน เป็นแพ็คเกจที่ทำให้เราเตอร์ของเราสามารถที่จะรู้จักอุปกรณ์ usb-storage เช่น flash drive ได้ (ลงแค่อันเดียว เดี๋ยวมันจะหา dependencies ให้เอง ) ส่วนอันถัดมาก็จะเกี่ยวข้องกับระบบไฟล์ เช่น ถ้าเราเอา flash drive ที่มาจาก Windows (ที่ส่วนใหญ่จะฟอร์แมตเป็น FAT32) ก็ให้เราลง kmod-fs-vfat เพื่อให้เราเตอร์สามารถที่จะรู้จักระบบไฟล์นั้น ๆ ได้

ต่อมาก็จะเป็นโปรแกรมที่ใช้ในการจัดการพาร์ติชั่นของดิสก์ ให้ลงแพ็คเกจต่อไปนี้
* fdisk
* e2fsprogs
* swap-utils

จากนั้นก็ให้เราใช้คำสั่ง fdisk จัดแบ่งพาร์ติชันตามที่เราต้องการ โดยเราอาจจะแบ่งเป็นพื้นที่ข้อมูล และพื้นที่ swap เล็กน้อยนะครับ พอแบ่งดิสก์ด้วยคำสั่ง fdisk เสร็จแล้ว ให้เราฟอร์แมตพาร์ติชันที่เป็น ext3 ด้วยคำสั่งนี้นะครับ

```
# issue this so mk2fs doesn't fail
cp /proc/mounts /etc/mtab

# format the linux ext3 partition
mke2fs -j /dev/discs/disc0/part1

# if you're using a drive with existing partitions use umount to unmount things as required.
# now we can mount our new partition
mount -t ext3 /dev/discs/disc0/part1 /mnt
```

ส่วนพาร์ติชันที่เป็น swap ให้ใช้คำสั่งในการเตรียมกับเรียกใช้งานตามนี้

```
# setup some swap space
mkswap /dev/discs/disc0/part2

# activate it - note, it doesn't get reactivated after a reboot, yet.
swapon /dev/discs/disc0/part2
```

ต่อมาก็ให้เราก็อปปี้ไฟล์ที่อยู่บนรอมของเราเตอร์ให้ไปอยู่บน flash drive ด้วยคำสั่งพวกนี้นะครับ

```
# make some folders
mkdir -p /usb /mnt

# copy everything from to the usb device
tar cvO -C / bin/ etc/ lib/ sbin/ usr/ www/ var/ | tar x -C /mnt

# make more folders
mkdir -p /mnt/tmp && mkdir -p /mnt/dev && mkdir -p /mnt/proc && mkdir -p /mnt/jffs

# unmount the usb drive
umount /mnt
```

เมื่อเราเตรียมทุกอย่างเรียบร้อยแล้ว เราก็ต้องแก้ไขสคริปบนเราเตอร์ให้บูตโดยใช้ flash drive ของเรา ตามนี้นะครับ

```
# remove /sbin/init
rm /sbin/init

# replace it with this script:
vi /sbin/init
=== add start ===
#!/bin/sh

# change this to your boot partition
boot_dev="/dev/scsi/host0/bus0/target0/lun0/part1"

# install needed modules for usb and the filesystems
insmod usbcore
insmod uhci && sleep 2s
# insmod ehci-hcd && sleep 2s
insmod scsi_mod && insmod sd_mod && insmod sg && insmod usb-storage
insmod ext2 && insmod jbd && insmod ext3
sleep 2s

# mount the usb stick
mount -t ext3 -o rw "$boot_dev" /usb

# if everything looks ok, do the pivot root
if [ -x /usb/sbin/init ] && [ -d /usb/jffs ]; then
 pivot_root /usb /usb/jffs
 mount none /proc -t proc
 mount none /dev -t devfs
 mount none /tmp -t tmpfs size=50%
 mkdir -p /dev/pts
 mount none /dev/pts -t devpts
 umount /jffs/proc /jffs/dev/pts
 sleep 1s
 umount /jffs/tmp /jffs/dev
fi

# finally, run the real init (from USB hopefully).
exec /bin/busybox init
=== add end ===

# make sure it's executable
chmod a+x /sbin/init

# reboot!
reboot
```

พอเครื่องรีบูทเสร็จ ให้เราพิมพ์คำสั่ง mount ดู ผลลัพธ์ควรจะออกมาเป็นแบบนี้

```
/dev/root on /jffs type jffs2 (rw)
none on /jffs/dev type devfs (rw)
none on /jffs/proc type proc (rw)
none on /jffs/proc/bus/usb type usbfs (rw)
/dev/scsi/host0/bus0/target0/lun0/part1 on / type ext3 (rw)  #<---important line
none on /proc type proc (rw)
none on /dev type devfs (rw)
none on /tmp type tmpfs (rw)
none on /dev/pts type devpts (rw)
none on /proc/bus/usb type usbfs (rw)
```

ทีนี้เราก็เขียนสคริปเพิ่มเติมให้มันเปิด swap เองอัตโนมัติ

```
# now that's ok we can make it so we automatically enable the swap space
vi /etc/init.d/S10boot

=== add this line before first ifconfig line ===
#activate swap
swapon /dev/discs/disc0/part2
=== add end ===
```

เท่านี้ก็เรียบร้อยครับ เราสามารถลงโปรแกรมได้ตามที่ต้องการเลย

อ้างอิงจากนี้นะครับ <http://mpd.wikia.com/wiki/OpenWRT_FullInstall>