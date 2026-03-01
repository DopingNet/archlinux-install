# ARCHLINUX-INSTALL

задать пароль для root

```bash
passwd
```

запустить разметку диска
```bash
cfdisk -z /dev/sda
```

UEFI

список разделов:

/dev/sda1 - EFI 1G

/dev/sda2 - SWAP 8G

/dev/sda3 - ROOT все остальное

создать файловые системы

```bash
mkfs.fat -F32 /dev/sda1
```

```bash
mkswap /dev/sda2
```

```bash
swapon /dev/sda2
```

```bash
mkfs.ext4 /dev/sda3
```

подключить репозиторий multilib
```bash
nano /etc/pacman.conf
```

#[multilib]

#Include = /etc/pacman.d/mirrorlist

убрать комментарий #

обновить список пакетов
```bash
pacman -Syy
```

монтируем корневой раздел
```bash
mount /dev/sda3 /mnt
```
установить базовую систему
```bash
pacstrap -K /mnt base linux linux-firmware sudo nano networkmanager openssh mc
```

сгенерировать таблицу разделов
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

переходим в окружение новой системы
```bash
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
echo arch-vbox > /etc/hostname # задаёт имя хоста
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
pacman -S gnome gnome-extra networkmanager # устанавливает GNOME, GNOME-дополнительные пакеты и сетевой менеджер
systemctl enable gdm && systemctl enable NetworkManager # включает автозапуск GDM и NetworkManager
