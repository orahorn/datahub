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


