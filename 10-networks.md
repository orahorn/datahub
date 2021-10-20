# Сети и телекоммуникации

Долгое время обмен информацией между компьютерами осуществлялся последовательной передачей данных одним каналом. Сейчас посредником между одним или несколькими компьютерами является сеть. Тем не менее - долгое время в большей, а в наши дни - в меньшей степени это взаимодействие с дополнительными передающими устройствами (например, модемами), происходит по последовательной линии связи.
Часто интерфейсом к этой линии служит оборудования стандарта RS-232.

## Библиотеки и фреймворки

### Управление терминалом с помощью функций семейства termios

В Unix для управление терминалом, служит набор средств и функций, определённых
в заголовочном файле  `termios.h`.

### Классическая сетевая библиотека Berkley Sockets

Для работы со стеком протоколов TCP/IP после разработки Unix и языка Си появился
в университете Berkly набор функций для передачи сообщений в сеть в формате т.н.
"гнёзд" (sockets) подобно тому, как записывают данные в файл (потоком, через дескриптор).

Сокеты описаны в библиотеке, которую нужно подключить серез заголовочный файл:

```c
#include <sys/socket.h>
```

Но также требуется ещё ряд библиотек.
В частности описание типов сокетов:

```c
#include <sys/types.h>
```

Простейший сервер, который будет принимать входящие сетевые соединения будет выглядеть так:

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {

        /* create the server socket */
        int server_socket;
        server_socket = socket(AF_INET, SOCK_STREAM, 0);

        /* define the server address */
        struct sockaddr_in server_address;
        server_address.sin_family = AF_INET;
        server_address.sin_port = htons(9002);
        server_address.sin_addr.s_addr = INADDR_ANY;

        /* bind the socket to our specified IP and port */
        bind(server_socket, (struct sockaddr*) &server_address, sizeof(server_address));

        /* second agrument is a backlog - how many connections can be waiting for this socket simultaneously */
        listen(server_socket, 5);
        int client_socket;
        client_socket = accept(server_socket, NULL, NULL);

        /* send the message */
        char server_message[256] = "You have reached the server!";
        send(client_socket, server_message, sizeof(server_message), 0);

        /* close the socket */
        close(server_socket);
        return 0;
}

```

Здесь для понимаемости опущена обработка ошибок.
Сборка сервера:

```
$ make srv2
cc     srv2.c   -o srv2
```

Никаких дополнительных библиотек компоновать не нужно.
Запускаем сервер в одном терминале:

```
$ ./srv2

```

Он слушает порт номер 9002.
В другом терминале на этот TCP порт запрос и получаем в ответ сообщение:

```
$ netcat  localhost 9002
You have reached the server!
```


Обратная задача не намного сложнее.
Запускаем на порту 8080 импровизированный сервер:

```
$ netcat -l localhost 8080

```

Пишем в файл `clnt2.c` код приложения-клиента:

```c
/* Client side C/C++ program to demonstrate Socket programming */
#include <stdio.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#define PORT 8080

int main(int argc, char const *argv[])
{
    int sock = 0, valread;
    struct sockaddr_in serv_addr;
    char *hello = "Hello from client";
    char buffer[1024] = {0};
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0)
    {
        printf("\n Socket creation error \n");
        return -1;
    }

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);

    // Convert IPv4 and IPv6 addresses from text to binary form
    if(inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)<=0)
    {
        printf("\nInvalid address/ Address not supported \n");
        return -1;
    }

    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0)
    {
        printf("\nConnection Failed \n");
        return -1;
    }
    send(sock , hello , strlen(hello) , 0 );
                                                                                                                 1,1       Наверху
    printf("Hello message sent\n");
    valread = read( sock , buffer, 1024);
    printf("%s\n",buffer );
    return 0;
}

```

Также его собираем без доп. библиотек:

```
$ make clnt2
cc     clnt2.c   -o clnt2
```

Запускаем и видим, что сообщение отослано:

```
./clnt2
Hello message sent
```

Это же сообщение в другом терминале должен вывести сервер и можно там набрать своё,
которое на клиенте примет функция `read()`, а распечатает `printf()`.


### Библиотека ADAPTIVE Communication Environment

Для разработки объектно ориентированного кода
на языке C++ [Дугласом Шмидтом](http://www.dre.vanderbilt.edu/~schmidt/)
была создана библиотека ADAPTIVE Communication Environment ([ACE](http://www.dre.vanderbilt.edu/~schmidt/ACE.html)).
В некоторых частях - это обёртка к Berkley Sockets.
Библиотека ставит целью создавать распределённые, кроссплатформенные приложения
в разных ОС, например [FreeBSD](https://www.freshports.org/devel/ace), Linux, macOS, любом другом Unix 
и даже Windows (есть возможность это сделать в среде [Mingw](https://stackoverflow.com/questions/42763665/building-adaptive-communication-environment-ace-using-mingw)).
Есть интеграция с передачей информации в формате XML, несколько интеграций с GUI (в том числе FLTK).

Коммерческую поддержку и заказную разработку на базе библиотеки ведёт компания [Riverace](https://www.riverace.com/).


Список с достаточным количеством пакетов Ubuntu/Debian можно получить, выполнив команду:

	apt search libace

Рекомендуется начать освоение с установки пакета с документацией по библиотеке:

	sudo apt install libace-doc

## Средства анализа и управления сетевым трафиком

###  tcpdump

Утилита tcpdump отображает состояние трафика проходящего через сетевые интерфейсы.

### wireshark/tshark


### netcat

netcat - это простейшее сетевое приложение - которое может аналогично утилите копирования cat, отправлять или принимать
по сети данные, файлы и прочие потоки. Очень удобно для отладки приложений.

### ntop


### Пакет iproute2

## Уровень представления данных: ещё раз о кодировках символов

Передача естественных языков происходит посимвольно: т.е. символ становится минимальной передаваемой единицей информацией.
Традиционно первой кодировкой была ASCII (7-бит), она умещалась в представление байта (8-бит): старший бит
использовался для контроля чётности или иных целей.
Т.о. алфавит языка составлял 128 значений, первые 32 использовались для кодирования не символов, но
команд управления устройством, через которое проходят этот поток. Т.е. они часто отсеивались на выходе.
Но не все: например, при сохранении в файл, нужно сохранять символы переводы строк.

### Кодировки фиксированной и переменной ширины: KOI8-R, CP-1251... vs UTF-8

Следующим этапом эволюции в кодировании символов было упразнение старшего бита байта как контрольного по чётности.
Т.о. появились новые значения от 128 до 255 в кодиравнии символов. Их стали занимать псевдографикой и кодировками
отдельных национальных алфавитов (за искл. азиатских языков).

В Европе алфавиты языков условно можно разделить на группы:

* [Латинница](https://ru.wikipedia.org/wiki/%D0%9A%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D1%8F:%D0%90%D0%BB%D1%84%D0%B0%D0%B2%D0%B8%D1%82%D1%8B_%D0%BD%D0%B0_%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%B5_%D0%BB%D0%B0%D1%82%D0%B8%D0%BD%D1%81%D0%BA%D0%BE%D0%B3%D0%BE)
* [Греческий](https://ru.wikipedia.org/wiki/%D0%9A%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D1%8F:%D0%90%D0%BB%D1%84%D0%B0%D0%B2%D0%B8%D1%82%D1%8B_%D0%BD%D0%B0_%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%B5_%D0%B3%D1%80%D0%B5%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B3%D0%BE_%D0%BF%D0%B8%D1%81%D1%8C%D0%BC%D0%B0)
* [Кириллица](https://ru.wikipedia.org/wiki/%D0%9A%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D1%8F:%D0%9A%D0%B8%D1%80%D0%B8%D0%BB%D0%BB%D0%B8%D1%86%D0%B0)
* Тюркские и т.д.

Большинство из них умещается в 8-битный [набор символов](https://ru.wikipedia.org/wiki/%D0%9D%D0%B0%D0%B1%D0%BE%D1%80_%D1%81%D0%B8%D0%BC%D0%B2%D0%BE%D0%BB%D0%BE%D0%B2):

* Англо-латинские:
	- ISO 646
		+ [ASCII](https://ru.wikipedia.org/wiki/ASCII) - 7 бит
	- [BCDIC](https://ru.wikipedia.org/wiki/BCDIC) - 6 бит
	- [EBCDIC](https://ru.wikipedia.org/wiki/EBCDIC) - 8 бит
* Кодировки Windows:
	- [Windows-1250](https://ru.wikipedia.org/wiki/Windows-1250) - языки на основе латиницы, Центральная Европа
	- [Windows-1251](https://ru.wikipedia.org/wiki/Windows-1251) - русская кодировка до сих пор популярна. Придумана в организациях "Диалог" и "Параграф" при участи офиса Майкрософт...
	- Windows-1252 - Западная Европа
	- [Windows-1253](https://ru.wikipedia.org/wiki/Windows-1253) - греческий
	- [Windows-1254](https://ru.wikipedia.org/wiki/Windows-1254) - турецкий, в т.ч. азербайджанский
* [КОИ-8](https://ru.wikipedia.org/wiki/%D0%9A%D0%9E%D0%98-8) - русская unix'овая сетевая кодировка. Варинты есть Укр.
* [ISO 8859](https://ru.wikipedia.org/wiki/ISO_8859) - международные кодировки
	- [ISO 8859-1](https://ru.wikipedia.org/wiki/ISO_8859-1) - расширенная латиница. Как Windows−1252.
	- [ISO 8859-2](https://ru.wikipedia.org/wiki/ISO_8859-2) - восточноевропейские языки. Как Windows-1250.
	- [ISO 8859-3](https://ru.wikipedia.org/wiki/ISO_8859-3) - южноевропейские языки, в том числе турецкий
	- [ISO 8859-4](https://ru.wikipedia.org/wiki/ISO_8859-4) - североевропейские языки: прибалтийский, финский
	- [ISO 8859-5](https://ru.wikipedia.org/wiki/ISO_8859-5) - **кириллические языки**:
		+ русский
		+ украинский
		+ белорусский
		+ сербский
		+ македонский
	- [ISO 8859-6](https://ru.wikipedia.org/wiki/ISO_8859-6) - арабский язык
	- [ISO 8859-7](https://ru.wikipedia.org/wiki/ISO_8859-7) - греческий
	- [ISO 8859-8](https://ru.wikipedia.org/wiki/ISO_8859-8) - иврит
	- [ISO 8859-9](https://ru.wikipedia.org/wiki/ISO_8859-9) - турецкий. Аналог: Windows-1254.
	- [ISO 8859-10](https://ru.wikipedia.org/wiki/ISO_8859-10) - снова Северная Европа
	- [ISO 8859-11](https://ru.wikipedia.org/wiki/ISO_8859-11) - тайский язык: Таиланд и Лаос
	- [ISO 8859-13](https://ru.wikipedia.org/wiki/ISO_8859-13) - прибалтийские языки
	- [ISO 8859-14](https://ru.wikipedia.org/wiki/ISO_8859-14) - кельтские языки
	- [ISO 8859-15](https://ru.wikipedia.org/wiki/ISO_8859-15) - снова западноевропейские языки
	- [ISO 8859-16](https://ru.wikipedia.org/wiki/ISO_8859-15) - снова восточноевропейская латиница
* [MIK](https://ru.wikipedia.org/wiki/%D0%91%D0%BE%D0%BB%D0%B3%D0%B0%D1%80%D1%81%D0%BA%D0%B0%D1%8F_%D0%BA%D0%BE%D0%B4%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0) - болгарская кириллица

Обилие этих и других непредставлянных однобайтных языков для каждого набора символов, а также то, что ими
можно составлять одну единицу (файл, сообщение) и невозможность чередовать языки привели к созданию
многобайтной кодировки всех языков мира и др. символов - [Юникод](https://ru.wikipedia.org/wiki/Юникод).

## SNMP протокол - информация и управление:

Есть несколько реализаций SNMP-протокола, самый популярный- это пакет программ Net-SNMP

### Сервис snmpd

### Программа опроса snmpwalk

## Системы мониторинга

### zabbix

### cacti

### icinga2

### mrtg

## Система сигнализации №7 в телефонии и мобильных сетях GSM-5G

### Традиционная телефония

Это сети с коммутацией каналов.
Первые коммутаторы были ручныы: "алло, девушка, соедините меня со Смольным...".
Позже стали появляться электромеханические автоматические телефонные станции с соотв. коммутаторами:

* машинные
* декадно-шаговые
* ...

Позже появились квази-электронные АТС, но все они также коммутировали речь представленную
аналоговым сигналом.

Полностью электронные АТС делались команиями Siemens, Lucent.
В таких сетях речь уже кодировалась дискретным сигналом.

Появилась выделенная сеть коммутации, которая управляла процессом установления соединения.

### Сети с цифровой интеграцией услуг: ISDN

### Интеллектуальные сети

### Сети подвижной связи : AMPS/DAMPS/GSM/UMTS/CDMA...

## Источники информации

*  [Serial Programming HOWTO](http://linux.yaroslavl.ru/docs/howto/Serial-Programming/Serial-Programming-HOWTO_ru.html) ; автор: Gary Frerking <gary@frerking.org>, Peter Baumann ; Русский перевод А.Гавва (2002 г.Львов).


