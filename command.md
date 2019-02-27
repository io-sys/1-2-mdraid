
Показать железо
disk -l
lsblk
lshw
lsscsi
lshw -short | grep disk 

Стереть метаданные на блочных устройствах от старых raid
предыдущих raid - ругается, всё хорошо.
mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde
mdadm: Unrecognised md component device - /dev/sdf
mdadm: Unrecognised md component device - /dev/sdg
Создать новый raid
	§ устройство md0 
	§ raid5 уровня -l 5
	§ из 6 дисков -n 6
	§ подсовываем диски /dev/sd{b,c,d,e,f,g}
mdadm --create --verbose /dev/md0 -l 5 -n 6 /dev/sd{b,c,d,e,f,g}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 253952K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
Проверка, raid собрался, все юниты U на месте.
cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdg[6] sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      1269760 blocks super 1.2 level 5, 512k chunk, algorithm 2 [6/6] [UUUUUU]

unused devices: <none>
Проверка, raid собрался, активных устройств 6
mdadm --detail /dev/md0
 /dev/md0:
           Version : 1.2
     Creation Time : Tue Feb 26 22:01:12 2019
        Raid Level : raid5
        Array Size : 1269760 (1240.00 MiB 1300.23 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 6
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Tue Feb 26 22:01:19 2019
             State : clean
    Active Devices : 6
   Working Devices : 6
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 8d599f5e:f124b382:c434b6b6:5564f07d
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf
       6       8       96        5      active sync   /dev/sdg

Созданеи mdadm.conf для ОС, 
	1. какой RAID массив и 
	2. из чего состоит
Показать информацию raid
mdadm --detail --scan --verbose
ARRAY /dev/md0 level=raid5 num-devices=6 metadata=1.2 name=otuslinux:0 UUID=8d599f5e:f124b382:c434b6b6:5564f07d
   devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde,/dev/sdf,/dev/sdg


Создать и заполнить mdadm.conf  - устаревшее, рекомендуется создать.

Показать справку по формату файла mdadm.conf
man mdadm.conf
Создать файл mdadm.conf
touch /etc/mdadm.conf
В mdadm.conf добавить текст "DEVICE partition" для шапки файла конфигурации.
echo "DEVICE partitions" > /etc/mdadm.conf
Добавить >>  в файл mdadm.conf информацию mdraid через пробел.
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf


Поломать и починить raid
Ломаю raid флагом --fail на диске sdf  
после установки флага --fail если перезагрузиться не надо будет делать --remove
mdadm /dev/md0 --fail /dev/sdf
mdadm: set /dev/sdf faulty in /dev/md0

Показать юниты U и _
cat /proc/mdstat
md0 : active raid5 sdg[6] sdf[4](F) sde[3] sdd[2] sdc[1] sdb[0]
      1269760 blocks super 1.2 level 5, 512k chunk, algorithm 2 [6/5] [UUUU_U]

unused devices: <none>
Показать количество дисков в raid и активность дисков.
mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue Feb 26 22:01:12 2019
        Raid Level : raid5
        Array Size : 1269760 (1240.00 MiB 1300.23 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 6
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Wed Feb 27 00:01:46 2019
             State : clean, degraded
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 8d599f5e:f124b382:c434b6b6:5564f07d
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       -       0        0          4      removed
       6       8       96        5      active sync   /dev/sdg

       4       8       80        -      faulty   /dev/sdf



Убрать faulty диск из raid  чтобы новый установить или перезагрузиться после установки флага --fail
mdadm /dev/md0 --remove /dev/sdf
mdadm: added /dev/sdf
Стереть метаданные.
mdadm --zero-superblock --force /dev/sdf
Добавить новый диск.
mdadm /dev/md0 --add /dev/sdf
Проверить.
watch cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdf[7] sdg[6] sde[3] sdd[2] sdc[1] sdb[0]
      1269760 blocks super 1.2 level 5, 512k chunk, algorithm 2 [6/5] [UUUU_U]
      [===================>.]  recovery = 97.5% (248320/253952) finish=0.0min speed=41386K/sec

unused devices: <none>
cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdf[7] sdg[6] sde[3] sdd[2] sdc[1] sdb[0]
      1269760 blocks super 1.2 level 5, 512k chunk, algorithm 2 [6/6] [UUUUUU]

unused devices: <none>
mdadm --detail /dev/md0
    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       7       8       80        4      spare rebuilding   /dev/sdf
       6       8       96        5      active sync   /dev/sdg

lvm и ext4 на неё и в fstab
Создать physical volume
pvcreate /dev/md0
pvs -av
  PV         VG Fmt  Attr PSize PFree DevSize PV UUID

  /dev/md0      lvm2 ---  1.21g 1.21g   1.21g 2rPHZt-56xB-gHiY-0axQ-botM-am4r-DpwyPF
  /dev/sda1          ---     0     0  <40.00g

Создать volume group , создать блочное устройство LVM
vgcreate vg0 /dev/md0
Показать, что получилось.
vgs
  VG  #PV #LV #SN Attr   VSize  VFree
  vg0   1   0   0 wz--n- <1.21g <1.21g
lvcreate -l100%FREE -n lv01 vg0
Показать, что получилось.
lvs
  Logical volume "lv01" created.
Создать файловую систему на lvm , который на raid
mkfs.ext4 /dev/vg0/lv01
lsblk-f

Монтировать raid - lvm - ext4
mkdir /mnt/lv01-data
mount /dev/vg0/lv01 /mnt/lv01-data
Для теста создал filetext записал в него 123456789

Показать id устройства lvm
blkid
UUID=82117052-ba01-4b76-aef8-b7a9fab85226 /mnt/lv01-data ext4
В автозагрузку
vi /etc/fstab
UUID=82117052-ba01-4b76-aef8-b7a9fab85226 /mnt/lv01-data ext4 defaults  0 0

Создать GPT раздел, пять партиций и смонтировать их на диск.

# parted /dev/md0
(parted) print
(parted) mklabel gpt
(parted) print(parted) ?
(parted) mkpart primary ext4 0% 20%(parted) mkpart primary ext4 20% 40%
(parted) mkpart primary ext4 40% 60%
(parted) mkpart primary ext4 60% 80%
(parted) mkpart primary ext4 80% 100%
(parted) print
Model: Linux Software RAID Array (md)
Disk /dev/md0: 1300MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size   File system  Name     Flags
 1      2621kB  260MB   257MB  ext4         primary
 2      260MB   519MB   260MB               primary
 3      519MB   781MB   262MB               primary
 4      781MB   1041MB  260MB               primary
 5      1041MB  1298MB  257MB               primary

$ ls /dev/md0*
/dev/md0  /dev/md0p1  /dev/md0p2  /dev/md0p3  /dev/md0p4  /dev/md0p5

Создать файловую систему на все х созданных патрициях.
# for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
lsblk -f
mkdir -p /mnt/raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /mnt/raid/part$i; done
df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/sda1      xfs        40G  3.0G   38G   8% /
devtmpfs       devtmpfs  110M     0  110M   0% /dev
tmpfs          tmpfs     118M     0  118M   0% /dev/shm
tmpfs          tmpfs     118M  4.5M  114M   4% /run
tmpfs          tmpfs     118M     0  118M   0% /sys/fs/cgroup
tmpfs          tmpfs      24M     0   24M   0% /run/user/1000
/dev/md0p1     ext4      234M  2.1M  215M   1% /mnt/raid/part1
/dev/md0p2     ext4      236M  2.1M  218M   1% /mnt/raid/part2
/dev/md0p3     ext4      239M  2.1M  220M   1% /mnt/raid/part3
/dev/md0p4     ext4      236M  2.1M  218M   1% /mnt/raid/part4
/dev/md0p5     ext4      234M  2.1M  215M   1% /mnt/raid/part5


из методички (pdf) про pated?так можно
Создаем раздел GPT на RAID
parted -s /dev/md0 mklabel gpt
Создаем партиции
parted /dev/md0 mkpart primary ext4 0% 20%
parted /dev/md0 mkpart primary ext4 20% 40%
parted /dev/md0 mkpart primary ext4 40% 60%
parted /dev/md0 mkpart primary ext4 60% 80%
parted /dev/md0 mkpart primary ext4 80% 100%

