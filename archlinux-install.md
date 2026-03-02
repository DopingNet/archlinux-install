# ARCHLINUX-INSTALL

задать пароль для root

```
passwd
```

запустить разметку диска
```
cfdisk -z /dev/sda
```

UEFI
-

 список разделов:
 
> sda1 - EFI 1G
> 
> sda2 - SWAP 8G
> 
> sda3 - ROOT все остальное

создать файловые системы

```
mkfs.fat -F32 /dev/sda1
```

```
mkswap /dev/sda2
```

```
swapon /dev/sda2
```

```
mkfs.ext4 /dev/sda3
```

BIOS
-

список разделов:

> sda1 - SWAP 8G
>
> sda2 - ROOT все остальное

создать файловые системы

```
mkswap /dev/sda1
```

```
swapon /dev/sda1
```

```
mkfs.ext4 /dev/sda2
```

НАСТРОЙКА ПАКЕТНОГО МЕНЕДЖЕРА
-
подключить репозиторий multilib
```
nano /etc/pacman.conf
```

#[multilib]

#Include = /etc/pacman.d/mirrorlist

убрать комментарий #

обновить список пакетов
```
pacman -Syy
```

МОНТИРОВАНИЕ КОРНЕВОГО РАЗДЕЛА И УСТАНОВКА БАЗОВОЙ СИСТЕМЫ
-
монтируем корневой раздел

UEFI
```
mount /dev/sda3 /mnt
```

BIOS
```
mount /dev/sda2 /mnt
```

установить базовую систему
```
pacstrap -K /mnt base linux linux-firmware sudo nano networkmanager openssh mc
```

сгенерировать таблицу разделов
```
genfstab -U /mnt >> /mnt/etc/fstab
```

переходим в окружение новой системы
```
arch-chroot /mnt
```

включаем службу NetworkManager
```
systemctl enable NetworkManager
```
задаем часовой пояс
```
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
```
включаем локали
```
nano /etc/locale.gen
```
```
en_US.UTF-8 UTF-8
ru_RU.UTF-8 UTF-8
```
генерируем локали
```
locale-gen
```
включаем русский шрифт в консоли
```
nano /etc/vconsole.conf
```
```
FONT=cyr-sun16
KEYMAP=us
```
сгенерируем initramfs заново
```
mkinitcpio -P
```

синхронизируем время
```
hwclock --systohc
```

установим локаль системы
```
echo LANG=ru_RU.UTF-8 > /etc/locale.conf && export LANG=ru_RU.UTF-8
```

зададим имя хоста
```
echo arch-vbox > /etc/hostname
```
отредактируем hosts
```
nano /etc/hosts
```
```
127.0.0.1      localhost
::1            localhost
127.0.1.1      arch-vbox
```
зададим пароль root в новой системе
```
passwd
```

ЗАГРУЗЧИК

UEFI
```
pacman -S grub efibootmgr os-prober mtools
```
```
mkdir /boot/efi && mount /dev/sda1 /boot/efi
```
```
grub-install --target=x86_64-efi --bootloader-id=grub_uefi
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```

BIOS
```
pacman -S grub os-prober mtools
```
```
grub-install /dev/sda
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```

добавим пользователя test в группу wheel
```
useradd -m -G wheel test
```

зададим для него пароль
```
passwd test
```

в файле sudoers расскомментируем %wheel (убрать #)
```
nano /etc/sudoers
```

создадим папку пользователя
```
mkdir /home/test
```
зададим права для нее
```
chown test /home/test
```
Базовая установка завершена

выйти из chroot
```
exit
```

отмонтировать корневой раздел
```
umount -R /mnt
```


GNOME
-----
```
pacman -S gnome gnome-extra networkmanager
```
```
systemctl enable gdm && systemctl enable NetworkManager
```
