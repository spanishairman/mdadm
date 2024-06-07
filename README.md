### Дисковая подсистема. mdadm
#### Подготовка окружения
В нашем примере используется гипервизор Qemu-KVM, библиотека Libvirt. В качестве хостовой системы - OpenSuse Leap 15.5.

Для работы Vagrant с Libvirt установлен пакет vagrant-libvirt:
```
Сведения — пакет vagrant-libvirt:
---------------------------------
Репозиторий            : Основной репозиторий
Имя                    : vagrant-libvirt
Версия                 : 0.10.2-bp155.1.19
Архитектура            : x86_64
Поставщик              : openSUSE
Размер после установки : 658,3 KiB
Установлено            : Да
Состояние              : актуален
Пакет с исходным кодом : vagrant-libvirt-0.10.2-bp155.1.19.src
Адрес источника        : https://github.com/vagrant-libvirt/vagrant-libvirt
Заключение             : Провайдер Vagrant для libvirt
Описание               : 

    This is a Vagrant plugin that adds a Libvirt provider to Vagrant, allowing
    Vagrant to control and provision machines via the Libvirt toolkit.
```
Пакет Vagrant также устанавливаем из репозиториев. Текущая версия для OpenSuse Leap 15.5:
```
max@localhost:~/vagrant/vg3> vagrant -v
Vagrant 2.2.18
```
Образ операционной системы создан заранее, для этого установлен [Debian Linux из официального образа netinst](https://www.debian.org/distrib/netinst)

#### Особенности работы Vagrant, установленного из официального репозитория OpenSuse Leap 15.5, с дополнительными дисковыми устройствами в гостевой ОС
Для добавления дисков в гостевой системе, используем следующий код в [Vagrantfile](Vagrantfile):
```
  config.vm.provider "libvirt" do |lv|
    lv.memory = "2048"
    lv.cpus = "2"
    lv.title = "Debian12"
    lv.description = "Виртуальная машина на базе дистрибутива Debian Linux"
    lv.management_network_name = "vagrant-libvirt-mgmt"
    lv.management_network_address = "192.168.121.0/24"
    lv.management_network_keep = "true"
    lv.management_network_mac = "52:54:00:27:28:83"
    lv.storage :file, :size => '1G', :device => 'vdc', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdd', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vde', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdf', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdg', :allow_existing => false
  end
```
Здесь мы добавляем в нашу гостевую систему пять дисковых устройств, являющихся файлами в системе хоста, это vdc, vdd, vde, vdf и vdg. В случае, если не задавать явно имена устройств
то Vagrant сам назначит их по латинскому алфавиту с префиксом vd, начиная с первого свободного символа. Например, если в гостевой системе после установки используется устройство
/dev/vda, то первое дополнительное дисковое устройство будет /dev/vdb. 

Напомню, в нашем случае, дисковые устройства дожны занять такие имена vd{c,d,e,f,g}. Однако, после выполнения команды vagrant up и входа по ssh  в остемую систему, 
мы видим следующую картину:
```
vagrant@debian12:~$ sudo -i
root@debian12:~# lsblk 
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda                     254:0    0   40G  0 disk 
├─vda1                  254:1    0  953M  0 part /boot
├─vda2                  254:2    0    1K  0 part 
└─vda5                  254:5    0 39,1G  0 part 
  ├─vg--system-lvolroot 253:0    0 18,6G  0 lvm  /
  ├─vg--system-lvolswap 253:1    0  3,7G  0 lvm  [SWAP]
  └─vg--system-lvolhome 253:2    0 16,7G  0 lvm  /home
vdb                     254:16   0    1G  0 disk 
vdc                     254:32   0    1G  0 disk 
vdd                     254:48   0    1G  0 disk 
vde                     254:64   0    1G  0 disk 
vdf                     254:80   0    1G  0 disk
```
При этом, во время разворачивания гостевой машины, в консоли Vagrant были строки с "правильными" устройствами:
```
==> Debian12: Creating image (snapshot of base box volume).
==> Debian12: Creating domain with the following settings...
==> Debian12:  -- Name:              vg3_Debian12
==> Debian12:  -- Title:             Debian12
==> Debian12:  -- Description:       Виртуальная машина на базе дистрибутива Debian Linux
==> Debian12:  -- Domain type:       kvm
==> Debian12:  -- Cpus:              2
==> Debian12:  -- Feature:           acpi
==> Debian12:  -- Feature:           apic
==> Debian12:  -- Feature:           pae
==> Debian12:  -- Clock offset:      utc
==> Debian12:  -- Memory:            2048M
==> Debian12:  -- Management MAC:    52:54:00:27:28:83
==> Debian12:  -- Loader:            
==> Debian12:  -- Nvram:             
==> Debian12:  -- Base box:          /home/max/vagrant/images/debian12
==> Debian12:  -- Storage pool:      images
==> Debian12:  -- Image(vda):        /home/max/libvirt/images/vg3_Debian12.img, virtio, 40G
==> Debian12:  -- Disk driver opts:  cache='default'
==> Debian12:  -- Kernel:            
==> Debian12:  -- Initrd:            
==> Debian12:  -- Graphics Type:     vnc
==> Debian12:  -- Graphics Port:     -1
==> Debian12:  -- Graphics IP:       127.0.0.1
==> Debian12:  -- Graphics Password: Not defined
==> Debian12:  -- Video Type:        cirrus
==> Debian12:  -- Video VRAM:        16384
==> Debian12:  -- Video 3D accel:    false
==> Debian12:  -- Sound Type:        
==> Debian12:  -- Keymap:            en-us
==> Debian12:  -- TPM Backend:       passthrough
==> Debian12:  -- TPM Path:          
==> Debian12:  -- Disks:         vdc(qcow2, virtio, 1G), vdd(qcow2, virtio, 1G), vde(qcow2, virtio, 1G), vdf(qcow2, virtio, 1G), vdg(qcow2, virtio, 1G)
==> Debian12:  -- Disk(vdc):     /home/max/libvirt/images/vg3_Debian12-vdc.qcow2
==> Debian12:  -- Disk(vdd):     /home/max/libvirt/images/vg3_Debian12-vdd.qcow2
==> Debian12:  -- Disk(vde):     /home/max/libvirt/images/vg3_Debian12-vde.qcow2
==> Debian12:  -- Disk(vdf):     /home/max/libvirt/images/vg3_Debian12-vdf.qcow2
==> Debian12:  -- Disk(vdg):     /home/max/libvirt/images/vg3_Debian12-vdg.qcow2
==> Debian12:  -- INPUT:             type=mouse, bus=ps2
==> Debian12: Creating shared folders metadata...
==> Debian12: Starting domain.
==> Debian12: Waiting for domain to get an IP address...
==> Debian12: Waiting for machine to boot. This may take a few minutes...
```
#### Provisioning. Конфигурируем RAID-массив, создаём разделы
После того, как мы убедились, что дисковые устройства успешно добавляются (пусть и игнорируя параметр :device), используем provisioning для постустановочной настройки гостевой ОС.
Для этого добавим блок:
```
  config.vm.provision "shell", inline: <<-SHELL
    apt -y install mdadm
    mdadm --create /dev/md0 --level=5  --raid-devices=3 /dev/vd{c,d,e} --spare-devices=1 /dev/vdf
    echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
    mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
    parted -s /dev/md0 mklabel gpt
    parted /dev/md0 mkpart primary ext4 0% 20%
    parted /dev/md0 mkpart primary ext4 20% 40%
    parted /dev/md0 mkpart primary ext4 40% 60%
    parted /dev/md0 mkpart primary ext4 60% 80%
    parted /dev/md0 mkpart primary ext4 80% 100%
    for i in $(seq 1 5); do mkfs.ext4 /dev/md0p$i; done
    for i in $(seq 1 5); do mount --mkdir /dev/md0p$i /media/part$i; done
    SHELL
```
В файл [Vagrantfile](Vagrantfile).
После успешного разворачивания гостя, заходим на машину по ssh и проверяем, какие блочные устройства появились:
```
vagrant@debian12:~$ sudo -i
root@debian12:~# lsblk 
NAME                    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
vda                     254:0    0   40G  0 disk  
├─vda1                  254:1    0  953M  0 part  /boot
├─vda2                  254:2    0    1K  0 part  
└─vda5                  254:5    0 39,1G  0 part  
  ├─vg--system-lvolroot 253:0    0 18,6G  0 lvm   /
  ├─vg--system-lvolswap 253:1    0  3,7G  0 lvm   [SWAP]
  └─vg--system-lvolhome 253:2    0 16,7G  0 lvm   /home
vdb                     254:16   0    1G  0 disk  
vdc                     254:32   0    1G  0 disk  
└─md0                     9:0    0    2G  0 raid5 
  ├─md0p1               259:0    0  408M  0 part  /media/part1
  ├─md0p2               259:1    0  409M  0 part  /media/part2
  ├─md0p3               259:2    0  408M  0 part  /media/part3
  ├─md0p4               259:3    0  409M  0 part  /media/part4
  └─md0p5               259:4    0  408M  0 part  /media/part5
vdd                     254:48   0    1G  0 disk  
└─md0                     9:0    0    2G  0 raid5 
  ├─md0p1               259:0    0  408M  0 part  /media/part1
  ├─md0p2               259:1    0  409M  0 part  /media/part2
  ├─md0p3               259:2    0  408M  0 part  /media/part3
  ├─md0p4               259:3    0  409M  0 part  /media/part4
  └─md0p5               259:4    0  408M  0 part  /media/part5
vde                     254:64   0    1G  0 disk  
└─md0                     9:0    0    2G  0 raid5 
  ├─md0p1               259:0    0  408M  0 part  /media/part1
  ├─md0p2               259:1    0  409M  0 part  /media/part2
  ├─md0p3               259:2    0  408M  0 part  /media/part3
  ├─md0p4               259:3    0  409M  0 part  /media/part4
  └─md0p5               259:4    0  408M  0 part  /media/part5
vdf                     254:80   0    1G  0 disk  
└─md0                     9:0    0    2G  0 raid5 
  ├─md0p1               259:0    0  408M  0 part  /media/part1
  ├─md0p2               259:1    0  409M  0 part  /media/part2
  ├─md0p3               259:2    0  408M  0 part  /media/part3
  ├─md0p4               259:3    0  409M  0 part  /media/part4
  └─md0p5               259:4    0  408M  0 part  /media/part5
```
Содержимое mdadm.conf:
```
root@debian12:~# cat /etc/mdadm/mdadm.conf 
DEVICE partitions
ARRAY /dev/md0 level=raid5 num-devices=3 metadata=1.2 spares=2 name=debian12:0 UUID=a1f39900:c9b3f8e0:e8c791d7:be7bf952
```
Содержимое каталогов со смонтированными разделами на новом дисковом массиве:
```
root@debian12:~# ls -l /media/part{1,2,3,4,5}
/media/part1:
итого 12
drwx------ 2 root root 12288 июн  7 12:40 lost+found

/media/part2:
итого 12
drwx------ 2 root root 12288 июн  7 12:40 lost+found

/media/part3:
итого 12
drwx------ 2 root root 12288 июн  7 12:40 lost+found

/media/part4:
итого 12
drwx------ 2 root root 12288 июн  7 12:40 lost+found

/media/part5:
итого 12
drwx------ 2 root root 12288 июн  7 12:40 lost+found
```
Проверим состояние массива:
```
vagrant@debian12:~$ sudo -i
root@debian12:~# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 vde[4] vdf[3](S) vdd[1] vdc[0]
      2093056 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
root@debian12:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jun  7 13:25:56 2024
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Fri Jun  7 13:26:32 2024
             State : clean 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : debian12:0  (local to host debian12)
              UUID : 008448a0:0110af8c:6d8bc620:d607bb36
            Events : 32

    Number   Major   Minor   RaidDevice State
       0     254       32        0      active sync   /dev/vdc
       1     254       48        1      active sync   /dev/vdd
       4     254       64        2      active sync   /dev/vde

       3     254       80        -      spare   /dev/vdf
```
Видим, что устройства vdc, vdd и vde собраны в массив raid5, а устройство vdf является диском горячей замены.

Пометим устройство vdc, как сбойное:
```
root@debian12:~# mdadm --manage /dev/md0 --fail /dev/vdc
mdadm: set /dev/vdc faulty in /dev/md0
root@debian12:~# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 vde[4] vdf[3] vdd[1] vdc[0](F)
      2093056 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [_UU]
      [==============>......]  recovery = 74.7% (783036/1046528) finish=0.0min speed=156607K/sec
      
unused devices: <none>
root@debian12:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jun  7 13:25:56 2024
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Fri Jun  7 13:27:06 2024
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : debian12:0  (local to host debian12)
              UUID : 008448a0:0110af8c:6d8bc620:d607bb36
            Events : 61

    Number   Major   Minor   RaidDevice State
       3     254       80        0      active sync   /dev/vdf
       1     254       48        1      active sync   /dev/vdd
       4     254       64        2      active sync   /dev/vde

       0     254       32        -      faulty   /dev/vdc
```
Здесь мы видим, что mdadm сразу включил в работу дисковое устройство vdf, являющееся hot-spare для данного массива и запустил процедуру перестройки массива.

Теперь выведем его из массива и очистим метаданные, чтобы удалить всю информацию о принадлежности диска массиву:
```
root@debian12:~# mdadm --manage /dev/md0 --remove /dev/vdc
mdadm: hot removed /dev/vdc from /dev/md0
root@debian12:~# mdadm --zero-superblock /dev/vdc
```
Снова проверим статус:
```
root@debian12:~# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 vde[4] vdf[3] vdd[1]
      2093056 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
root@debian12:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jun  7 13:25:56 2024
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Fri Jun  7 13:27:50 2024
             State : active 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : debian12:0  (local to host debian12)
              UUID : 008448a0:0110af8c:6d8bc620:d607bb36
            Events : 63

    Number   Major   Minor   RaidDevice State
       3     254       80        0      active sync   /dev/vdf
       1     254       48        1      active sync   /dev/vdd
       4     254       64        2      active sync   /dev/vde

```
И добавим очищенный диск обратно в массив:
```
root@debian12:~# mdadm --manage /dev/md0 --add /dev/vdc
mdadm: added /dev/vdc
root@debian12:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jun  7 13:25:56 2024
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Fri Jun  7 13:28:04 2024
             State : active 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : debian12:0  (local to host debian12)
              UUID : 008448a0:0110af8c:6d8bc620:d607bb36
            Events : 66

    Number   Major   Minor   RaidDevice State
       3     254       80        0      active sync   /dev/vdf
       1     254       48        1      active sync   /dev/vdd
       4     254       64        2      active sync   /dev/vde

       5     254       32        -      spare   /dev/vdc
```
Видим, что теперь у нас устройство /dev/vdc является диском горячей замены.

К данной работе прилагаю также запись консоли. Для того, чтобы воспроизвести выполненные действия, необходимо скачать файлы [screenrecord-2024-06-07.script](screenrecord-2024-06-07.script)
и [screenrecord-2024-06-07.time](screenrecord-2024-06-07.time), после чего выполнить в каталоге с загруженными файлами команду scriptreplay ./screenrecord-2024-06-07.time ./screenrecord-2024-06-07.script

Спасибо за прочтение!
