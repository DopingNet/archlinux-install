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
