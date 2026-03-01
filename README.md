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

systemctl enable NetworkManager

ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime # устанавливает часовой пояс через символическую ссылку

nano /etc/locale.gen # открывает список доступных локалей в редакторе

en_US.UTF-8 UTF-8
ru_RU.UTF-8 UTF-8

locale-gen # генерирует локали на основе /etc/locale.gen

nano /etc/vconsole.conf
FONT=cyr-sun16
KEYMAP=us

mkinitcpio -P

hwclock --systohc # записывает системное время в аппаратные часы

echo LANG=ru_RU.UTF-8 > /etc/locale.conf && export LANG=ru_RU.UTF-8 # задаёт системную локаль

echo arch-vbox > /etc/hostname # задаёт имя хоста

nano /etc/hosts # открывает файл сопоставления имён и IP-адресов в редакторе
127.0.0.1      localhost
::1            localhost
127.0.1.1      arch-vbox

passwd

UEFI
----
pacman -S grub efibootmgr os-prober mtools # устанавливает GRUB и сопутствующие утилиты для UEFI
mkdir /boot/efi && mount /dev/sda1 /boot/efi # создаёт и монтирует точку монтирования EFI
grub-install --target=x86_64-efi --bootloader-id=grub_uefi # устанавливает GRUB в EFI-раздел
grub-mkconfig -o /boot/grub/grub.cfg # генерирует конфигурационный файл GRUB

BIOS
----
pacman -S grub os-prober mtools # устанавливает GRUB и сопутствующие утилиты для UEFI
grub-install /dev/sda # устанавливает GRUB в /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg # генерирует конфигурационный файл GRUB

useradd -m -G wheel test

passwd test

nano /etc/sudoers

%wheel раскомментировать

mkdir /home/test
chown test /home/test

GNOME
-----
pacman -S gnome gnome-extra networkmanager # устанавливает GNOME, GNOME-дополнительные пакеты и сетевой менеджер
systemctl enable gdm && systemctl enable NetworkManager # включает автозапуск GDM и NetworkManager
