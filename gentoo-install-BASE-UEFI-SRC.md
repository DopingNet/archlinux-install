Задаю пароль для root
```
passwd
```

запускаю ssh сервер для подключения к виртуальной машине
```
/etc/init.d/sshd start
```

Запускаю разметку диска
```
cfdisk -z /dev/sda
```

создаю файловые системы

UEFI
-

> sda1 - EFI
>
> sda2 - SWAP
>
> sda3 - root

```
mkfs.fat -F 32 /dev/sda1
```
```
mkswap /dev/sda2
```
```
swapon /dev/sda2
```
```
mkfs.xfs /dev/sda3
```


BIOS
-

> sda1 - SWAP
>
> sda2 - root

```
mkswap /dev/sda1
```
```
swapon /dev/sda1
```
```
mkfs.xfs /dev/sda2
```

Создаю корневую папку
```
mkdir --parents /mnt/gentoo
```

Монтирую корневой раздел (для BIOS - /dev/sda2)
```
mount /dev/sda3 /mnt/gentoo
```

Создаю папку для EFI (Для BIOS пропустить)
```
mkdir --parents /mnt/gentoo/efi
```

Перехожу в корневую папку
```
cd /mnt/gentoo
```

Устанавливаю время (месяц, день, час, минуты, год)
```
date 030219562026
```

Скачиваю stage3 (смотри видео как найти нужный)
```
links https://www.gentoo.org/downloads/mirrors/
```

Распаковываю stage3
```
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
```

Редактирую файл настроек make.conf (что и где писать в видео, тут чуть позже)
```
nano /mnt/gentoo/etc/portage/make.conf
```

Копирую информацию о сети в новую систему
```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

Монтирую необходимые папки
```
mount --types proc /proc /mnt/gentoo/proc
```
```
mount --rbind /sys /mnt/gentoo/sys
```
```
mount --make-rslave /mnt/gentoo/sys
```
```
mount --rbind /dev /mnt/gentoo/dev
```
```
mount --make-rslave /mnt/gentoo/dev
```
```
mount --bind /run /mnt/gentoo/run
```
```
mount --make-slave /mnt/gentoo/run
```

Выполняю вход в новое окружение и перезагружаю сеанс
```
chroot /mnt/gentoo /bin/bash
```
```
source /etc/profile
```
```
export PS1="(chroot) ${PS1}"
```

Монтирую раздел загрузчика
```
mount /dev/sda1 /efi
```

Если создавался раздел boot для BIOS то монтируем (для UEFI - пропустить)
```
mount /dev/sda1 /boot
```

Обновляю дерево портежей
```
emerge-webrsync
```

Обновляю список ebuild файлов
```
emerge --sync
```

Читаю новости
```
eselect news list
```
```
eselect news read
```

Смотрю список профилей
```
eselect profile list
```

Выбираю профиль 3
```
eselect profile set 3
```

Обновляю систему
```
emerge --ask --verbose --update --deep --changed-use @world
```

Делаю очистку
```
emerge --ask --depclean
```

Устанавливаю часовой пояс
```
ln -sf ../usr/share/zoneinfo/Europe/Brussels /etc/localtime
```

Редактирую список локалей
```
nano /etc/locale.gen
```

Устанавливаю русскую и английскую локаль

> en_US.UTF-8 UTF-8
> 
> ru_RU.UTF-8 UTF-8

Генерирую локали
```
locale-gen
```

Смотрю список локалей в системе
```
eselect locale list
```

Выбираю локаль
```
eselect locale set 5
```

Перезагружаю сеанс и профиль
```
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

Установил dbus
```
emerge sys-apps/dbus
```

Установил прошивки
```
emerge --ask sys-kernel/linux-firmware
```
```
emerge --ask sys-firmware/sof-firmware
```

Установил микрокод
```
emerge sys-firmware/intel-microcode
```

Редактирую /etc/portage/package.use/installkernel
```
nano /etc/portage/package.use/installkernel
```

> sys-kernel/installkernel grub dracut

Ставлю ядро
```
emerge --ask sys-kernel/installkernel
emerge --ask sys-kernel/gentoo-kernel-bin
emerge --depclean
emerge --ask sys-kernel/gentoo-sources
```

nano /etc/fstab

BIOS
-
/dev/sda1   /boot        xfs    defaults    0 2
/dev/sda2   none         swap    sw                   0 0
/dev/sda3   /            xfs    defaults,noatime              0 1

UEFI
-
/dev/sda1   /efi        vfat    umask=0077,tz=UTC     0 2
/dev/sda2   none         swap    sw                   0 0
/dev/sda3   /            xfs    defaults,noatime              0 1

Имя компьютера
echo tux > /etc/hostname


emerge --ask net-misc/dhcpcd

openRC
-
rc-update add dhcpcd default
rc-service dhcpcd start

systemD
-
systemctl enable dhcpcd

nano /etc/hosts
# Это обязательные настройки для текущей системы
127.0.0.1     tux.homenetwork tux localhost
::1           tux.homenetwork tux localhost

passwd

emerge --ask app-admin/sysklogd
rc-update add sysklogd default

emerge --ask sys-process/cronie
rc-update add cronie default

emerge --ask app-shells/bash-completion

emerge --ask sys-block/io-scheduler-udev-rules

emerge --ask sys-fs/xfsprogs sys-fs/e2fsprogs sys-fs/dosfstools 	sys-fs/btrfs-progs 	sys-fs/f2fs-tools 	sys-fs/ntfs3g sys-fs/zfs

echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask sys-boot/grub


BIOS
-
grub-install /dev/sda

UEFI
-
grub-install --efi-directory=/efi

grub-mkconfig -o /efi/EFI/Gentoo/grub.cfg
env-update
exit

cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo
reboot
