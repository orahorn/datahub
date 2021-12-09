# Информационная безопасность

## Криптография: шифрование, дайджесты, подписи

Для шифрования данных используются различные математические методы
преобразования чисел. Например, популярны медоты
[эллиптической криптографии](https://ru.wikipedia.org/wiki/%D0%AD%D0%BB%D0%BB%D0%B8%D0%BF%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F_%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F)
используемые в
[алгоритмах одностороннего преобразования](https://ru.wikipedia.org/wiki/%D0%9E%D0%B4%D0%BD%D0%BE%D1%81%D1%82%D0%BE%D1%80%D0%BE%D0%BD%D0%BD%D1%8F%D1%8F_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F).

Самый первый аспект безопасности, с которым связывается пользователь,
который хочет получить авторизованный доступ в операционную систему (ОС) -
это ввод и проверка пароля.

Долгое время в первых Unix'ах пароли принимались и сверялись
со значением из второго поля файла [/etc/passwd](https://ru.wikipedia.org/wiki//etc/passwd).
Разделители полей в файле - это двоеточия `:`.
Т.е. по сути эталонные значения паролей хранились в файлах в открытом виде.
Т.о. любой человек, который имел прямой физический доступ к компьютеру,
мог прочесть нешифрованную файловую систему (ФС) и узнать пароль.

Первым защитным действием от подобного рода незаконных вторжений в систему
было изобретение метода применение [односторонних функций](https://ru.wikipedia.org/wiki/Односторонняя_функция).

Подобные функции используются для создания значений от преобразования хеш-функций от пароля, метки времени и случайного числа. Два последних аргументов хранятся в открытом виде, а значение хеша будет соответственно меняться, даже если пользователь будет вводить на сохранение один и тот же пароль.

Среди применяемых в Unix функций для создания криптографических
хешей паролей:

* [DES](https://ru.wikipedia.org/wiki/DES) - создан в 1977 году алгоритм. Сейчас не применяется.
* [MD5](https://ru.wikipedia.org/wiki/MD5) - Message Digest 5 - придуман в 1991, опубликован в 1992 году. Известен многочисленными методами атаки. Самый известный: Метод Ван Сяоюня и Юй Хунбо (на интеллектуальном переборе). 
* [Blowfish](https://ru.wikipedia.org/wiki/Blowfish) - Разработан Брюсом Шнайером в 1993 году. Достаточно криптостоек.
* [bcrypt](https://ru.wikipedia.org/wiki/Bcrypt) - опубликован в 1999 году довольно эффективен и защищён. Может быть реализован в ПЛИС.
* [семейство SHA](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms) - с 1993 по 2015 годы.

Эти пароли записываются в файл `/etc/shadow`.



### Библиотеки шифрования: OpenSSL и LibreSSL

[OpenSSL](https://ru.wikipedia.org/wiki/OpenSSL)
- и библиотека с разными криптографическими функциями
и одноимённая утилита.

Она, например, может создавать пароли, как в файле `/etc/shadow`:

Воспользуемся субкомандой задания 
пароля [passwd](https://www.openssl.org/docs/manmaster/man1/openssl-passwd.html).

Типичный формат:

	openssl passwd -номер_алгоритма -salt соль_строка открытый_пароль

Пример, зададим два разных значения salt для одного и того же пароля, с простым MD5 шифрованием:

```
$ openssl passwd -1 -salt Yb12o hazel1%
$1$Yb12o$RH6.X7ysM2ZRQmK0wPFcH1
```

Второй вариант будет отличаться по хешу, чтобы не догадались что исходный один и тот:

```
$ openssl passwd -1 -salt N7f_ hazel1%
$1$N7f_$39Lg7wjdhExi8p/ZdKvL7/
```


Ещё пример проверки реальности пароля из shadow-файла:

```
$ openssl passwd -6 -salt jqWL53HOk2Qj/KHS 123456
$6$jqWL53HOk2Qj/KHS$/eT/4vS0SybAObc4y247HMECapAIm8Y/XJuo.jp7yuoa6gEmkpWDJsIczzBGu/XXvKOPtlSV5Vg4F53y0Jlot/
```

Есть библиотека crypt, с помощью которой можно разрабатывать приложения.
Пример написания кода:

	vim passwd1.c

с таким текстом:

```c
#include <stdio.h>
#include <crypt.h>

int main(){
        printf("hash: %s\n", crypt("123456", "jb"));
        return 0;
}
```

Компилируем:

```
$ LDLIBS=-lcrypt make passwd1
cc     passwd1.c  -lcrypt -o passwd1
```

Запускаем:

```
$ ./passwd1
hash: jbAcldPn0ogKo
```

Первые два символа вывода хеша - это та самая соль, которая вторым аргументом задавалась в функции `crypt(3)`.

Ещё пример утилиты OpenSSL для получения сертификата сайта mts.ru:

```
$ openssl s_client -connect www.mts.ru:443
CONNECTED(00000005)
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
verify return:1
depth=1 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = Thawte RSA CA 2018
verify return:1
depth=0 C = RU, L = Moscow, O = Mobile TeleSystems Public Joint Stock Company, OU = MTSPJSC, CN = mts.ru
verify return:1
---
Certificate chain
 0 s:C = RU, L = Moscow, O = Mobile TeleSystems Public Joint Stock Company, OU = MTSPJSC, CN = mts.ru
   i:C = US, O = DigiCert Inc, OU = www.digicert.com, CN = Thawte RSA CA 2018
 1 s:C = US, O = DigiCert Inc, OU = www.digicert.com, CN = Thawte RSA CA 2018
   i:C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
 2 s:C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
   i:C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIGejCCBWKgAwIBAgIQAvdLvGhEn29Qox+KNYS9yjANBgkqhkiG9w0BAQsFADBc
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3
...
```

Можно закачать тест проверки вашего SSL инструментария:

	wget https://testssl.sh -O  testssl.sh


Если есть сомнения, версию SSL можно посмотреть так:

```
$ openssl version -a
OpenSSL 1.1.1  11 Sep 2018
built on: Mon Aug 23 17:02:39 2021 UTC
platform: debian-amd64
options:  bn(64,64) rc4(16x,int) des(int) blowfish(ptr) 
compiler: gcc -fPIC -pthread -m64 -Wa,--noexecstack -Wall -Wa,--noexecstack -g -O2 -fdebug-prefix-map=/build/openssl-Flav1L/openssl-1.1.1=. -fstack-protector-strong -Wformat -Werror=format-security -DOPENSSL_USE_NODELETE -DL_ENDIAN -DOPENSSL_PIC -DOPENSSL_CPUID_OBJ -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DKECCAK1600_ASM -DRC4_ASM -DMD5_ASM -DAES_ASM -DVPAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DX25519_ASM -DPADLOCK_ASM -DPOLY1305_ASM -DNDEBUG -Wdate-time -D_FORTIFY_SOURCE=2
OPENSSLDIR: "/usr/lib/ssl"
ENGINESDIR: "/usr/lib/x86_64-linux-gnu/engines-1.1"
Seeding source: os-specific
```
Итого, что может библиотека OpenSSL:

* [SSL](https://ru.wikipedia.org/wiki/SSL) - англ. Secure Sockets Layer шифрованные сессии сетевых соединений Berlkey Socket 
* [TLS](https://ru.wikipedia.org/wiki/TLS) - англ. transport layer security - шифрование VoIP, 
* [RSA](https://ru.wikipedia.org/wiki/RSA) - аббревиатура от фамилий Rivest, Shamir и Adleman - стандарт шифрования
* [Протокол Диффи — Хеллмана](https://ru.wikipedia.org/wiki/Протокол_Диффи_—_Хеллмана) -  криптографический протокол, позволяющий двум и более сторонам получить общий секретный ключ
* [DSA](https://ru.wikipedia.org/wiki/DSA) - англ. Digital Signature Algorithm — алгоритм цифровой подписи
* [цифровые сертификаты](https://ru.wikipedia.org/wiki/Цифровой_сертификат)
* [X.509](https://ru.wikipedia.org/wiki/X.509) — стандарт ITU-T для инфраструктуры открытого ключа (англ. Public key infrastructure, PKI)

В 2014 году появилась уязвимость OpenSSL и команда проекта OpenBSD сделала форк
[LibreSSL](https://ru.wikipedia.org/wiki/LibreSSL). В нём производится чистка кода.


### Надёжное управление по шифрованному каналу SSH

ssh позволят осуществлять ассимметричное [шифрование с открытым ключом](https://ru.wikipedia.org/wiki/Криптосистема_с_открытым_ключом) .

Командой

	$ ssh-keygen

Производится формирование пар "закрытый-открытый" ключ.
Закрытый ключ содержится (желательно в зашифрованном паролем виде)
у пользователя на ФС. А открытый - копируется на те сервера,
на которые нужно будет получить доступ.
Пример:

```
$ ssh-copy-id user@10.32.0.17
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user@10.22.0.7's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'user@10.32.0.17'"
and check to make sure that only the key(s) you wanted were added.
```

Видим, что соединяемся с сервером по адресу 10.22.0.7 и копируется туда открытый ключ.
Дальнейший вход на сервер будет без пароля:

	$ ssh user@10.32.0.17

Сервер будет шифровать трафик пользователю открытым ключом, а пользователь - расшифровывать его закрытым
(который есть только у него). В этом и есть смысл одностороннего преобразования математических функций.

SSH полностью шифрует не только взаимодействие с удалёнными хостами
по командной строке, но и трафик приложений X-Window.
Пример:

	ssh -X 40.3.24.64

Можно будет запускать на удалённой машине графические приложения 
и они будут взаимодействовать с вашим локальным X-сервером. 
Т.о. можно даже вести (но не рекомендуется) разработку на production-сервере.

SSH используется как доверенный транспорт при коллективной разработке
в системе git.
Например, если есть регистрация на GitHub-сервисе, то нужно
добавить публичный ключ из локального файла (например, 
`$HOME/.ssh/id_rsa.pub`) по адресу: https://github.com/settings/keys .

Проверяем, соединение по SSH:

```
ssh -T git@github.com
Hi пользователь! You've successfully authenticated, but GitHub does not provide shell access.
```
Потом в выделенный каталог, можно клонировать проекты с GitHub.
Пример для https://github.com/пользователь/проект:

	git clone git@github.com:пользователь/проект.git

Иногда для проталкивания на Git изменений кода
приходится по его совету сначала делать:

	git push --set-upstream origin main

Только потом работает `git push` (естественно, после `git commit`) .


### Средство конфиденциального обмена данными PGP и GnuPG

В 1991 году [Филипп Циммерман](https://ru.wikipedia.org/wiki/%D0%A6%D0%B8%D0%BC%D0%BC%D0%B5%D1%80%D0%BC%D0%B0%D0%BD,_%D0%A4%D0%B8%D0%BB%D0%B8%D0%BF%D0%BF) в США создал PGP - Pretty Good Privacy.
Программа позволяла шифровать и подписывать контент с помощью
специальных криптографических алгоритмов.
Благодаря чему стало возможно формирование доверенных сетей
между корреспондентами (абонентами), пересылку сообщений и
файлов по открытым каналам, без доступа к ним третьих лиц.
Экспортные ограничения США мешали распространению программы и
разных технологий связанных с криптографией за её пределами
министерством торговли.
Но Филипп опубликовал алгоритм в книге, а публикация книг была разрешена
поправкой коснтитуции США. 
Тогда потом с 1993 по 1996 год было уголовное преследование Циммермана.

В 1997 году появился стандарт [OpenPGP](https://www.openpgp.org/about/)
как документы RFC 4880:
OpenPGP Message Format; RFC 5581: The Camellia Cipher in OpenPGP;
RFC 6637: Elliptic Curve Cryptography (ECC) in OpenPGP.

В 1999 году в рамках проекта GNU появился продукт
[GnuPG](https://en.wikipedia.org/wiki/GNU_Privacy_Guard) - 
GNU Privacy Guard.
Это основное средство для коллективной разработки среди
энтузиастов свободного программного обеспечения. Пользоваться
программой не так сложно, как может показаться. Прежде всего
производится генерация ключа:

	gpg --gen-key

Просмотр ключей:

	gpg --list-public-keys


Поиск отпечатка привязанного к почте:

	$ gpg --fingerprint <адрес_электронной_почты>

Также ключ можно опубликовать на сервере.
Для этого нужно указать идентификатор отпечатка.
Например:

	gpg --send-keys --keyserver keyserver.ubuntu.com отпечаток

Потом можно с этого же сервера запросить зарегистрированный ключ.
Пример из той же серии:

	gpg --keyserver keyserver.ubuntu.com --recv-keys отпечаток



Также возможна защищённая (сокрытая) передача файлов 
по недоверенным каналам.

Пример шифрования файла, допустим с именем `list.txt` в
зашифрованный файл `list.txt.gpg` :

	gpg --symmetric list.txt

Расшифровка файла list.txt.gpg (на стандартный вывод):

	gpg --decrypt list.txt.gpg





### Защита информации на уровне ОС: SELinux, RBAC  и другие


SELinux - основное средство защиты и безопасности, в Fedora, RHEL, CentOS...

Есть параметры ядра при загрузке.
Конфигурационный файл:

	/etc/selinux/config

Посмотреть режим работы подсистемы безопасности SELinux:

	getenforce

Существуют три вида режима работы:

* *Enforce* - полностью включённый режим, когда нарушения правил блокируют действие субъектов над объектами
* *Permissive* - разрешаются все действия, но нарушения правил фиксируются в журнале ОС (syslog).
* *Disabled* - SELinux полностью отключён и ничего не проверяется.

В SELinux используется следующий формат маркировки:

	пользователь:роль:тип:(уровень)

Смотреть контекстные метки:

```
ls -Z
ps -Z
```

Смотреть логические булевы переменные:

	getsebool <-a|переменные>

Изменение контекста SELinux для объектов файловой системы:

	chcon

Пример:

	chcon -t httpd_sys_content_t <каталог>

Управление и конфигурирование политик SELinux:

	semanage

Пример добавим файловый контекст для доступа к web серверу (apache httpd):

	semanage fcontext -a -t httpd_sys_content_t '/var/www/html(/.*)?

Восстановление контекста SELinux для элемента ФС <каталог>:

	restorecon -Rv <каталог>

Программа `sealert` используется для поиска неисправности при работе с SELinux.

Сетевая безопасность основана на том, чтобы регулировать доступ по портам
TCP. Например, чтобы открыть порт 80 (для web-сервера):

	firewall-cmd --zone=public --add-port=80/tcp --permanent


В других системах есть управление доступом на основе ролей
(англ. Role Based Access Control, RBAC). Это когда пользователям
назначаются роли для выполнения привилигированных операций.
Например, этим может управлять команда `sudo` в Debian и Ubuntu. 

Более сложное управление происходит при
[мандатном доступе](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D0%BD%D0%B4%D0%B0%D1%82%D0%BD%D0%BE%D0%B5_%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%BE%D0%BC) - выдаётся вре́менная
метка на доступ конкретного субъекта (пользователя, процесса) к объекту.

Подобные доступы связаны с понятием избирательного управления доступом (англ. discretionary access control, DAC). Это когда есть матрица доступа субъектов
к объекту или списки доступа [ACL](https://ru.wikipedia.org/wiki/ACL).


### ЭЦП для подписи пакетов ОС

В наши дни ЭЦП редко используется для переписки.
Но тем не менее все пакеты ОС выложенные в сеть 
в т.н. сгруппированном виде в репозитории - имеют подпись.

На системах RHEL/CentOS/Fedora можно проверить командой:

	rpm -K имя_файла_с_пакетом.rpm




## Методы повышения безопасности

В современных Linux дистрибутивах используют средство System-D.
Оно написано на языке Си.
Считается, что в отличае от старого средства SysV, написанного на скриптах - оно более безопасно.
SystemD позволяет:

* запускать сервисы в изолированных окружениях, чтобы злоумышленник не мог навредить всей системе
* Интегрируют подсистемы безопасности:
	- AppArmor - для Debian/Ubuntu-основанных дистрибутивов
	- SELinux - для Fedora/RHEL/CentOS-наследованных дистрибутивах
* Централизует управление: т.е. нет разбросанных точек для потенциальных атак.
* Упрощает разработчику реализацию низкоуровневой защиты. Например, можно писать код на Python
* Нет необходимости иметь привелегии суперпользователя для разных операций. Напрмир, пользователю можно разрешить работать с модулем монтирования носителей в ФС.

При разработке сервиса сначала пишется файл-дескриптор модуля.
Потом для актуализации конфигурации даётся команда:

	# systemctl daemon-reload

Потом специальной командой можно посмотреть журнал:

	$ journalctl

Пример сессии:

```
$ systemd-run whoami
Running as unit: run-u308.service

$ journalctl -u run-u308.service
-- Logs begin at Tue 2021-03-30 10:00:25 MSK, end at Tue 2021-10-12 14:00:12 MSK. --
окт 12 13:57:45 c003m1cpc512 systemd[1]: Started /usr/bin/whoami.
окт 12 13:57:45 c003m1cpc512 whoami[15845]: root

```

## Стеганография

## Источники

* [Криптография](https://ru.wikipedia.org/wiki/%D0%9A%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F)
* [Шифрование](https://ru.wikipedia.org/wiki/%D0%A8%D0%B8%D1%84%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)
* [Ключ (криптография)](https://ru.wikipedia.org/wiki/%D0%9A%D0%BB%D1%8E%D1%87_(%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F))
* [Эллиптическая криптография](https://ru.wikipedia.org/wiki/%D0%AD%D0%BB%D0%BB%D0%B8%D0%BF%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F_%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F)
* [Симметричные криптосистемы](https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D0%BC%D0%BC%D0%B5%D1%82%D1%80%D0%B8%D1%87%D0%BD%D1%8B%D0%B5_%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B)
* [Целостность информации](https://ru.wikipedia.org/wiki/%D0%A6%D0%B5%D0%BB%D0%BE%D1%81%D1%82%D0%BD%D0%BE%D1%81%D1%82%D1%8C_%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D0%B8)
* [Конфиденциальность](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BD%D1%84%D0%B8%D0%B4%D0%B5%D0%BD%D1%86%D0%B8%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D1%81%D1%82%D1%8C)
* [Аутентификация](https://ru.wikipedia.org/wiki/%D0%90%D1%83%D1%82%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F)
* [Стеганография](https://ru.wikipedia.org/wiki/%D0%A1%D1%82%D0%B5%D0%B3%D0%B0%D0%BD%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F)
* [Стеганография в цифровых изображениях](https://ru.wikipedia.org/wiki/%D0%A1%D1%82%D0%B5%D0%B3%D0%B0%D0%BD%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F_%D0%B2_%D1%86%D0%B8%D1%84%D1%80%D0%BE%D0%B2%D1%8B%D1%85_%D0%B8%D0%B7%D0%BE%D0%B1%D1%80%D0%B0%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F%D1%85)
* [LibreSSL: очищенная версия OpenSSL (проект OpenBSD)](https://habr.com/ru/post/220367/) ; 22 апреля 2014 в 13:23 - 
* [Ещё раз об OpenSSL](https://habr.com/ru/post/417069/) ; 13 июля 2018 в 10:25
* [Pretty Good Privacy](https://en.wikipedia.org/wiki/Pretty_Good_Privacy)


