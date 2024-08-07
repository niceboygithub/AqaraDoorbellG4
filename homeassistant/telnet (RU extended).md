# Enable Telnet of Aqara DoorBell G4 (extended) (RU)
Это перевод оригинальной стать автора с подробным описанием некоторых моментов.
Данный перевод будет полезен новичкам, для 

Тот парень который написал оригинальную статью Гений и Волшебник! Выражаю ему глубочайшую благодарность!


## ВНИМАНИЕ! Высокие риски!
**Только вы несете всю ответственность за все дальнейшии действия!**
**Есть риск, что после данных действий камера превратиться в кирпичик.**

Все действия у меня проводились глобальной версии (global/EU), у автора оригинальной статьи на версии для китайского рынка (CN)
Прошивка 4.0.1_0028.0046 (про обновления/другии версии ничего сказать не смогу)


## Для чего это нужно?
Это аппаратное изменение настройки готового устройства Aqara G4, которое позволит открыть к нему доступ из локальной сети и попрощаться с гарантией.
Для большинства из читателей основная польза от действий - это трансляция потокового видео с камеры в локальную сеть через rtsp.
Данная реализация позволит:

- сохранить текущую настройку и функцианальность в Aqara Home 
- сохранить текущую настройку и функцианальность в Apple Home 
- открыть достук к подключению через Telnet на 23 порту
- включить трансляцию видео по rtsp в разном качестве:
    - rtsp://[your_g4_ip]:8554/360p
    - rtsp://[your_g4_ip]:8554/720p
    - rtsp://[your_g4_ip]:8554/1080p





#### 1. Разабрать внутренний модуль камеры (модуль c micro sd).
Разобрать аккуратно давольно тяжело, но возможно. Нижняя крышка держится на защелки по 2 с каждой из стороны.
Проще всего, справой стороны от usb порта расширить щель между крышкой (с наклейкой) и корпусом и вствить плоскую длинную лопатку, чтобы отщелкнуть крепления. 

   
#### 2. Подключения разъемов к плате UART.
На плате нас интересуют 3 пятачка, подробнее на картинке:

<img src="/images/g4_uart.png" alt="uart" height="520" width="460">

Для подключения лучше использовать пайку, провода в последствии можно заизолировать и легко убрать в корпус.
Так же плату можно давольно легко достать из корпуса, отключив два разъема (антена и динамик) и выкрутив 3 болта.


#### 3. Подключение по UART.
Внимание! Крайне осторожно отнеситесь к выбору USB-TTL (UART) адаптера, он должен работать в режиме 3.3V, т.е. напряжение между контактами Tx и Gnd адаптера должно равняться ~3.3 Вольта. (Это можно проверить вольтметром при подключении к пк)

При подключении на картинке выше подписаны не выводы с платы, а к каким разъемам подключать на USB-TTL. Те подписанные Tx на плате к Tx на USB-TTL, аналогично Rx, Gnd. (питание подключать не нужно)
    
1. Проверьте, что для USB-TTL установлены все необходимые драйвера и ПК его определяет и видит.
2. Подключите плату без питания к USB-TTL и USB-TTL к pc. 
3. Откройте putty и подключитесь по serial (connetion type). (номер порта можно подсмотреть в диспетчере устройств (скорее всего это COM3), скорость 115200)
4. В открывшемся окне нажмите и удерживайте "Enter"
5. Подключите питание к модулю G4 и чуть подождите
6. Если в окне подключения вы начали видеть обновляющююся строку - вы остановили загрузку и готовы к шагам далее, если нет, попробуйте снова


#### 4. Подключение к ядру NAND.
Эта команда может быть доступна не сразу, а через 10-30сек.
```
nand info
```
Проверьте правильность инициализации nand. Если все успешно, вы увидете следующее:
```
Device 0: nand0, sector size 128 KiB
  Page size      2048 b
  OOB size         64 b
  Erase size   131072 b
```
Если NAND инициализирован, выполните следующую команду:
```
printenv bootargs
```
   Она выведет глобальную переменную с настройками запуска, в формате ``bootargs=XXX``
   Вам нужно сохранить все что есть в оригинальной и изменить параметр init с ``"/linuxrc"`` на ``"/bin/sh"``
   После этого записать все это в глобальную переменную: ``setenv bootargs new_xxx``
   
   Это будет примерно следующая команда:
```
setenv bootargs root=/dev/mtdblock7 rootfstype=squashfs ro init=/bin/sh LX_MEM=0x7FE0000 mma_heap=mma_heap_name0,miu=0,sz=0x500000 cma=2M mmap_reserved=fb,miu=0,sz=0x300000,max_start_off=0x7C00000,max_end_off=0x7F00000 mtdparts=nand0:1664k@0x140000(BOOT0),1664k(BOOT1),256k(ENV),256k(ENV1),128k(KEY_CUST),5m(KERNEL),5m(KERNEL_BAK),16m(rootfs),16m(rootfs_bak),1m(factory),20m(RES),-(UBI)
```
   После этого можно продолжить загрузку
```
run bootcmd
```
#### 5. Он продолжит загрузку ядра и загрузит консоль.
Если у вас есть возможность вводить команды, значит все верно. выполните команду:
```
ubifs_mount.sh 0 /res
ubifs_mount.sh 1 /data
```
#### 6. Удалите файл с паролем
```
rm /res/passwd
```
#### 7. Создайте post_init.sh
```
mkdir -p /data/scripts; chattr -i /data/scripts/post_init.sh; echo -e "#\!/bin/sh\n\nfw_manager.sh -r\nfw_manager.sh -t -k\n" > /data/scripts/post_init.sh; chmod a+x /data/scripts/post_init.sh; chattr +i /data/scripts/post_init.sh
```
   если вы хотите включить rtsp по умолчанию:
```
mkdir -p /data/scripts; chattr -i /data/scripts/post_init.sh; echo -e "#\!/bin/sh\n\nfw_manager.sh -r\nfw_manager.sh -t -k\nrtsp &\n" > /data/scripts/post_init.sh; chmod a+x /data/scripts/post_init.sh; chattr +i /data/scripts/post_init.sh
```

Отключите USB-TTL, перезагрузите Doorbell G4. Теперь все готово.
---

#### 8. Подключитесь к камере через Telnet, порт 23, логин root, пароль пустой (возможно стоит подождать 1мин после перезагрузки)
Установите новый пароль 
```
passwd
```
#### 9. Проверьте трансляцию:
Она без пароля:
- rtsp://[your_g4_ip]:8554/360p
- rtsp://[your_g4_ip]:8554/720p
- rtsp://[your_g4_ip]:8554/1080p
