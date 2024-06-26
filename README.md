# ДЗ: Загрузка системы_Работа с загрузчиком
-----------------------------------------------------------------------
### Домашнее задание

    1. Попасть в систему без пароля несколькими способами
    2. Установить систему с LVM, после чего переименовать VG
    3. Добавить модуль в initrd

## Попасть в систему без пароля несколькими способами
```ruby
    Для получения доступа необходимо открыть GUI VirtualBox (или другой системы виртуализации), 
    запустить виртуальную машину и при выборе ядра для загрузки нажать e - в данном контексте edit. 
    Попадаем в окно, где мы можем изменить параметры загрузки:
```
```ruby
    Способ 1. init=/bin/sh
- В конце строки, начинающейся с linux16, добавляем init=/bin/sh и нажимаем сtrl-x для загрузки в систему
- В целом на этом все, Вы попали в систему. Но есть один нюанс. Рутовая файловая
система при этом монтируется в режиме Read-Only.
```
![Screenshot from 2024-04-11 16-30-47](https://github.com/d4rkgh0m/BOOT/assets/120186195/28cb41d9-450c-456a-a9c8-337bdfe1ec37) 
![Screenshot from 2024-04-11 16-32-44](https://github.com/d4rkgh0m/BOOT/assets/120186195/bb37ac9e-9dce-437e-bca5-f67bf46bd6df)
```ruby
Если вы хотите перемонтировать ее в режим Read-Write, можно воспользоваться командой:
[root@localhost ~]# mount -o remount,rw /
```
![Screenshot from 2024-04-11 16-33-13](https://github.com/d4rkgh0m/BOOT/assets/120186195/5f837bf8-ca96-4ae4-a808-d6c36faebd3b)

```ruby
- После чего можно убедиться, записав данные в любой файл или прочитав вывод
команды:
[root@localhost ~]# mount | grep root
```
![Screenshot from 2024-04-11 16-38-08](https://github.com/d4rkgh0m/BOOT/assets/120186195/c4b6d02a-2277-4cdf-8c9b-1990ddac5feb)

```ruby
Способ 2. rd.break

- В конце строки, начинающейся с linux16, добавляем rd.break и нажимаем сtrl-x для
загрузки в систему
- Попадаем в emergency mode. Наша корневая файловая система смонтирована (опять же в режиме Read-Only, но мы не в ней).
Далее будет пример, как попасть в нее и поменять пароль администратора:
[root@localhost ~]# mount -o remount,rw /sysroot
[root@localhost ~]# chroot /sysroot
[root@localhost ~]# passwd root
[root@localhost ~]# touch /.autorelabel
- После чего можно перезагружаться и заходить в систему с новым паролем.
Полезно, когда вы потеряли или вообще не имели пароль администратор.
```
![Screenshot from 2024-04-11 16-44-58](https://github.com/d4rkgh0m/BOOT/assets/120186195/07c174a8-55a1-459f-b87d-539f2b08afd7)
![Screenshot from 2024-04-11 16-45-23](https://github.com/d4rkgh0m/BOOT/assets/120186195/aa1517c3-1472-4757-9286-69d2e43816d8)
![Screenshot from 2024-04-11 16-46-55](https://github.com/d4rkgh0m/BOOT/assets/120186195/f346104c-b1f0-4b72-95b0-ab1aea1d9ac4)
![Screenshot from 2024-04-11 16-50-30](https://github.com/d4rkgh0m/BOOT/assets/120186195/72e36e02-f719-4b13-b4cd-474e5383623c)

```ruby
Способ 3. rw init=/sysroot/bin/sh
- В строке, начинающейся с linux16, заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-x для загрузки в систему
- В целом то же самое, что и в прошлом примере, но файловая система сразу смонтирована в режим Read-Write
- В прошлых примерах тоже можно заменить ro на rw
```
![Screenshot from 2024-04-11 16-59-44](https://github.com/d4rkgh0m/BOOT/assets/120186195/b2b2097a-f49d-4689-beee-70ab5d226679)

```ruby
Установить систему с LVM, после чего переименовать VG

- Сначала посмотрим текущее состояние системы:
[root@localhost ~]# vgs
  VG     #PV #LV #SN Attr   VSize  VFree 
  centos   1   2   0 wz--n- 38.57g 44.00m

- Нас интересует вторая строка с именем Volume Group
- Приступим к переименованию:

[root@localhost ~]# vgrename centos OtusRoot
Volume group "VolGroup00" successfully renamed to "OtusRoot"

- Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое
название на новое. По ссылкам можно увидеть примеры получившихся файлов.
```ruby
#
# /etc/fstab
# Created by anaconda on Fri Oct 30 07:45:29 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/OtusRoot-root /                       xfs     defaults        0 0
UUID=42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4 /boot                   xfs     defaults        0 0
/dev/mapper/OtusRoot-swap swap                    swap    defaults        0 0
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
vagrant /vagrant vboxsf uid=1001,gid=1001,_netdev 0 0
#VAGRANT-END

```
```ruby
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap crashkernel=auto rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```
<details>
#
# DO NOT EDIT THIS FILE
#
# It is automatically generated by grub2-mkconfig using templates
# from /etc/grub.d and settings from /etc/default/grub
#

### BEGIN /etc/grub.d/00_header ###
set pager=1

if [ -s $prefix/grubenv ]; then
  load_env
fi
if [ "${next_entry}" ] ; then
   set default="${next_entry}"
   set next_entry=
   save_env next_entry
   set boot_once=true
else
   set default="${saved_entry}"
fi

if [ x"${feature_menuentry_id}" = xy ]; then
  menuentry_id_option="--id"
else
  menuentry_id_option=""
fi

export menuentry_id_option

if [ "${prev_saved_entry}" ]; then
  set saved_entry="${prev_saved_entry}"
  save_env saved_entry
  set prev_saved_entry=
  save_env prev_saved_entry
  set boot_once=true
fi
function savedefault {
  if [ -z "${boot_once}" ]; then
    saved_entry="${chosen}"
    save_env saved_entry
  fi
}

function load_video {
  if [ x$feature_all_video_module = xy ]; then
    insmod all_video
  else
    insmod efi_gop
    insmod efi_uga
    insmod ieee1275_fb
    insmod vbe
    insmod vga
    insmod video_bochs
    insmod video_cirrus
  fi
}

terminal_output console
if [ x$feature_timeout_style = xy ] ; then
  set timeout_style=menu
  set timeout=5
# Fallback normal timeout code in case the timeout_style feature is
# unavailable.
else
  set timeout=5
fi
### END /etc/grub.d/00_header ###

### BEGIN /etc/grub.d/00_tuned ###
set tuned_params=""
### END /etc/grub.d/00_tuned ###

### BEGIN /etc/grub.d/10_linux ###
menuentry 'CentOS Linux (3.10.0-229.14.1.el7.x86_64) 7 (Core)' --class rhel fedora --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-229.el7.x86_64-advanced-35848a50-a159-4774-a009-9530a62df6e8' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        else
          search --no-floppy --fs-uuid --set=root 42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        fi
        linux16 /vmlinuz-3.10.0-229.14.1.el7.x86_64 root=/dev/mapper/OtusRoot-root ro rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap crashkernel=auto LANG=en_US.UTF-8 systemd.debug
        initrd16 /initramfs-3.10.0-229.14.1.el7.x86_64.img
}
menuentry 'CentOS Linux (3.10.0-229.14.1.el7.x86_64) 7 (Core) with debugging' --class rhel fedora --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-229.el7.x86_64-advanced-35848a50-a159-4774-a009-9530a62df6e8' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        else
search --no-floppy --fs-uuid --set=root 42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        fi
        linux16 /vmlinuz-3.10.0-229.14.1.el7.x86_64 root=/dev/mapper/OtusRoot-root ro rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap crashkernel=auto LANG=en_US.UTF-8 systemd.debug
        initrd16 /initramfs-3.10.0-229.14.1.el7.x86_64.img
}
menuentry 'CentOS Linux 7 (Core), with Linux 3.10.0-229.el7.x86_64' --class rhel fedora --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-229.el7.x86_64-advanced-35848a50-a159-4774-a009-9530a62df6e8' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        else
          search --no-floppy --fs-uuid --set=root 42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        fi
        linux16 /vmlinuz-3.10.0-229.el7.x86_64 root=/dev/mapper/OtusRoot-root ro rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap crashkernel=auto LANG=en_US.UTF-8
        initrd16 /initramfs-3.10.0-229.el7.x86_64.img
}
menuentry 'CentOS Linux 7 (Core), with Linux 0-rescue-60acfdd3b8024663b0a3a00d96a49deb' --class rhel fedora --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-60acfdd3b8024663b0a3a00d96a49deb-advanced-35848a50-a159-4774-a009-9530a62df6e8' {
        load_video
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        else
search --no-floppy --fs-uuid --set=root 42843ce3-bb6e-4ea7-b2e6-51eab49bbfb4
        fi
        linux16 /vmlinuz-0-rescue-60acfdd3b8024663b0a3a00d96a49deb root=/dev/mapper/OtusRoot-root ro rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap crashkernel=auto
        initrd16 /initramfs-0-rescue-60acfdd3b8024663b0a3a00d96a49deb.img
}

### END /etc/grub.d/10_linux ###

### BEGIN /etc/grub.d/20_linux_xen ###
### END /etc/grub.d/20_linux_xen ###

### BEGIN /etc/grub.d/20_ppc_terminfo ###
### END /etc/grub.d/20_ppc_terminfo ###

### BEGIN /etc/grub.d/30_os-prober ###
### END /etc/grub.d/30_os-prober ###

### BEGIN /etc/grub.d/40_custom ###
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.
### END /etc/grub.d/40_custom ###

### BEGIN /etc/grub.d/41_custom ###
if [ -f  ${config_directory}/custom.cfg ]; then
  source ${config_directory}/custom.cfg
elif [ -z "${config_directory}" -a -f  $prefix/custom.cfg ]; then
  source $prefix/custom.cfg;
fi
### END /etc/grub.d/41_custom ###
</details>

```ruby
- Пересоздаем initrd image, чтобы он знал новое название Volume Group

[root@localhost ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
...
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

- После чего можем перезагружаться и, если все сделано правильно, успешно грузимся с новым именем Volume Group и проверяем:

[root@localhost ~]# vgs
  VG       #PV #LV #SN Attr   VSize  VFree 
  OtusRoot   1   2   0 wz--n- 38.57g 44.00m

- При желании можно так же заменить название Logical Volume

```
```ruby
Добавить модуль в initrd

Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того, чтобы добавить свой модуль, создаем там папку с именем 01test:

[root@localhost ~]# mkdir /usr/lib/dracut/modules.d/01test

В нее поместим два скрипта:

1. module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh
```ruby
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}
```
```ruby
2. test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик
```ruby
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'

Hello! You are in dracut module!

 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."
```
```ruby
- Пересобираем образ initrd

[root@localhost ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
или
[root@localhost ~]# dracut -f -v

- Можно проверить/посмотреть, какие модули загружены в образ:
[root@localhost ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test

- После чего можно пойти двумя путями для проверки:
- Перезагрузиться и руками выключить опции rghb и quiet и увидеть вывод
- Либо отредактировать grub.cfg, убрав эти опции

- В итоге при загрузке будет пауза на 10 секунд и вы увидите пингвина в выводе
терминала
```
![Screenshot from 2024-04-11 18-37-01](https://github.com/d4rkgh0m/BOOT/assets/120186195/5c591769-bbb3-4774-8aad-687b648a30db)
  
  



