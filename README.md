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
arch-chroot /mnt # меняет корень на /mnt и входит в новую систему
```
