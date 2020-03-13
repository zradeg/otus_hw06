# Загрузка системы
## Задачи:
1. Сбросить пароль root
2. Переименовть VG с корневым томом
3. Добавить свой модуль в initrd


## Сброс пароля root
### Шаг первый. Для всех способов
В меню загрузчика нажимаем клавишу __е__ чтобы получить доступ к редактированию параметров загрузчика ядра.
![reset_pwd_01.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_01.png)
### Способ №1
Производим правку аналогично скриншоту:
![reset_pwd_02.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/sreenshot/reset_pwd_02.png)

После чего нажимаем __Ctrl+x__
В результате проделанных действий получили доступ к шеллу с правами суперпользователя. Благодаря параметру __rw__ примененному выше раздел /root смонтирован в режиме read-write.
Теперь можно сменить пароль стандартным способом. После чего создаем файл /.autorelabel чтобы подтвердить легитимность внесенных изменений в /etc/shadow для selinux. Затем производим перемонтирование в режим read-only.
Отправляем систему в перезагрузку:
![reset_pwd_03.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_03.png)

В процессе загрузки наблюдаем реакцию selinux на смену пароля:
![reset_pwd_04.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_04.png)

Проверям авторизацию с новым паролем:

![reset_pwd_05.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_05.png)


#### Способы №2 и №3 аналогичны первому, за исключением того, что при получении доступа к шеллу, в корень смонтирован не сам корень, а sysroot и получить доступ к содержимому корня и действиями над ним можно посредством chroot.

### Способ №2
Производим правку аналогично скриншоту:
![reset_pwd_12.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_12.png)

После чего нажимаем __Ctrl+x__
В результате проделанных действий получили доступ к шеллу с правами суперпользователя. При этом способе при получении доступа к шеллу, в корень смонтирован не сам корень, а sysroot и получить доступ к содержимому корня и действиями над ним можно посредством chroot. Аналогично первому способу раздел смотнтирован в режиме read-write.
После применения chroot действия аналогичны первому способу:

![reset_pwd_13.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_13.png)

Далее аналогично способу №1:

![reset_pwd_14.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_14.png)

![reset_pwd_15.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_15.png)


### Способ №3
Производим правку аналогично скриншоту:

![reset_pwd_22.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_22.png)

Дальнейшие шаги аналогичны способу №2:

![reset_pwd_23.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_23.png)


![reset_pwd_24.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_24.png)


![reset_pwd_25.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/reset_pwd_25.png)



## Переименование VG
Выясняем данные по VG:
```[root@hw06 vagrant]# vgs
  VG                  #PV #LV #SN Attr   VSize  VFree
  centos_centos7-hw06   1   2   0 wz--n- <5.00g    0
  ```
И для логических томов в этой группе:
```[root@hw06 vagrant]# lvs centos_centos7-hw06
  LV   VG                  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root centos_centos7-hw06 -wi-ao----   4.39g
  swap centos_centos7-hw06 -wi-ao---- 616.00m
 ```
Переименовываем VG с помощью утилиты __vgrename__ и проверяем результат:
```[root@hw06 vagrant]# vgrename centos_centos7-hw06 hw06
  Volume group "centos_centos7-hw06" successfully renamed to "hw06"
[root@hw06 vagrant]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  hw06   1   2   0 wz--n- <5.00g    0
[root@hw06 vagrant]# sed 's/centos_centos7-hw06/hw06/g' /etc/fstab
 ```

Проверяем содержимое /etc/fstab:
```[root@hw06 vagrant]# cat /etc/fstab
Created by anaconda on Wed Mar 11 13:13:28 2020

Accessible filesystems, by reference, are maintained under '/dev/disk'
See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info

/dev/mapper/centos_centos7--hw06-root /                       xfs     defaults        0 0
UUID=867dc2a2-f38a-422e-a518-b346654e1614 /boot                   xfs     defaults        0 0
/dev/mapper/centos_centos7--hw06-swap swap                    swap    defaults        0 0
```
Здесь видно, что необходимо учесть для замены посредством __sed:
```[root@hw06 vagrant]# sed -i 's/centos_centos7--hw06/hw06/g' /etc/fstab
```
Так же, с помощью __sed редактируем /etc/default/grub
```[root@hw06 vagrant]# sed -i 's/centos_centos7-hw06/hw06/g' /etc/default/grub
 ```
И /boot/grub2/grub.cfg:
```[root@hw06 vagrant]# sed -i 's/centos_centos7-hw06/hw06/g' /boot/grub2/grub.cfg
 ```
Проверяем:
```[root@hw06 vagrant]# grep hw06 /boot/grub2/grub.cfg
        linux16 /vmlinuz-3.10.0-1062.12.1.el7.x86_64 root=/dev/mapper/centos_centos7--hw06-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=hw06/root rd.lvm.lv=hw06/swap rhgb quiet LANG=en_US.UTF-8
        linux16 /vmlinuz-3.10.0-1062.el7.x86_64 root=/dev/mapper/centos_centos7--hw06-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=hw06/root rd.lvm.lv=hw06/swap rhgb quiet LANG=en_US.UTF-8
        linux16 /vmlinuz-0-rescue-a002dae36b884b389571c3b93e615705 root=/dev/mapper/centos_centos7--hw0-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=hw06/root rd.lvm.lv=hw06/swap rhgb quiet
 ```
Заменяем оставшееся:
```[root@hw06 vagrant]# sed -i 's/centos_centos7--hw06/hw06/g' /boot/grub2/grub.cfg
 ```
Проверяем:
```[root@hw06 vagrant]# grep hw06 /boot/grub2/grub.cfg
        linux16 /vmlinuz-3.10.0-1062.12.1.el7.x86_64 root=/dev/mapper/hw06-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=hw06/root rd.lvm.lv=hw06/swap rhgb quiet LANG=en_US.UTF-8
        linux16 /vmlinuz-3.10.0-1062.el7.x86_64 root=/dev/mapper/hw06-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=hw06/root rd.lvm.lv=hw06/swap rhgb quiet LANG=en_US.UTF-8
        linux16 /vmlinuz-0-rescue-a002dae36b884b389571c3b93e615705 root=/dev/mapper/hw06-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=hw06/root rd.lvm.lv=hw06/swap rhgb quiet
 ```
Обновляем атрибуты VG
```[root@hw06 vagrant]# vgchange -ay
  2 logical volume(s) in volume group "hw06" now active
 ```

Обновляем initrd:
```[root@hw06 vagrant]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
/sbin/dracut: line 681: warning: setlocale: LC_MESSAGES: cannot change locale (C.UTF-8): No such file or directory
/sbin/dracut: line 682: warning: setlocale: LC_CTYPE: cannot change locale (C.UTF-8): No such file or directory
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-1062.12.1.el7.x86_64.img 3.10.0-1062.12.1.el7.x86_64
dracut module 'modsign' will not be installed, because command 'keyctl' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found! dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'cifs' will not be installed, because command 'mount.cifs' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsistart' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsi-iname' could not be found!
95nfs: Could not find any command of 'rpcbind portmap'!
dracut module 'modsign' will not be installed, because command 'keyctl' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found! dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'cifs' will not be installed, because command 'mount.cifs' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsistart' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsi-iname' could not be found!
95nfs: Could not find any command of 'rpcbind portmap'!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: network ***
*** Including module: ifcfg ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: microcode_ctl-fw_dir_override ***
  microcode_ctl module: mangling fw_dir
    microcode_ctl: reset fw_dir to "/lib/firmware/updates /lib/firmware"
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel"...
intel: model '', path ' intel-ucode/*', kvers ''
intel: blacklist ''
    microcode_ctl: intel: Host-Only mode is enabled and "intel-ucode/06-9e-0a" matches "intel-ucode/*"
      microcode_ctl: intel: caveats check for kernel version "3.10.0-1062.12.1.el7.x86_64" passed, adding "/usr/share/microcode_ctl/ucode_with_caveats/intel" to fw_dir variable
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel-06-2d-07"...
intel-06-2d-07: model 'GenuineIntel 06-2d-07', path ' intel-ucode/06-2d-07', kvers ''
intel-06-2d-07: blacklist ''
intel-06-2d-07: caveat is disabled in configuration
    microcode_ctl: kernel version "3.10.0-1062.12.1.el7.x86_64" failed early load check for "intel-06-2d-07", skipping
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel-06-4f-01"...
intel-06-4f-01: model 'GenuineIntel 06-4f-01', path ' intel-ucode/06-4f-01', kvers ' 4.17.0 3.10.0-894 3.10.0-862.6.1 3.10.0-693.35.1 3.10.0-514.52.1 3.10.0-327.70.1 2.6.32-754.1.1 2.6.32-573.58.1 2.6.32-504.71.1 2.6.32-431.90.1 2.6.32-358.90.1'
intel-06-4f-01: blacklist ''
intel-06-4f-01: caveat is disabled in configuration
    microcode_ctl: kernel version "3.10.0-1062.12.1.el7.x86_64" failed early load check for "intel-06-4f-01", skipping
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel-06-55-04"...
intel-06-55-04: model 'GenuineIntel 06-55-04', path ' intel-ucode/06-55-04', kvers ''
intel-06-55-04: blacklist ''
intel-06-55-04: caveat is disabled in configuration
    microcode_ctl: kernel version "3.10.0-1062.12.1.el7.x86_64" failed early load check for "intel-06-55-04", skipping
    microcode_ctl: final fw_dir: "/usr/share/microcode_ctl/ucode_with_caveats/intel /lib/firmware/updates /lib/firmware"
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** Constructing GenuineIntel.bin ****
*** Store current command line parameters ***
*** Creating image file ***
*** Creating microcode section ***
*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-1062.12.1.el7.x86_64.img' done ***
 ```
Перезагружаемся:
```[root@hw06 vagrant]# reboot
 ```
### Проверка
```[root@hw06 vagrant]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  hw06   1   2   0 wz--n- <5.00g    0
[root@hw06 vagrant]# lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root hw06 -wi-ao----   4.39g
  swap hw06 -wi-ao---- 616.00m
[root@hw06 vagrant]#
 ```

## Добавить свой модуль в initrd
Переходим в директорию с модулями, создаем новую директорию для своего модуля, в ней создаем файлы модуля:
```[root@hw06 vagrant]# cd /usr/lib/dracut/modules.d/
[root@hw06 modules.d]# mkdir 01logo
[root@hw06 modules.d]# cd 01logo

[root@hw06 01logo]# cat > logo.sh
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/logo.sh"
}
[root@hw06 01logo]# cat logo.sh
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'
++++++++++++++++++++++++++++++++++
+--++++++--+++++++++++++++++++++++
+--++++++++++-----++--+--++--++--+
+--+++-++--++--+--++--+--++++--+++
+------++--++--+--++-----++--++--+
++++++++++++++++++++++++++++++++++
+г==¬+++++++++++г¬г¬гT=¬г=T¬гT¬+++
+L¦¦-г¬г=T=T=T=¬¦L¦L+¦=¦¦=¦L+¦L¬++
+г¦¦¬¦L¦+¦¬¦г¦¦¦¦г¦¦¦¦=¦¦=¦¦¦¦г¦++
+L==-L=¦=-L=-L=-L=¦¦¦¦=-L=¦¦¦¦=-++
++++++++++++++++++++++++++++++++++
msgend
sleep 10
echo " continuing...."

[root@hw06 01logo]# cat > module-setup.sh
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/logo.sh"
}
 ```

Пересоздаем initrd:
```[root@hw06 vagrant]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
/sbin/dracut: line 681: warning: setlocale: LC_MESSAGES: cannot change locale (C.UTF-8): No such file or directory
/sbin/dracut: line 682: warning: setlocale: LC_CTYPE: cannot change locale (C.UTF-8): No such file or directory
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-1062.12.1.el7.x86_64.img 3.10.0-1062.12.1.el7.x86_64
dracut module 'modsign' will not be installed, because command 'keyctl' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found! dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'cifs' will not be installed, because command 'mount.cifs' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsistart' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsi-iname' could not be found!
95nfs: Could not find any command of 'rpcbind portmap'!
dracut module 'modsign' will not be installed, because command 'keyctl' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found! dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'cifs' will not be installed, because command 'mount.cifs' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsistart' could not be found!
dracut module 'iscsi' will not be installed, because command 'iscsi-iname' could not be found!
95nfs: Could not find any command of 'rpcbind portmap'!
*** Including module: bash ***
*** Including module: logo ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: network ***
*** Including module: ifcfg ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: microcode_ctl-fw_dir_override ***
  microcode_ctl module: mangling fw_dir
    microcode_ctl: reset fw_dir to "/lib/firmware/updates /lib/firmware"
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel"...
intel: model '', path ' intel-ucode/*', kvers ''
intel: blacklist ''
    microcode_ctl: intel: Host-Only mode is enabled and "intel-ucode/06-9e-0a" matches "intel-ucode/*"
      microcode_ctl: intel: caveats check for kernel version "3.10.0-1062.12.1.el7.x86_64" passed, adding "/usr/share/microcode_ctl/ucode_with_caveats/intel" to fw_dir variable
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel-06-2d-07"...
intel-06-2d-07: model 'GenuineIntel 06-2d-07', path ' intel-ucode/06-2d-07', kvers ''
intel-06-2d-07: blacklist ''
intel-06-2d-07: caveat is disabled in configuration
    microcode_ctl: kernel version "3.10.0-1062.12.1.el7.x86_64" failed early load check for "intel-06-2d-07", skipping
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel-06-4f-01"...
intel-06-4f-01: model 'GenuineIntel 06-4f-01', path ' intel-ucode/06-4f-01', kvers ' 4.17.0 3.10.0-894 3.10.0-862.6.1 3.10.0-693.35.1 3.10.0-514.52.1 3.10.0-327.70.1 2.6.32-754.1.1 2.6.32-573.58.1 2.6.32-504.71.1 2.6.32-431.90.1 2.6.32-358.90.1'
intel-06-4f-01: blacklist ''
intel-06-4f-01: caveat is disabled in configuration
    microcode_ctl: kernel version "3.10.0-1062.12.1.el7.x86_64" failed early load check for "intel-06-4f-01", skipping
    microcode_ctl: processing data directory  "/usr/share/microcode_ctl/ucode_with_caveats/intel-06-55-04"...
intel-06-55-04: model 'GenuineIntel 06-55-04', path ' intel-ucode/06-55-04', kvers ''
intel-06-55-04: blacklist ''
intel-06-55-04: caveat is disabled in configuration
    microcode_ctl: kernel version "3.10.0-1062.12.1.el7.x86_64" failed early load check for "intel-06-55-04", skipping
    microcode_ctl: final fw_dir: "/usr/share/microcode_ctl/ucode_with_caveats/intel /lib/firmware/updates /lib/firmware"
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** Constructing GenuineIntel.bin ****
*** Store current command line parameters ***
*** Creating image file ***
*** Creating microcode section ***
*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-1062.12.1.el7.x86_64.img' done ***
 ```
Проверяем:
```[root@hw06 01logo]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep logo
logo
 ```
Отправляем систему в перезагрузку и в процессе наблюдаем результат:
![add_own_module_proof.png](https://github.com/zradeg/otus_hw06/blob/master/screenshot/add_own_module_proof.png) 
