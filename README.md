# MAW
Manjaro in Arch Way - гайд по установке Manjaro Linux через CLI
---
### ПЕРВОНАЧАЛЬНАЯ ЗАГРУЗКА
Скачиваем образ [Manjaro Linux](https://manjaro.org/download/)  
Входим в live режим, подключаемся к интернету и открываем Konsole  
`sudo su` входим в root  
`timedatectl set-ntp true` проверка точности системных часов  
`timedatectl status` проверка статуса сервиса  

### РАЗМЕТКА ДИСКА
`fdisk -l` для идентификации всех дисков  
`fdisk *ваш диск*` разметка (пример - `fdisk /dev/sda`)  

Примеры разметки из [Arch Wiki](https://wiki.archlinux.org/title/Installation_guide#Example_layouts):  
>##### BIOS with MBR:
>| Mount Point | Partition 		| Partition Type | Suggested size 	   |
>|-------------|-------------------------|----------------|-------------------------|
>|   `[SWAP]`  | `/dev/swap_partition`   |   Linux swap   | More than 512 MiB       |
>|   `/mnt`    | `/dev/root_partition`   |   Linux        | Remainder of the device |
>##### UEFI with GPT
>| Mount Point | Partition 		| Partition Type | Suggested size 	   |
>|-------------|-------------------------|----------------|-------------------------|
>|`/mnt/boot` or `/mnt/efi`|`/dev/efi_system_partition`|EFI system partition |At least 260 MiB |
>|`[SWAP]`  		  | `/dev/swap_partition`     |   Linux swap        |More than 512 MiB|
>|`/mnt`    		  | `/dev/root_partition`     |Linux x86-64 root (/)| Remainder of the device|

Автоматическая разметка Manjaro Linux для UEFI систем размечает следующим образом:
| Mount Point | Partition 		| Partition Type | Suggested size	           |
|-------------|-------------------------|----------------|---------------------------------|
|`/mnt/boot/efi`| `/dev/efi_system_partition`|EFI system partition |300 MiB                |
|`[SWAP]`       | `/dev/swap_partition`      |Linux swap           |More than 512 MiB      |
|`/mnt`         | `/dev/root_partition`      |Linux x86-64 root (/)|Remainder of the device|

### ФОРМАТИРОВАНИЕ РАЗДЕЛОВ
`lsblk` вывод списка всех созданных разделов  
`mkfs.ext4 /dev/root_partition` форматирование корневого раздела под ext4  
Также можно использовать другую файловую систему (например, btrfs) и указать имя раздела, пример: `mkfs.btrfs -L "Root" /dev/root_partition`  
`mkfs.fat -F32 /dev/efi_system_partition` форматирование efi раздела под fat32  
`mkswap /dev/swap_partition` форматирование swap раздела  

### МОНТИРОВАНИЕ ФАЙЛОВЫХ СИСТЕМ
`mount /dev/root_partition /mnt` монтирование корневого раздела  
`swapon /dev/swap_partition` включение swap раздела  
`mkdir -p /mnt/boot/efi`  
`mount /dev/efi_system_partition /mnt/boot/efi` монтирование efi раздела  
Также с помощью `lsblk` можно проверить правильность ранее выполненных действий  

### УСТАНОВКА
#### ВЫБОР ЗЕРКАЛ
Для загрузки с более быстрых зеркал стоит использовать утилиту `pacman-mirrors`  
Пример выбора самых быстрых зеркал по протоколу https:  
`pacman-mirrors --fasttrack --api --protocol https && pacman -Syyu`  
Больше информации о вариантах выбора зеркал можно почитать на [Manjaro Wiki](https://wiki.manjaro.org/index.php/Pacman-mirrors)  

#### УСТАНОВКА ОСНОВНЫХ ПАКЕТОВ
`basestrap /mnt base linux510 mhwd linux-firmware nano` установка базовых пакетов  

#### КОНФИГУРАЦИЯ СИСТЕМЫ
`fstabgen -U /mnt >> /mnt/etc/fstab` генерация fstab с идентификацией по UUID  
`manjaro-chroot /mnt /bin/bash` смена root на новую систему  
`mhwd -i pci video-nvidia` установка драйвера (ТОЛЬКО ЕСЛИ ВИДЕОКАРТА ОТ NVIDIA)  
`ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime` установка часового пояса (пример с Москвой)  

#### ЛОКАЛИЗАЦИЯ
Для установки локализации надо зайти в `/etc/locale.gen` и раскомментировать нужные локали  
`nano /etc/locale.gen`  
Для установки английской и русской локализации надо раскомментировать следующие строки:  
`en_US.UTF-8 UTF-8`  
`ru_RU.UTF-8 UTF-8`  
`locale-gen` команда для генерации всех раскомментированных ранее локалей  
Далее, если вам нужна отличающийся от английского язык системы, в `/etc/locale.conf` надо заменить значение LANG на одну из сконфигурированных локалей (далее пример с русской локализацией):  
`nano /etc/locale.conf`  
`LANG=ru_RU.UTF-8`  

#### КОНФИГУРАЦИЯ СЕТИ
Создадите имя вашего компьютера (hostname):  
`nano /etc/hostname`  
`myhostname`  
Отредактируйте `/etc/hosts`  по следующему [шаблону](https://wiki.archlinux.org/title/Installation_guide#Network_configuration):  
>127.0.0.1  localhost  
>::1        localhost  
>127.0.1.1  myhostname.localdomain  myhostname  

#### ПАРОЛЬ СУПЕРПОЛЬЗОВАТЕЛЯ
`passwd` установка пароля суперпользователя  

#### ЗАГРУЗЧИК
Далее рассмотрен пример установки загрузчика для uefi систем, для установки загрузчика для bios систем уточняйте детали на [Arch Wiki](https://wiki.archlinux.org/title/GRUB)  
`pacman -S grub efibootmgr` установка grub  

Также опционально можно установить микрокод для intel или amd процессоров:  
`pacman -S amd-ucode`  микрокод для amd процессоров  
`pacman -S intel-ucode`  микрокод для intel процессоров  

Установка загрузчика для uefi систем с efi разделом на /boot/efi, bootloader-id можно выставить любой - это название вашей системы в выборе вариантов загрузки в биосе  
`grub-install --target=x86_64-efi --efi-directory=/boot/efi  --bootloader-id=manjaro`  
`update-grub` генерация конфигурационного файла  

### УСТАНОВКА ГРАФИЧЕСКОГО ОКРУЖЕНИЯ
#### СОЗДАНИЕ АДМИНИСТРАТОРА
`useradd -m username` создание пользователя username  
`usermod -a -G wheel username` добавление username в группу wheel  
`passwd username` установка пароля для username  
`pacman -S sudo` установка sudo и редактора nano (если его не было ранее)  
`EDITOR=nano visudo` редактирование параметров sudo  
Далее требуется раскомментировать строку, связанную с группой wheel:  
`%wheel ALL=(ALL) ALL`  

#### УСТАНОВКА KDE PLASMA
`pacman -S xorg-server plasma-desktop sddm-kcm` установка xorg-server и минимального набора plasma  
  
Для полного набора KDE Plasma, а также ее приложений, на странице установленных пакетов Manjaro KDE (ссылка в TIPS & TRICKS) нужно установить пакеты из пунктов Plasma5 и KDE Applications  
Если Вас не сильно волнует "засоренность" вашей системы, то можно выполнить следующую команду, которая установит все нужные пакеты для KDE Plasma и KDE Applications:  
`pacman -S xorg-server plasma-meta kde-applications`  
  
`systemctl enable sddm.service` включение sddm  
`systemctl enable NetworkManager.service` включение NetworkManager  

#### УСТАНОВКА GNOME
`pacman -S xorg-server gnome` установка xorg-server и gnome  
`systemctl enable gdm.service` включение gdm  
`systemctl enable NetworkManager.service` включение NetworkManager  

### ЗАВЕРШЕНИЕ УСТАНОВКИ
`exit` выход из chroot  
`shutdown now` выключение системы  

### TIPS & TRICKS
Если вы хотите получить тот же набор установленных программ, как и в обычных образах Manjaro Linux, то не выходя из chroot вы можете пройтись по списку установленных пакетов в [KDE](https://gitlab.manjaro.org/profiles-and-settings/iso-profiles/-/blob/master/manjaro/kde/Packages-Desktop), [GNOME](https://gitlab.manjaro.org/profiles-and-settings/iso-profiles/-/blob/master/manjaro/gnome/Packages-Desktop) и [XFCE](https://gitlab.manjaro.org/profiles-and-settings/iso-profiles/-/blob/master/manjaro/xfce/Packages-Desktop)  

Также на KDE могут возникнуть проблемы при изменении параметров пользователя в настройках  
Если выдает ошибку при сохранении изменений, то в `/etc/login.defs` требуется изменить поле `CHFN_RESTRICT`, поменяв значение на `frwh`  

Если при разметке в качестве файловой системы была выбрана btrfs, то, при надобности, subvolume придется создавать самому, следуя [данной статье](https://wiki.archlinux.org/title/Btrfs#Subvolumes)  

Различные детали, связанные с установкой и не только, можно уточнить на [ArchWiki](https://wiki.archlinux.org/)  

---
Created by @eightbyte81
