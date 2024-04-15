# zagr
Попасть в систему без пароля несколькими способами

При выборе ядра для загрузки нажать e - в данном контексте edit. Попадаем в окно, где мы можем изменить параметры загрузки:


Способ 1. init=/bin/sh
```
при выборе ядра для загрузки нажать e

Добавляем init=/bin/sh
```
[https://github.com/alexxeykz/zagr/blob/main/1.jpg](https://github.com/alexxeykz/zagr/blob/main/1.jpg)
Попадаем в консоль
рис2

После чего можно убедиться, жем ли мы записывать данные, записав данные в любой файл, или прочитав вывод
команды:
```
mount | grep root
```
рис 3

Команда показывает ro, что означает что мы не можем записывать, приеним команду:
```
mount -o remount,rw /
```

рис4
Теперь видим система пишет rw, можем записывать.


Способ 2. rd.break

В конце строки, начинающейся с linux16, добавляем rd.break и нажимаем сtrl-x для
загрузки в систему

рис5

Попадаем в emergency mode. Наша корневая файловая система смонтирована (опять же в режиме Read-Only, но мы не в ней). Далее будет пример, как попасть в нее и поменять пароль администратора:

рис6
```
switch_root:/#mount -o remount,rw /sysroot
switch_root:/#chroot /sysroot
sh-4.2#passwd root   
меняем пароль root

sh-4.2#touch /.autorelabel

Перезагружаемся, заходим по новому паролю
рис 7.

Способ 3. rw init=/sysroot/bin/sh

В строке, начинающейся с linux16, заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-x для загрузки в систему

Это тоже самое, что и в предыдущих, только сразу с правами на изменение:

рис 8

рис 9

Установить систему с LVM, после чего переименовать VG

Первым делом посмотрим текущее состояние системы:
```
[root@Centest ~]# vgs
  VG         #PV #LV #SN Attr   VSize  VFree
  cl_centest   1   2   0 wz--n- 31.00g 4.00m
```
Нас интересует вторая строка с именем Volume Group
Переименуем:
```
[root@Centest ~]# vgrename cl_centest OtusRoot
  Volume group "cl_centest" successfully renamed to "OtusRoot"
```
Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое
название на новое. По ссылкам можно увидеть примеры получившихся файлов.

#
# /etc/fstab
# Created by anaconda on Mon Apr 15 08:38:43 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/OtusRoot-root /                       xfs     defaults        0 0
UUID=11e9b616-1c08-484d-b7f3-e21bbca7c90d /boot                   xfs     defaults        0 0
/dev/mapper/OtusRoot-swap swap                    swap    defaults        0 0
```
```
[root@Centest ~]# vi /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```
```
[root@localhost ~]# vi /boot/grub2/grub.cfg

### BEGIN /etc/grub.d/10_linux ###
menuentry 'CentOS Linux (3.10.0-514.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-514.el7.x86_64-advanced-d99b9a85-c7c2-43b5-841a-4aca82d1252b' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  cd3ae96e-81c6-4644-b373-9e91b2b0b543
        else
          search --no-floppy --fs-uuid --set=root cd3ae96e-81c6-4644-b373-9e91b2b0b543
        fi
        linux16 /vmlinuz-3.10.0-514.el7.x86_64 root=/dev/mapper/OtusRoot-root ro crashkernel=auto rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap rhgb quiet LANG=en_US.UTF-8
        initrd16 /initramfs-3.10.0-514.el7.x86_64.img
}
menuentry 'CentOS Linux (0-rescue-897641e01a2344c6bd2d9d01ff7b0388) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-897641e01a2344c6bd2d9d01ff7b0388-advanced-d99b9a85-c7c2-43b5-841a-4aca82d1252b' {
        load_video
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  cd3ae96e-81c6-4644-b373-9e91b2b0b543
        else
          search --no-floppy --fs-uuid --set=root cd3ae96e-81c6-4644-b373-9e91b2b0b543
        fi
        linux16 /vmlinuz-0-rescue-897641e01a2344c6bd2d9d01ff7b0388 root=/dev/mapper/OtusRoot-root ro crashkernel=auto rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap rhgb quiet
        initrd16 /initramfs-0-rescue-897641e01a2344c6bd2d9d01ff7b0388.img
}



 Пересоздаем initrd image, чтобы он знал новое название Volume Group

[root@Centest ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-514.el7.x86_64.img' done ***

После чего можем перезагружаться и, если все сделано правильно, успешно грузимся с новым именем Volume Group и проверяем:
```
[root@Centest ~]# vgs
  VG         #PV #LV #SN Attr   VSize  VFree
  OtusRoot   1   2   0 wz--n- 31.00g 4.00m
```
Все загружается, идем дальше.

Добавить модуль в initrd

```
[root@localhost ~]# mkdir /usr/lib/dracut/modules.d/01test
```

В нее поместим два скрипта:
```
module-setup.sh
test.sh
```

Пересобираем образ initrd
```
[root@localhost ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
Просматриваем модули
```
[root@localhost ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
Перезагрузиться и руками выключить опции rghb и quiet и увидеть вывод
Либо отредактировать grub.cfg, убрав эти опции

```
Редактируем
[root@localhost ~]# vi /etc/default/grub
```
рис 10

