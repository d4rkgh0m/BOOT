# ДЗ: Загрузка системы_Работа с загрузчиком
-----------------------------------------------------------------------
### Домашнее задание

    1. Попасть в систему без пароля несколькими способами
    2. Установить систему с LVM, после чего переименовать VG
    3. Добавить модуль в initrd

## Попасть в систему без пароля несколькими способами
```
    Для получения доступа необходимо открыть GUI VirtualBox (или другой системы виртуализации), 
    запустить виртуальную машину и при выборе ядра для загрузки нажать e - в данном контексте edit. 
    Попадаем в окно, где мы можем изменить параметры загрузки:
```
```
    Способ 1. init=/bin/sh
- В конце строки, начинающейся с linux16, добавляем init=/bin/sh и нажимаем сtrl-x для загрузки в систему
- В целом на этом все, Вы попали в систему. Но есть один нюанс. Рутовая файловая
система при этом монтируется в режиме Read-Only.
```
<details>![Screenshot from 2024-04-11 16-30-47](https://github.com/d4rkgh0m/BOOT/assets/120186195/28cb41d9-450c-456a-a9c8-337bdfe1ec37)</details>
|| ![Screenshot from 2024-04-11 16-32-44](https://github.com/d4rkgh0m/BOOT/assets/120186195/bb37ac9e-9dce-437e-bca5-f67bf46bd6df) ||
```
Если вы хотите перемонтировать ее в режим Read-Write, можно воспользоваться командой:
[root@otuslinux ~]# mount -o remount,rw /
```
![Screenshot from 2024-04-11 16-32-44](https://github.com/d4rkgh0m/BOOT/assets/120186195/246d074b-b97c-4eb9-908d-ce62592dfc96)
```
- После чего можно убедиться, записав данные в любой файл или прочитав вывод
команды:
[root@otuslinux ~]# mount | grep root
```   
  



