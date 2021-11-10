# Журналирование событий и ведение статистики работы ИУС

Опытный системный администратор первым делом на только что установленом сервере,
иногда даже до его полной настройки включает запись журналов и файлов со
статистикой.
Это нужно для своевременного понимания характеристик работы системы:

* чтобы не пропустить важное событие на сервере или сбой
* мониторинг безопасности и предотвращение неавторизованных доступов и вторжений в систему
* регулировка и распределение нагрузки на части системы уже на этапе установки
* наблюдение за работой сервера удалённо и последующего его мониторинга


## Сервисы ОС по записи событий

### syslog

По сути регистрацию событий составляет специальный серис,
который в классическом Unix реализовывался syslogd, заменённый частью systemd (см. далее)
в наши дни.
Также в пакет bsdutils входит программа logger которая в файлах журнала оставляет записи.

Пример будет работать на системах старого типа:

Включаем мониторинг всех записей основного системного журнала:

	$ tail -0f syslog

В другом терминале отправляем сообщение в журнал:

	$ logger hello

В терминале с мониторинга выходит сообщение:

```
$ tail -0f syslog
Nov  3 23:21:02 brix dron: hello
```

Здесь в строке: дата и время события, имя хоста, пользователь ОС который сгенерировал событие через
syslog API и через двоеточие (:), собственно, само сообщение (в данном примере hello).

Данную технику можно использовать для отладки скриптов оболочки ОС, либо ведения журналов.

В примере показано и важно понимать, что syslog - это не только ПО, но и сетевой протокол.
Он регламентирует с помощью разных документов формат передачи событий по разным машинам на сети.
Есть специально выделенные сервера, которых от других машин централизованно хранят журналы,
события от которых можно сверять и корреляция может сказать, что в целом произошло на сети...

В последних системах Debian/Ubuntu по умолчанию установлено ПО [rsyslog](https://www.rsyslog.com/)
в качестве подсистемы регистрации событий в файлы. Оно запускается как сервис [SystemD](https://ru.wikipedia.org/wiki/Systemd).

Посмотреть состояние rsyslog можно, например, так:

```
$ systemctl status rsyslog.service
● rsyslog.service - System Logging Service
   Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2021-11-07 14:48:56 MSK; 4h 52min ago
     Docs: man:rsyslogd(8)
           http://www.rsyslog.com/doc/
 Main PID: 895 (rsyslogd)
    Tasks: 4 (limit: 4915)
   CGroup: /system.slice/rsyslog.service
           └─895 /usr/sbin/rsyslogd -n

ноя 07 14:48:56 brix.localdomain systemd[1]: Starting System Logging Service...
ноя 07 14:48:56 brix.localdomain rsyslogd[895]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3) from systemd.
ноя 07 14:48:56 brix.localdomain rsyslogd[895]: rsyslogd's groupid changed to 106
ноя 07 14:48:56 brix.localdomain rsyslogd[895]: rsyslogd's userid changed to 102
ноя 07 14:48:56 brix.localdomain rsyslogd[895]:  [origin software="rsyslogd" swVersion="8.32.0" x-pid="895" x-info="http://www.rsys
ноя 07 14:48:56 brix.localdomain systemd[1]: Started System Logging Service.
ноя 07 14:53:56 brix.localdomain rsyslogd[895]:  [origin software="rsyslogd" swVersion="8.32.0" x-pid="895" x-info="http://www.rsys
...
```

Конфигурация находится в файлах  `/etc/rsyslog.d/*.conf` и `/etc/rsyslog.conf`.
Там определяются сообщения от каких подсистем и с какой важностью пишутся в какие файлы.
Периодически эти файлы меняют своё имя.
Старые файлы могут упаковываться и обязательно должны автоматически удаляться
(иначе будет переполнено дисковое пространство доступоное файловой системе).
Эту функцию на себя берёт ПО [logrotate](https://github.com/logrotate/logrotate) .
Эта служба запускается [cron](https://ru.wikipedia.org/wiki/Cron) сервисом.
В файле `/etc/cron.daily/logrotate` записан скрипт,
который запускает программу logrotate для каждого файла журнала.
Файл `/etc/logrotate.d/rsyslog` описывает файлы журналов, которые вырабатывает rsyslog и что с ними делать:

* кого сжимать
* сколько последних серий файлов держать
* какие скрипты запускать после ротации логов

Можно организовать централизованный сбор сообщений на одном **сервере** по сети от **клиентов**.
В файле на **сервере** `/etc/rsyslog.conf` раскомментируем две строки:

```
$ModLoad imudp
$UDPServerRun 514
```

или такие

```
# provides UDP syslog reception
#module(load="imudp")
#input(type="imudp" port="514")
```


Открываем UDP порт 514 для входящих на сервере соединений:

	iptables -I INPUT 1 -p UDP --destination-port 514 -j ACCEPT

Проверяем состояние файервола:

```
$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere             udp dpt:syslog

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination       
```

Чтобы сервис воспринял конфигурацию, перезапускаем его:

	systemctl restart rsyslog


В `/etc/rsyslog.conf` файле на **клиенте**  добавляем строку правило маршрутизации сообщений syslog:

	*.info @<адрес_сервера>

Также можно создавать файлы в каталоге :
Чтобы  воспринялась конфигурация, перезапускаем rsyslog:

	systemctl restart rsyslog

Проверяем доставку сообщений.
На сервере очищаем консоль и контроллируем сообщения попадающие в лог:

```
clear
tail -0f /var/log/messages
```

На клиенте формируем в журнал сообщение:

	logger "Привет с рабочей станции"

Смотрим всплыло ли оно на консоли сервера.

Литература: *Red Hat Enterprise Linux 7 System Administrator’s Guide, Chapter 18: Viewing and Managing Log Files*.



### syslogng

В качестве альтернативы стандартному syslog или rsyslog может быть поставлен
[syslog-ng](https://en.wikipedia.org/wiki/Syslog-ng).
У него много плагинов и преобразователей данных в разные форматы для мониторинга в реальном времени.


###  Подсистема systemd и средство journalctl

В отличае от классических текстовых журнальных файлов (логов)
журналы [systemd](https://ru.wikipedia.org/wiki/Systemd) представляют из себя
систему записей в двоичном структурированном формате.
Располагаются они каталоге `/var/log/journal`.
Для их ротации и автоматической очистки не требуется отдельное внешнее ПО.
Файлы журнала удобно фильтровать по заданным критериям (см. далее).
Централизованный сервис ведения журналов
[journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html)
может предоставлять данные в формате JSON.

Просмотр сообщений с приоритетом серьёзности (важности) наперёд заданной или больше
(в данном случае критической - crit):

```
journalctl -p crit
...
ноя 03 17:57:27 c003m1cpc512 postfix/sendmail[12179]: fatal: open /etc/postfix/main.cf: No such file or directory
ноя 03 19:55:51 c003m1cpc512 gnome-session-binary[1546]: CRITICAL: We failed, but the fail whale is dead. Sorry....
-- Reboot --
ноя 08 19:15:17 c003m1cpc512 gnome-session-binary[2115]: CRITICAL: We failed, but the fail whale is dead. Sorry....
-- Reboot --
ноя 09 19:02:30 c003m1cpc512 gnome-session-binary[1926]: CRITICAL: We failed, but the fail whale is dead. Sorry....
```

Мониторинг за вновь поступающими сообщениями важности от info:

```
$ journalctl -n0 -p info -f
-- Logs begin at Tue 2021-03-30 10:00:25 MSK. --
ноя 10 15:58:49 c003m1cpc512 sudo[30719]:     dron : TTY=pts/1 ; PWD=/home/dron ; USER=root ; COMMAND=/usr/bin/apt update
ноя 10 15:58:49 c003m1cpc512 sudo[30719]: pam_unix(sudo:session): session opened for user root by (uid=0)
ноя 10 15:58:59 c003m1cpc512 sudo[30719]: pam_unix(sudo:session): session closed for user root
ноя 10 15:59:00 c003m1cpc512 sudo[31102]:     dron : TTY=pts/1 ; PWD=/home/dron ; USER=root ; COMMAND=/usr/bin/apt full-upgrade
ноя 10 15:59:00 c003m1cpc512 sudo[31102]: pam_unix(sudo:session): session opened for user root by (uid=0)
ноя 10 15:59:00 c003m1cpc512 dbus-daemon[906]: [system] Activating via systemd: service name='org.freedesktop.fwupd' unit='fwupd.service' requested by ':1.124' (uid=2767 pid=3493 comm="/usr/bin/gnome-software --gapplication-service " label="unconfined")
ноя 10 15:59:00 c003m1cpc512 gnome-software[3493]: Only 0 apps for recent list, hiding
ноя 10 15:59:00 c003m1cpc512 systemd[1]: Starting Firmware update daemon...
ноя 10 15:59:00 c003m1cpc512 fwupd[31115]: 12:59:00:0384 FuPluginUefi         kernel efivars support missing: /sys/firmware/efi/efivars
ноя 10 15:59:00 c003m1cpc512 dbus-daemon[906]: [system] Successfully activated service 'org.freedesktop.fwupd'
ноя 10 15:59:00 c003m1cpc512 systemd[1]: Started Firmware update daemon.

```

[Конфигурация journald](https://www.freedesktop.org/software/systemd/man/journald.conf.html)
находится в файле `/etc/systemd/journald.conf`.

Просматривать журнал можно в диаппазоне времени. Формат такой:

	journalctl [--since "ГГГГ-ММ-ДД  ЧЧ:МИН:СС" [--until ...]]

Вместо даты/времени допустимы сокращения:

* yesterday
* today
* tomorrow

Если время не задано, то подразумеваются нули.
Если выражение не задано - то подразумевается сегодня.
Пример вывода всех сообщений с обеда сегодняшнего дня:

	journalctl --since 12:00 --until tomorrow

сообщения можно листать, как в программе less.

Или ещё пример просмотра сообщений от 6 ноября (меня не было дома) до вчерашнего дня:

```
$ journalctl --since 2021-11-06 --until yesterday
-- Logs begin at Tue 2021-03-16 11:27:43 MSK, end at Wed 2021-11-10 18:35:01 MSK. --
ноя 07 14:48:54 brix.localdomain kernel: microcode: microcode updated early to revision 0x21, date = 2019-02-13
ноя 07 14:48:54 brix.localdomain kernel: Linux version 5.4.0-89-generic (buildd@lgw01-amd64-034) (gcc version 7.5.0 (Ubuntu 7.5.0-3
ноя 07 14:48:54 brix.localdomain kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-5.4.0-89-generic root=UUID=ba13ca3b-094c-4abb-96f0-
ноя 07 14:48:54 brix.localdomain kernel: KERNEL supported cpus:
ноя 07 14:48:54 brix.localdomain kernel:   Intel GenuineIntel
ноя 07 14:48:54 brix.localdomain kernel:   AMD AuthenticAMD
ноя 07 14:48:54 brix.localdomain kernel:   Hygon HygonGenuine
ноя 07 14:48:54 brix.localdomain kernel:   Centaur CentaurHauls
ноя 07 14:48:54 brix.localdomain kernel:   zhaoxin   Shanghai  
ноя 07 14:48:54 brix.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
ноя 07 14:48:54 brix.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
ноя 07 14:48:54 brix.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
ноя 07 14:48:54 brix.localdomain kernel: x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
ноя 07 14:48:54 brix.localdomain kernel: x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'standard' format.
ноя 07 14:48:54 brix.localdomain kernel: BIOS-provided physical RAM map:
ноя 07 14:48:54 brix.localdomain kernel: BIOS-e820: [mem 0x0000000000000000-0x000000000009d7ff] usable
ноя 07 14:48:54 brix.localdomain kernel: BIOS-e820: [mem 0x000000000009d800-0x000000000009ffff] reserved
...
```

Как видим, 6 ноября ни одной записи не было, т.е. компьютер реально ни кем не включался.


Подробный вывод сообщений:

	journalctl -o verbose

Можно фильтровать по критериям.
Пример, фильтации сообщений от процесса DHCP клиента:

```
$ pgrep -l dhc
1367 dhclient
```

здась 1367 - это номер процесса в ОС.
смотрим сквозь фильтр:

```
$ journalctl _PID=1367
...
ноя 08 21:06:19 brix.localdomain gsd-color[1367]: could not find device: property match 'XRANDR_name'='HDMI-1' does not exist
ноя 08 21:06:58 brix.localdomain gsd-color[1367]: failed to set screen _ICC_PROFILE: Не удалось открыть файл «/home/dron/.local/sha
ноя 08 21:07:00 brix.localdomain gsd-color[1367]: failed to set screen _ICC_PROFILE: Не удалось открыть файл «/home/dron/.local/sha
ноя 08 23:26:35 brix.localdomain gsd-color[1367]: failed to connect to device: Failed to connect to missing device /org/freedesktop
ноя 08 23:26:35 brix.localdomain gsd-color[1367]: gsd-color: Fatal IO error 11 (Ресурс временно недоступен) on X server :1024.
-- Reboot --
ноя 10 17:50:27 brix.localdomain dhclient[1367]: DHCPREQUEST of 192.168.3.174 on enp2s0 to 255.255.255.255 port 67 (xid=0x1dc36ea8)
ноя 10 17:50:27 brix.localdomain dhclient[1367]: DHCPACK of 192.168.3.174 from 192.168.3.1
ноя 10 17:50:27 brix.localdomain dhclient[1367]: bound to 192.168.3.174 -- renewal in 37822 seconds.
```

Сегодня 10 ноября под номером 1367 значится процесс dhclient , а позавчера gsd-color.

Пример просмотра сообщений от пользователя ОС с UID равным 1000:

```
$ journalctl _UID=1000
-- Logs begin at Tue 2021-03-16 11:27:43 MSK, end at Wed 2021-11-10 18:55:01 MSK. --
мар 16 11:27:57 brix systemd[1557]: Listening on GnuPG cryptographic agent and passphrase cache (access for web browsers).
мар 16 11:27:57 brix systemd[1557]: Starting D-Bus User Message Bus Socket.
мар 16 11:27:57 brix systemd[1557]: Started Pending report trigger for Ubuntu Report.
мар 16 11:27:57 brix systemd[1557]: Listening on GnuPG network certificate management daemon.
мар 16 11:27:57 brix systemd[1557]: Listening on GnuPG cryptographic agent (ssh-agent emulation).
мар 16 11:27:57 brix systemd[1557]: Reached target Paths.
мар 16 11:27:57 brix systemd[1557]: Reached target Timers.
мар 16 11:27:57 brix systemd[1557]: Listening on GnuPG cryptographic agent and passphrase cache (restricted).
мар 16 11:27:57 brix systemd[1557]: Listening on GnuPG cryptographic agent and passphrase cache.
мар 16 11:27:57 brix systemd[1557]: Listening on REST API socket for snapd user session agent.
мар 16 11:27:57 brix systemd[1557]: Listening on D-Bus User Message Bus Socket.
...
```

Пример выбора записей активности сервиса времени с 9 утра:

```
$ journalctl --since 9:00:00 _SYSTEMD_UNIT="systemd-timesyncd.service"
-- Logs begin at Tue 2021-03-16 11:27:43 MSK, end at Wed 2021-11-10 19:05:01 MSK. --
ноя 10 17:50:53 brix.localdomain systemd-timesyncd[699]: Synchronized to time server 91.189.89.198:123 (ntp.ubuntu.com).
```

Просмотр активности сервиса cron с начала сегодняшнего дня:

```
$ journalctl -u cron.service --since today
-- Logs begin at Tue 2021-03-16 11:27:43 MSK, end at Wed 2021-11-10 19:25:01 MSK. --
ноя 10 17:50:23 brix.localdomain systemd[1]: Started Regular background program processing daemon.
ноя 10 17:50:23 brix.localdomain cron[916]: (CRON) INFO (pidfile fd = 3)
ноя 10 17:50:23 brix.localdomain cron[916]: (CRON) INFO (Running @reboot jobs)
ноя 10 17:55:01 brix.localdomain CRON[5574]: pam_unix(cron:session): session opened for user root by (uid=0)
ноя 10 17:55:01 brix.localdomain CRON[5575]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
ноя 10 17:55:01 brix.localdomain CRON[5574]: pam_unix(cron:session): session closed for user root
ноя 10 18:05:01 brix.localdomain CRON[6124]: pam_unix(cron:session): session opened for user root by (uid=0)
ноя 10 18:05:01 brix.localdomain CRON[6125]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
ноя 10 18:05:01 brix.localdomain CRON[6124]: pam_unix(cron:session): session closed for user root
...
```


## Библиотеки журналирования приложений

### POSIX

В стандартной библиотеке C стандарт POSIX
определяет заголовочный файл:

```c
#include <syslog.h>
```

Он определяет функции и константы для ведения журналов
в системе приложениям. Запишем в файл `syslog1.c` такой код:

```c
#include <syslog.h>

int main(){
        openlog(0, LOG_CONS|LOG_PERROR, LOG_LOCAL6);
        syslog(LOG_NOTICE, "Program started");
        closelog();
        return 0;
}
```

Константа `LOG_PERROR` будет дублировать сообщения от вызова `syslog()` на стандартный поток
ввода-вывода ошибок. `LOG_CONS` будет выводит данные на консоль, в случае даже не запуска сервиса (r)syslog(ng)...

Компилируем:

```
$ make syslog1
cc     syslog1.c   -o syslog1
```

После запуска:

```
$ ./syslog1
syslog1: Program started
```

В файле `/var/log/syslog` появляется запись:

	Nov  7 21:37:47 brix syslog1: Program started


### systemd journal API

Нативный прикладной интерфейс программирования написан для языка C
и библиотеки поддержки нужно устанавливать в Debian/Ubuntu:

	sudo apt install libsystemd-dev

Основный заголовочный файл `/usr/include/systemd/sd-journal.h`.

Пишем программу на Си в файл `sdj1.c`:

```c
#include <systemd/sd-journal.h>
#include <unistd.h>

int main() {
        sd_journal_print(LOG_NOTICE, "Hi. My PID is %lu", (unsigned long)getpid());
        return 0;
}
```

с единственной функцией 
[sd_journal_print()](https://www.freedesktop.org/software/systemd/man/sd_journal_print.html)


Собираем:

	LDLIBS=-lsystemd make sdj1

Запускаем:

	./sdj1

Смотрим результат в последней строчке вывода:

```
$ journalctl --since today
...
ноя 10 21:17:01 brix.localdomain CRON[14196]: (root) CMD (   cd / && run-parts --report /etc/cron.hourly)
ноя 10 21:17:01 brix.localdomain CRON[14195]: pam_unix(cron:session): session closed for user root
ноя 10 21:17:42 brix.localdomain sdj1[14205]: Hi. My PID is 14205
```

Вообще каталоги заголовочных файлов и имена разделяемых библиотек 
можно смотреть командой:

```
$ pkg-config --cflags --libs libsystemd
-lsystemd
```

В  данной системе каталог находится в стандартном месте, поэтому не печатается - только опция библиотеки.


## Анализаторы логов

### Статистика по логам

### logtool

## системы ведения статистики

### sysstat, sar, ksar


## Прикладные сервисы журналирования проекта Apache

Сервисы [Apache logging](https://logging.apache.org/) предназначены
для предоставления однообразных интерфейсов программирования к разным языкам программирования (ЯП)
для сохранения журналов регистрации событий, генерируемых приложениями.
Объединяет эти библиотеки-сервисы журналирования то, что они используют
для настроек конфигурационне XML-файлы, в отличае от Unix'овых `/etc/syslog.conf`.


### Java

Всё началось  с программного обеспечения (ПО) [log4j](https://logging.apache.org/log4j/) для языка проагрммирования Java.




### C++

**log4c++**

**log4cpp**

* [Apache log4cxx](https://logging.apache.org/log4cxx/)

### C#

* Apache [log4net](https://ru.wikipedia.org/wiki/Apache_log4net)

### Классический Си

Сервис [log4c](http://log4c.sourceforge.net/) физически не включён в Apache инкубатор ПО,
но использует ту же идеологию, что и Log4J.

### Perl

## Источники информации

* [Файл регистрации](https://ru.wikipedia.org/wiki/%D0%A4%D0%B0%D0%B9%D0%BB_%D1%80%D0%B5%D0%B3%D0%B8%D1%81%D1%82%D1%80%D0%B0%D1%86%D0%B8%D0%B8)
* [Использование journalctl для просмотра и анализа логов: подробный гайд](https://habr.com/ru/company/ruvds/blog/533918/)
* [Journal File Format](https://www.freedesktop.org/wiki/Software/systemd/journal-files/)


