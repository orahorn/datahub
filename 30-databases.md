# Роль баз данных в ИУС

Базы данных (БД) по-сути определяют способы хранения и представляения
информации в особом структурированном виде, помогающем человеку
определять порядки её накопления, хранения и выборки.
БД задают парадигмы, по работе с массивами сведений, фактов и других
прикладных и системных показателей из реального мира и вычислительной платформы
(внутернние данные).

Пример внутренних данных - информация по установленному ПО.
Например, RPM-пакеты инексируются Berkeley DB (см. далее)

Важной особенностью БД являются системы управления (СУБД): они определяют
не только файловые форматы хранения, но и способы запросов, обработки и
выдачи данных. СУБД позволяет данным быть доступными несмотря на отключения
электроэнергии, а также при различного рода сбоев.
СУБД содержит программное обеспечение, которое ускоряет доступ к данным
за счёт их кеширования в оперативной памяти (ОЗУ, RAM).
Есть базы данных, хранящиеся в памяти: т.н.
[In memory database](https://ru.wikipedia.org/wiki/%D0%A0%D0%B5%D0%B7%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%BD%D0%B0%D1%8F_%D0%B1%D0%B0%D0%B7%D0%B0_%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)
или отдельные кеширующие СУБД.

## Локальные СУБД

Локальные СУБД - это такие системы, которые рассчитаны на работу в пределах одного блока оперативной памяти (компьютера).
Сетевые возможности не реализованы или идут как внешние дополнения.
Такие СУБД ещё называют [встраиваемые СУБД](https://ru.wikipedia.org/wiki/%D0%92%D1%81%D1%82%D1%80%D0%B0%D0%B8%D0%B2%D0%B0%D0%B5%D0%BC%D0%B0%D1%8F_%D0%A1%D0%A3%D0%91%D0%94) .

Такие системы крайне важны также для автономных систем, где нет связи с сетями,
например для [геоинформационных систем](https://ru.wikipedia.org/wiki/%D0%93%D0%B5%D0%BE%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F_%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0).


### BerkleyDB (Oracle DBM, GDBM, LMDB, QDBM,...)

Библиотека [Berkeley DB](https://ru.wikipedia.org/wiki/Berkeley_DB) предназначена
для создания приложений, управляющих [базами данных типа ключ-значение](https://ru.wikipedia.org/wiki/%D0%91%D0%B0%D0%B7%D0%B0_%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85_%C2%AB%D0%BA%D0%BB%D1%8E%D1%87-%D0%B7%D0%BD%D0%B0%D1%87%D0%B5%D0%BD%D0%B8%D0%B5%C2%BB) .
Есть привязка к большинству распространённых языков программирования (ЯП).
БД позволяет хранить по одному ключу множество значений, одновременный доступ без блокировок,
последние версии позволяют вести журнал [транзакций](https://ru.wikipedia.org/wiki/%D0%A2%D1%80%D0%B0%D0%BD%D0%B7%D0%B0%D0%BA%D1%86%D0%B8%D1%8F_(%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%82%D0%B8%D0%BA%D0%B0)).
Придумали BDB в университете Беркли, позже компания SleepyCat Software занималась коммерческой разработкой и поддержкой СУБД,
а позже, продала все права компании [Oracle](https://ru.wikipedia.org/wiki/Oracle).
В настоящее время код BDB доступен свободно по AGPL лицензии.
Среди поддерживаемых API по Oracle Berlkley DB ЯП, помимо C/C++ (Visual C++ .NET 2005+, интеграция с STL): Java, C# , Perl, TCL, PHP.
Операционные системы: Unix; 32/64-битный Windows, технологии ODBC/JDBC/ADO, Visual Studio 2005+; Apple iOS.

Установка пакетов в Debian/Ubuntu:

* db-util - комплект утилит для манипуляциями файлами данных BDB
* db5.3-doc - документация
* libdb-dev - заголовочные файлы для программирования на классическом Си
* libdb++-dev - интерфейсы классов C++
* python3-bsddb3

Для того, чтобы манипулировать с Oracle Berkeley DB файлами с помощью SQL комманд,
нужно установить пакеты:

* db5.3-sql-util -
* db5.3-sql - утилиты командной строки
* libdb5.3-sql - разделеямая библиотека
* libdb5.3-sql-dev - заголовочные файлы

Утилита `db5.3_sql` работает аналогично SQLite (см. ниже).
 

Создадим текстовые файл формата строка ключ, следующая строка - значение. Потом повторяется: строка ключ, строка значение.
Т.е. нечётные с первой строчки через одну - ключи,  а чётные со второй (также через одну) - значения.
Пример содержания логинов и паролей в текстовом файле `logins.txt`:

```txt
user
123456
root
123456
```

Создаём файл БД :

	db_load -T -t hash -f logins.txt logins.db

Здесь:

* `-T` - указывает, формат входных текстовых данных, через строку
* `-t hash` - тип поиска строки с заданным ключём по хешу
* `-f logins.txt` - брать исходные данные из текстового файла `logins.txt`, а не из стандартного ввода

Смотрим всю структуру БД:

```
$ db_dump -d a logins.db
In-memory DB structure:
hash: 0x110000 (open called, read-only)
meta_pgno: 0
h_ffactor: 0
h_nelem: 2
h_hash: 0x7f4178beece0
=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
page 0: hash metadata: LSN [0][1]: level 0
	magic: 0x61561
	version: 9
	pagesize: 4096
	type: 8
	metaflags 0
	keys: 0	records: 0
	free list: 0
	last_pgno: 2
	flags: 0
	uid: cd 20 5a 0 13 8 0 0 8c 17 a2 7 19 30 0 0 0 0 0 0
	max_bucket: 1
	high_mask: 0x1
	low_mask:  0
	ffactor: 0
	nelem: 2
	h_charkey: 0x5e688dd1
	spare points:
	1 (1) 1 (2) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0)
	0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0)
	0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0)
	0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0) 0 (0)

page 1: hash: LSN [0][1]: level 0
	prev:    0 next:    0 entries:    2 offset: 4084
	[000] 4091 len:   4 data: root
	[001] 4084 len:   6 data: 123456
page 2: hash: LSN [0][1]: level 0
	prev:    0 next:    0 entries:    2 offset: 4084
	[000] 4091 len:   4 data: user
	[001] 4084 len:   6 data: 123456
```

Как видим: файл разбивается на страницы с записями по определённым смещениям.

Более упрощённый вариант:

```
$ db_dump -p logins.db
VERSION=3
format=print
type=hash
h_nelem=2
db_pagesize=4096
HEADER=END
 root
 123456
 user
 123456
DATA=END
```

Создадим файл `db2txt.py` на языке Python:


```python
#!/usr/bin/python3

import bsddb3
for k, v in bsddb3.hashopen("logins.db").iteritems():
        print(k,v)
```

Читаем данные:

```
$ ./db2txt.py
b'root' b'123456'
b'user' b'123456'

```

В языке Perl уже встроены средства для работы с Berkley DBM.
Пример файла `db2txt.pl`:

```perl
#!/usr/bin/perl -w

use DB_File;
use strict;

my (%h, $k, $v);

tie %h, "DB_File", "logins.db", O_RDONLY, 0666, $DB_HASH
    or die "Cannot open file : $!\n";

while (($k, $v) = each %h)
  { print "$k -> $v\n" }

untie %h ;

```

DBM-файл соединяется напрямую с хешем `%h`.

запускаем:

```
$ ./db2txt.pl
root -> 123456
user -> 123456
```

Есть ещё версия от [GNU](https://www.gnu.org/) проекта: [GDBM](https://www.gnu.org.ua/software/gdbm/).
Установка в Debian/Ubuntu:

	sudo apt install libgdbm-dev python3-gdbm

Ставится пакет с заголовочными файлами для ЯП Си и пакет для Python-3.

Напишем программу на Си для создания базы данных в файл `create1.c`:

```c
#include <gdbm.h>
#include <sys/stat.h>
#include <stdio.h>
#include <string.h>

int main() {
        GDBM_FILE dbf;
        char * fn = "students1.db";

        dbf = gdbm_open (fn, 0, GDBM_NEWDB, S_IREAD|S_IWRITE|S_IRGRP|S_IWGRP|S_IROTH, NULL);
        if (dbf==NULL) {
                perror ("gdbm_open");
                return 1;
        } else {
                printf ("File \"%s\" opened successfully at %p\n", fn, dbf);
        }

        char * names[] = {"Andrey", "Catherine", "Vlad", "Ivan" };
        char * themes[] = {"Logs", "JPEG dirs", "???", "PDD" };
        int i; datum key, content;
        for(i=0; i<sizeof(names)/sizeof(char *); i++){
                printf("i=%d\n", i);
                key.dptr=names[i];
                key.dsize=strlen(names[i])+1;
                content.dptr=themes[i];
                content.dsize=strlen(themes[i])+1;
                gdbm_store (dbf, key, content, GDBM_REPLACE);
        }


        gdbm_close (dbf);

        return 0;
}

```

Скопмилируем:

```
$ LDLIBS=-lgdbm make create1
cc     create1.c  -lgdbm -o create1
```

Приложение создаст файл с данными:

```
$ ./create1
File "students1.db" opened successfully at 0x55797027d260
i=0
i=1
i=2
i=3

$ file students1.db
students1.db: GNU dbm 1.x or ndbm database, little endian, 64-bit

```

Напишем программу для чтения этого файла ("students1.db" , в нашем примере)
в файл `reader1.c`:

```c
#include <gdbm.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main() {
        GDBM_FILE dbf;
        char *name="students1.db", *rep;
        dbf = gdbm_open (name, 0, GDBM_READER, 640, NULL);
        if (dbf == NULL) {
                const char * ret = gdbm_strerror(gdbm_errno);
                rep = (char *)malloc ((strlen(name)+strlen(ret))*2);
                strcpy (rep, ret);
                strcat (rep, " -> \"");
                strcat (rep, name);
                strcat (rep, "\"");
                perror(rep);
                free(rep);
                return EXIT_FAILURE;
        }
        datum key, nextkey, content;
        key = gdbm_firstkey (dbf);
        char *format="%-20s\t%-20s\n", *sep="---";
        printf(format, "Student", "Theme");
        printf(format, sep, sep);
        while ( key.dptr ) {
                nextkey = gdbm_nextkey ( dbf, key );
                content = gdbm_fetch(dbf, key);
                printf(format, key.dptr, content.dptr);
                key = nextkey;
                }
        gdbm_close (dbf);
        return EXIT_SUCCESS;
}

```

Скомпилируем и запустим:

```
$ LDLIBS=-lgdbm make reader1
cc     reader1.c  -lgdbm -o reader1

$ ./reader1
File open error -> "students1.db": No such file or directory

$ mv ../gdbm2/students1.db .

$ ./reader1 
Student             	Theme               
---                 	---                 
Catherine           	JPEG dirs           
Vlad                	???                 
Ivan                	PDD                 
Andrey              	Logs                

```


Среди приложений использующих хеши данных могут быть например счётчики с показаниями статистики,
файлы индексов полей таблиц реляционных баз данных, или каталоги в иерархии данных, такие,
например, как [LDAP](https://ru.wikipedia.org/wiki/LDAP).

Вот для ведения подобных
[служб каталогов](https://ru.wikipedia.org/wiki/%D0%A1%D0%BB%D1%83%D0%B6%D0%B1%D0%B0_%D0%BA%D0%B0%D1%82%D0%B0%D0%BB%D0%BE%D0%B3%D0%BE%D0%B2) , где данные лежат в форамте [LDIF](https://ru.wikipedia.org/wiki/LDIF) создана СУБД 
[LMDB](https://en.wikipedia.org/wiki/Lightning_Memory-Mapped_Database) . Несмотря на то, что писатель в файл
может быть только один - нет взаимных блокировок между ним и читателями.


БД [Tkrzw](https://en.wikipedia.org/wiki/Tkrzw) Tokyo Cabinet and Kyoto Cabinet - использует механизм поиска ключа по
[двоичному дереву](https://ru.wikipedia.org/wiki/B%E2%81%BA-%D0%B4%D0%B5%D1%80%D0%B5%D0%B2%D0%BE)
и хеширование ключей. База продолжает развитие СУБД "Tokyo Cabinet", который больше не будет разрабатываться, но
всё ещё присутствует в последних дистрибутивах ОС.

Компания Google производит свой вариант [LevelDB](https://en.wikipedia.org/wiki/LevelDB) СУБД типа "ключ-значение".

Также довольно популярна СУБД [QDBM](http://fallabs.com/qdbm/) в популярных дистрибутивах:

```
$ apt-cache pkgnames |grep -i qdbm
libqdbm-perl
qdbm-cgi
qdbm-doc
libqdbm3++c2
libxqdbm-dev
libxqdbm3c2
libqdbm-java
libqdbm++-dev
libqdbm-dev
ruby-qdbm
libqdbm14
qdbm-util
```

Позиционируется, как более быстрая, менее занимающая место в хранилище и СУБД с более простым интерфейсом API.


### SQLite

СУБД [SQLite](https://ru.wikipedia.org/wiki/SQLite)
не работает по принципу клиент-сервер.
Приложение с SQLite функционалом
использует только библиотеку времени исполнения `libsqlite*.so*` точно,
также, как и остальные разделяемые библиотеки.
Вся база данных лежит во одном файле.
Нулевая конфигурация.
Используется стандарт [SQL-92](https://ru.wikipedia.org/wiki/SQL-92),
активна версия SQLite-3.
SQLite — встраиваемая легковесная SQL система управления базами данных - изначально разрабатывалась как расширение
языка программирования [Tcl](https://ru.wikipedia.org/wiki/Tcl) .
В ОС [Android](https://ru.wikipedia.org/wiki/Android) для работы над приложениями доступна библиотека SQLite.
Вообще SQLite - это по сути [встраиваемая СУБД](https://ru.wikipedia.org/wiki/%D0%92%D1%81%D1%82%D1%80%D0%B0%D0%B8%D0%B2%D0%B0%D0%B5%D0%BC%D0%B0%D1%8F_%D0%A1%D0%A3%D0%91%D0%94) .

Практиковаться можно на сайте [REPL](https://repl.it/languages/sqlite)
в запущеном интерпретаторе команд SQL.

Синтаксис помимо стандарта мало чем добавляется,
т.к. нет дополнительного сетевого функционала.

В SQLite не так много разных типов данных.
Вот базовые:

* `NULL` - ничего нет в поле
* Числовые:
	- `INTEGER` - целые
	- `REAL` - вещественные
* `TEXT` - текст
* `BLOB` - всё что угодно (двоичные данные)

С маленькой буквы принято писать команды оболочки (начинаются с точки),
а с большой - сами выражения.

В SQLite нет команд:

* `CREATE DATABASE`
* `DROP DATABASE`


Есть специальные команды:

* `ATTACH DATABASE` - присоединить файл данных для обработки
* `DETACH DATABASE` - отсоединить его
* `VACUUM` -  сборка базы данных. Дефрагментация файла с БД.


Установка библиотеки и утилит:

	sudo apt install sqlite3


Документация:

	sudo apt install sqlite3-doc

Пакет `sqlite3` содержит программы:

* sqldiff - сравнение файлов с данными. Т.е. вывод таких команд, которые из одной СУБД сделают другую.
* sqlite3 - командный интерпретатор. 

Создание базы данных (БД) выполняется командой:

	sqlite3 файл.db

`файл.db` - будет содержать таблицы базы данных.

Создание таблицы (пример, для записей персональных данных):

	create table persons(persid integer, name varchar(20), title varchar(10));

Проверяем, что таблица создалась (ввод начинается с `sqlite> `):

```
sqlite> .tables
persons

sqlite> .schema
CREATE TABLE persons(persid integer, name varchar(20), title varchar(10));
```


Вводим данные:

	sqlite> insert into persons values(1, 'Alexandr Andrienko', 'Boss');

Смотрим содержимое таблицы. Пример всех полей для таблицы persons:

```SQL
select * from persons;
1|Александр|Андриенко|+7-905-486-2232
2|Иван|Маслов|+7-903-476-3441
```

Можно запускать заранее подготовленные в файлах SQL-скрипты .
Например, запишем в файл `employee.sql` такой текст скрипта:

```SQL
/*

Syntax:

sqlite3 employee.db < employee.sql

*/

PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE "employee"("EmployeeID" INTEGER PRIMARY KEY NOT NULL, "Name" VARCHAR, "Age" INTEGER);
INSERT INTO "employee" VALUES(1,'John',34);
INSERT INTO "employee" VALUES(2,'Mark',44);
INSERT INTO "employee" VALUES(3,'Mathew',50);
COMMIT;
```

Запускаем из командной строки:

	sqlite3 employee.db < employee.sql


Смотрим результат:

	sqlite3 employee.db

```SQL
sqlite> SELECT * FROM employee;
1|John|34
2|Mark|44
3|Mathew|50
```

Графическое приложение для просмотра БД содержится в пакете ОС `sqlitebrowser`.
У него [Qt интерфейс](https://ru.wikipedia.org/wiki/Qt)
с компонентом [Scientilla](https://ru.wikipedia.org/wiki/Scintilla).
Алгоритм работы прост.
Вызываем кнопку "Новая база данных": просят указать имя и место файла.
Набираем имя таблицы: параллельно создаётся конструкция "CREATE TABLE".
В той же форме добавляем поля кнопкой "Добавить поле".
Во вкладке "Данные" можно добавлять записи.
Сохраняем базу кнопкой и выходим.
Полученный файл содержит всю базу данных.

Пакет `libsqlite3-dev` содержит заголовочный файл для C/C++:

	/usr/include/sqlite3.h

и также статические библиотеки.

Пишем код в файл `persons2.c`:

```c
#include <sqlite3.h>
#include <stdio.h>

static int callback (void *NotUsed, int argc, char **argv, char **szColName) {
        int i;
        for (i = 0; i < argc; i++)
        {
                printf ("%s = %s\n", szColName[i], argv[i]);
        }
        puts("\n");
        return 0;
}

int main ()
{
        sqlite3 *db;
        char *szErrMsg = NULL;

        // open database
        int rc = sqlite3_open("persons.db", &db);
        if (rc)
        {
                puts ("Can't open database\n");
                return 2;
        } else {
                puts ("Open database successfully\n");
        }

        rc = sqlite3_exec (db, "SELECT * FROM PERSONS;", callback, 0, &szErrMsg);
        if (rc != SQLITE_OK)
        {
                fprintf (stderr, "SQL Error: %s", szErrMsg);
                return 1;
        }

        if (db) {
                sqlite3_close (db);
        }

        return 0;
}

```

Компилируем:

```
$ LDLIBS=-lsqlite3 make persons2
cc     persons2.c  -lsqlite3 -o persons2
```

Запускаем:

```
./persons2
Open database successfully

id = 1
fn = Александр
sn = Андриенко
tel = +7-909-372-7954


id = 2
fn = Иван
sn = Маслов
tel = +7-901-384-5488


id = 3
fn = Бабушка
sn = Клава
tel = +7-929-594-0696


id = 4
fn = Валерий
sn = Смирнов
tel = +7-905-380-1120

...

```

В языке программирования (ЯП) Perl - доступ к реляционным СУБД (РСУБД),
на языке SQL происходит универсальным способом.

Интерфейс с базами данных унифицирован.

В Debian, Ubuntu ставим пакет libdbd-sqlite3-perl .
Он зависит от библиотеки общего вида DBI (пакет libdbi-perl) ,
которая обобщает интерфейс к программированию баз данных.
В ALTLinux это пакет perl-DBD-SQLite .

В коде используем Perl-модули DBI и DBD::SQLite

Для работы с ними достаточно подключить в начале соотв. модуль:

```perl
use DBI;
```

Далее, в зависимости от конкретного типа СУБД нужно ставить нужный модуль DBD (DataBase Driver).


Подключение к СУБД (для других баз, см далее):

* MySQL: `$dbh = DBI -> connect ("DBI:mysql: ... ");`
* Postgres: `$dbh = DBI -> connect ("DBI:Pg:... ");`
* Oracle: `$dbh = DBI -> connect ("DBI:Oracle:... ");`

Рекомендации по именованию переменных:

* `$h` - описатель (дескриптор, handler) общего назначения
* `$fh` - описатель открытого файла (file)
* `$dbh` - описатель отркытой СУБД (database)
* `$sth` - описатель оператора (statement) обращения к СУБД
* `$rc` -  логический код (code) возвращаемый (return)  операторами (true / false)
* `$rv` - числовое значение (value) возвращаемой операторами

Пример кода для SQLite запишем в файл `persons.pl`:


```perl
#!/usr/bin/perl

use DBI;
use strict;


my $dbfile="persons.db";
my $dbh = DBI->connect("dbi:SQLite:dbname=$dbfile","","")
        or die $DBI::errstr;   # die передает сообщение в поток ошибок
print "Opened database successfully\n";

print "DBD SQLite version: $DBD::SQLite::sqlite_version\n";

my $sth = $dbh->prepare("SELECT * FROM persons WHERE id < 10");
        $sth->execute();

while(my @row = $sth->fetchrow_array()) {
        print "$row[0],$row[1],$row[2],$row[3]\n";
}
```

Делаем исполнимым и запускаем скрипт:

```
$ chmod u+x persons.pl
$ ./persons.pl
Opened database successfully
DBD SQLite version: 3.22.0
1,Александр,Андриенко,+7-905-382-1269
2,Иван,Маслов,+7-902-796-5642
3,Бабушка,Клава,+7-929-594-0696
4,Валерий,Смирнов,+7-900-832-3389
5,Геннадий,Кузнецов,+7-915-425-7609
6,Дежурная,Смена,+7-495-970-3382
7,Смена,Дежурная,+7-903-483-3435
```

В стиле Perl есть библиотека [libDBI](http://libdbi.sourceforge.net/) для языка Си.


Для TCL СУБД SQLite - это ребёнок проекта.
Однако в дистрибутивах она выделена в отдельный пакет.
Устанавливаем его:

	apt install libsqlite3-tcl 

В коде своего скрипта подключаем соответствующий пакет:

```tcl
package require sqlite3
```

Открываем базу данных db по имени файла:

```tcl
sqlite3 db "файл.db"
```

Скрипт `persons.tcl` будет таким:

```tcl
#!/usr/bin/tclsh

# On Ubuntu-18.04 run:
#       apt install libsqlite3-tcl
#
# Sample of running on "persons.db" SQLite database file
# and "Persons table"
#./persons.tcl persons.db 'select * from persons;'
# 

if {$argc!=2} {
        puts stderr "Usage: %s DATABASE SQL-STATEMENT"
        exit 1
}
package require sqlite3
sqlite3 db [lindex $argv 0]
db eval [lindex $argv 1] x {
        foreach v $x(*) {
                puts "$v = $x($v)"
        }
        puts ""
}
db close
```

```
$ ./persons.tcl persons.db 'select * from persons;'
id = 1
fn = Александр
sn = Андриенко
tel = +7-902-481-5255

id = 2
fn = Иван
sn = Маслов
tel = +7-916-712-5141

id = 3
fn = Бабушка
sn = Клава
tel = +7-929-594-0696

id = 4
fn = Валерий
sn = Смирнов
tel = +7-901-381-1164

id = 5
fn = Геннадий
sn = Кузнецов
tel = +7-903-727-6851

id = 6
fn = Дежурная
sn = Смена
tel = +7-495-974-2745

id = 7
fn = Смена
sn = Дежурная
tel = +7-915-381-2953

```


### Резидентная СУБД memcached

[Memcached](https://ru.wikipedia.org/wiki/Memcached) - не совсем БД - это такой способ хранения обрабатываемых записей
из базы в оперативной более быстрой памяти. Т.е. в логике программы пишутся функции, которые выгребают данные не из дисковой
удалённой СУБД сразу, а сначала смотрят, есть ли эти записи в Memcached базе. Если есть - то запрос отрабатывает на столько
быстрее, насколько это возможно при доступе из ОЗУ, чем при открытии сетевого соединения и передачи данных по сети + время
обращения к данным на дисковом хранилище.

Для перманентного хранения этих захешированных данных
можно использовать ПО [MemcacheDB](https://github.com/LMDB/memcachedb/) , которая в фоновом режиме складывает данные в 
файл на диск.


### Ретро БД DBase (xbase), Clipper, FoxPro

На рубеже 80-х-90-х годов прошлого века были популярны однофайловые
однопользовательские СУБД для ПК.
Как правило для них использовались [DBF-файлы](https://ru.wikipedia.org/wiki/DBF).
Сначала использовали системы [DBase](https://ru.wikipedia.org/wiki/DBase)-III, IV ;
Они были медленными, т.к. использовали интерпретаторы.
Потом появился быстрый компилятор [Clipper](https://ru.wikipedia.org/wiki/Clipper) создающий EXE файлы.
Потом Microsoft 
пересоздала удобный продукт [Visual FoxPro](https://ru.wikipedia.org/wiki/Visual_FoxPro) (в наши дни не развивается).

Чтобы посмотреть какое ПО Debian/Ubuntu всё ещё поддерживает старые DBF-файлы
достаточно делать запросы к пакетному менеджеру:

	apt search xbase


Ещё хотелось бы отметить, что в эру ПК в нашей стране использовалась
система для учёта и управления предприятием БЭСТ-4+ (до 1С).
Хотя компания [БЭСТ](https://bestnet.ru/) существует и в наше время.
А для приложений на Clipper развивается проект [Harbour](https://ru.wikipedia.org/wiki/Harbour).

## клиент-серверные СУБД

Наличие языка SQL способствовало разделению ИУС на две (три) части (звена):

1. Приложение с пользовательским интерфейсом на клиенте - frontend.
2. Сервер (web) приложений (может остустсвовать) - backend
3. Сервер баз данных, работающих с языком SQL.


### MySQL


Установка сервера на Ubuntu:

	sudo apt-get install mysql-server

также будет поставлен и клиент.

Процессы в ОС:

	mysqld
	mysqld_safe

Остановка сервера командой:

	mysqladmin shutdown

На Devuan сервисы MySQL запускаются так:

	service mysql stop
	service mysql start

Аналогично и в новых SystemD системах, например Ubuntu: только через утилиту `systemctl`.


Конфигурация лежит в файлах:

	/etc/mysql/my.cnf

При первой установке создаётся пользователь root с пустым или одноимённым паролем.

Клиент с сервером может соединяться из командной строки локально или удалённо.

Соединение с удалённой базой данных:

	mysql -u <пользователь> [-p<пароль>] [-h <машина>] [-P <порт>] [<БД>]
где:

* `<пользователь>` - имя пользователя к СУБД
* `<пароль>` - пароль. Пишется слитно с опцией -p. Опускается, только если пароль не задан для пользователя. 
* `<машина>` - адрес машины (IP или по имени). По умолчанию: локальная (localhost).
* `<порт>` - значение для TCP-порта. По умолчанию: 3360
* `<БД>` - имя СУБД на машине

Например:

	mysql -u andy -pnight1kap -h saturn customers

* andy - имя пользователя
* night1kap - пароль
* saturn - имя машины
* customers - база данных

Команды в MySQL клиенте (не зависят от регистра):

* Выйти из командного интерпретатора: `QUIT - \q`, нажать `[Ctrl-D], exit`
* Показать, какие есть базы данных: `SHOW DATABASES;`
* Отменить ввод набранного оператора или строки (cancel): `\c` 
* Стандартные команды SQL. Они как правило содержат в конце и разделяются точкой с запятой (;) и могут содержаться несколько в одной строке ввода. Например:

```sql
select now() as сейчас; select user() as пользователь;
+---------------------+
| сейчас              |
+---------------------+
| 2017-02-13 16:40:49 |
+---------------------+
1 row in set (0.00 sec)

+--------------------------+
| пользователь             |
+--------------------------+
| root@localhost           |
+--------------------------+
1 row in set (0.00 sec)
```


Функции, операции и переменные MySQL

Функции:

* VERSION() - выдаёт текущую версию сервера, с которым соединились\
* NOW() - выдаёт текущую дату и время на сервере.
* USER() - выдаёт имя текущего пользователя.
* Переменная CURRENT_DATE - даёт ту дату, которая стоит на сервере.

```sql
select version(), current_date as cd;
+-------------------------+------------+
| version()               | cd         |
+-------------------------+------------+
| 5.5.54-0ubuntu0.14.04.1 | 2017-02-13 |
+-------------------------+------------+
1 row in set (0.01 sec)
```

Математические операторы:

* `/` - деление.
* `*` - умножение.
* `+` - сложение.
* `-` - вычитание.
* `(`,`)` - скобки повышающие приоритет вычислений.
* Значения
* Комментарии:
	- Парые одно и более строчные (Как в языке C).
	- Открывающий `/*`
	- Закрывающий `*/`
	- От места и до конца строки: `--`, `#`.

Интеграция с Perl в Debian/Ubuntu поддерживается пакетом:

	apt show libdbd-mysql-perl


Интеграция с языком PHP
Ставим пакет:

	apt-get install phpmyadmin

Потом можно будет направлять браузер по адресу:

	http://<адрес_хоста>/phpmyadmin



### PostgreSQL



Установка на Debian/Ubuntu-основанных дистрибутивах:

```
apt update
apt dist-upgrade
apt install postgresql
apt install postgresql-<версия.обновление>
apt install postgresql-doc
```

Где `<версия.обновление>` - это, номер основной версии и минорного обновления.

Сервис запускается/останавливается командой:

	systemctl start postgresql.service

Потом в ОС становимся пользователем postgresql:

	su - postgres

Подсоединяемся в консоли ОС с PostgreSQL сервером командой:

	psql


Унаследованное ПО:
 
* Установка на устаревшие дистрибутивы старых версий: https://www.postgresql.org/message-id/YBMtd6nRuXyU2zS4%40msg.df7cb.de
* Репозитории, которые не актуальны кладутся в архив: https://apt-archive.postgresql.org/
* Там же есть инструкция : в файл  `/etc/apt/sources.list` добавить пару строк: `deb https://apt-archive.postgresql.org/pub/repos/apt DIST-pgdg-archive main` и `deb-src https://apt-archive.postgresql.org/pub/repos/apt DIST-pgdg-archive main`, где `DIST` - кодовое имя дистрибутива, надо смотреть в https://apt-archive.postgresql.org/pub/repos/apt/dists/index.html . Например:
	- trusty
	- jessie
	- ...



В утилите `psql` можно посмотреть таблицы командой `\l[ist]` . Например:

```sql
postgres=# \l
                                  Список баз данных
    Имя    | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |     Права доступа     
-----------+----------+-----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | 
 template0 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          +
           |          |           |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          +
           |          |           |             |             | postgres=CTc/postgres
(3 строки)
```

Можно создать свою базу данных командой CREATE DATABASE . Например:

```sql
CREATE DATABASE tutorial;
CREATE DATABASE
postgres=# \list
                                  Список баз данных
    Имя    | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |     Права доступа     
-----------+----------+-----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | 
 template0 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          +
           |          |           |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          +
           |          |           |             |             | postgres=CTc/postgres
 tutorial  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | 
(4 строки)
```

Потом можно соединиться с этой базой данных командой `\c` . Также работает. Например:

```sql
\c tutorial 
```

Вы подключены к базе данных "tutorial" как пользователь "postgres".

Для выхода из psql программы с интерфейсом командной строки используем команду `\q` .

По умолчанию postgresql даёт соединяться только локально.
Но можно настроить и удалённый доступ.
В каталоге `/etc/postgresql/<версия>/main` находятся конфигурационные файлы
Основной файл `postgresql.conf` содержит большинство настроек.
Чтобы сервер слушал нужный интерфейс (IP) - открыл сокет - нужно явно этот локальный адрес на локальном интерфейсе указать.
Брать можно, например из `ifconfig`.
Тогда например, для интерфейса на котором сконфигурирован адрес 40.3.24.63, строка будет такой (оставляем адрес локальной петли):

```
listen_addresses = '40.3.24.63,localhost'               # what IP address(es) to listen on;
```

На удалённой от сервера машине можно поставить только пакет с клиентом postgresql-client-common .

Общий синтаксис клиентской консольной утилиты psql:

	psql[ -h <хост>] <база> <пользователь>

Примеры:

```
psql -h 40.3.24.63 tutorial postgres
psql -h localhost basicdemos postgres
```

Чтобы сервер пускал, нужно в файле `/etc/postgresql/<версия>/main/pg_hba.conf` добавить соотв. запись.
Например, чтобы пускать ко всем базам данных всех пользователей со всех удалённых машин  по паролю - нужно внести запись:

```
host     all             all        0.0.0.0/0           md5
```

Пример тот же для удалённой машины с адресом 40.3.24.62:

```
host     all             all        40.3.24.62/32           md5
```

Для того, чтобы позволить удалённо соединяться пользователю postgres , нужно установить на него пароль локально (в psql утилите).
Делается это командой:

```sql
ALTER USER postgres WITH PASSWORD 'пароль';
```

После этого в базе данных можно создавать таблицы.
Например:

```sql
 create table test(id bigint);
CREATE TABLE
```

В таблицах поля могут иметь разные значения и правила их эксплуатации. Поля могут быть первичными ключами. Пример:

```sql
tutorial=# create table professors(id int primary key, first_name text not null, last_name text not null, class text null);
CREATE TABLE
```

Заполняет таблицу команда INSERT. Пример для таблицы professors:

```sql
tutorial=# insert INTO professors VALUES ( 1, 'Victor', 'Davur', 'Databases' );
INSERT 0 1
```

Просмотрим содержимое таблицы:

```sql
tutorial=# select * from professors ;
 id | first_name | last_name |   class   
----+------------+-----------+-----------
  1 | Victor     | Davur     | Databases
(1 row)
```


Обновлять (редактировать) поля (даже ключи) можно командой UPDATE . Например, обновим ключ с 2 на 4:

```sql
update professors SET id = 4 WHERE id = 2;
UPDATE 1
```

Можно создавать наследуемые таблицы. Например, создадим таблицу  coordinators с новым текстовым полем cource_area на основе таблицы professors:

```sql
 create table coordinators (cource_area text NOT NULL) inherits (professors);
CREATE TABLE
```

Тогда можно будет добавлять в таблицу в те же поля, что и у наследуемой таблицы + в конце в новые. Для нашего случая:

```sql
insert INTO coordinators values (5, 'Pablo', 'Lopez', 'Database Optimization', 'Database Area');
INSERT 0 1
```

И вот что получилось:

```sql
tutorial=# select * from coordinators ;
 id | first_name | last_name |         class         |  cource_area  
----+------------+-----------+-----------------------+---------------
  5 | Pablo      | Lopez     | Database Optimization | Database Area
(1 row)
```


При этом значения попадают и в родительскую таблицу.
Т.к. наследуемая таблица с ней связана.
Вот что с нашим примером:

```sql
tutorial=# select * from professors;
 id | first_name | last_name |         class         
----+------------+-----------+-----------------------
  1 | Victor     | Davur     | Databases
  3 | Landy      | Lopez     | Data Structures
  4 | Daniel     | Alvarez   | Compilers
  2 | Carlos     | Agular    | Databases 2
  5 | Pablo      | Lopez     | Database Optimization
(5 rows)
```


Изменение структуры таблицы возможно уже даже после её заполнение и обновлений.
Для этого используют команду ALTER TABLE.
Например, добавим колонку Salary в таблицу Professors:

```sql
tutorial=# alter table professors add  salary int;
ALTER TABLE
```

Если нужны данные только из родительской таблицы (без тех, что добавлены в унаследовавшую дочернюю), то в выражении SELECT используется после ключевого слова FROM выражение ONLY. Например, для нашей таблицы Professors:

```sql
tutorial=# select * from only professors;
 id | first_name | last_name |      class      | salary 
----+------------+-----------+-----------------+--------
  1 | Victor     | Davur     | Databases       |       
  3 | Landy      | Lopez     | Data Structures |       
  4 | Daniel     | Alvarez   | Compilers       |       
  2 | Carlos     | Agular    | Databases 2     |     40
(4 rows)
```

Как видно, строка с идентификатором 5, добавленная в coordinators - отсутствует.

Если нужно вспомнить структуру, уже созданной таблицы, то используется команда:

```sql
\d таблица
```

Табиляция работает, как автодополнение - подсказывает нужные таблицы.

Пример просмотра таблицы professors:

```sql
\d professors
 id         | integer | NOT NULL
 first_name | text    | NOT NULL
 last_name  | text    | NOT NULL
 class      | text    | 
 salary     | integer |
```


Также для администрирования можно поставить пакет pgadmin3 с одноимённой командой.
Это приложение на библиотеке QtGUI.
Запускается от обычного пользователя и сохраняет конфигурацию.

Более продвинутая 4-я версия находится в пакете pgadmin4 (ставится из репозиториев PostgreSQL.org).
Запускается командой:

	pgAdmin4

и в браузере открывается ссылка на сервер.
В нём можно манипулировать данными.
Только нужно поставить мастер-пароль (допустим 123456).


Третьи производители:

* https://en.wikipedia.org/wiki/DBeaver
* https://dbfiddle.uk/
* Azure Data Studio


Для работы в Perl нужно установить драйвер DBI. Например, в Debian/Ubuntu:

	sudo apt install libdbd-pg-perl


Задание:

На языке PL/pgSQL создать БД shop (интернет магазин) со следующей структурой:

* Таблица orders (заказы) с полями: `or_id` (идентификатор заказа), customer (имя покупателя), date (дата заказа).
* Таблица Warenhouse (товары на складе): `p_id` (идентификатор товара), название товара (product), rest (минимальный остаток на складе), quant (количество единиц товара на складе).
* Таблица reserved (резерв): `or_id` (идентификатор заказа), `p_id` (идентификатор товара), quant (зарезервированное количество), date (дата резервирования).
* Таблица `Pds_warn` (предупреждения): `date_time` (дата и время записи), mesg (содержание записи).

2. Заполнить таблицу Warenhouse данными (не менее 3 типов товара).
3. На языке PL/pgSQL создать триггер и триггерную функцию: `orders_after_insert()`, которая будет выполняться после вставки новой записи в таблицу orders и проверять, есть ли необходимое количество товара на складе.    

Если есть, то зарезервировать товар.

Если необходимого количества на складе нет, то проверить, не находится ли в резерве нужный товар свыше 5 дней. Если требуемый товар имеется и количество товара по просроченным заказам достаточно для исполнения текущего заказа, то просроченные заказы нужно аннулировать, а товар зарезервировать для нового заказа.

Проверить правильность работы триггера на примере записи нового заказа в таблицу Orders.



## распределённые СУБД больших данных (NoSQL)

Концепция NoSQL изначает не отрицание, а наоборот - расширение - **не только SQL**.
Т.е. предполагается:

1. Хранение данных не только в жёстких форматах таблиц, но и в другом разрозненном виде. Например, иерархически.
2. Возможность логической связности тех же таблиц с иными структурированными данными в другом формате.
3. Доступ к данным даже тех же таблиц более разнообразно: например, не только построчно (по записи), но и поколоночно...

### MongoDB



### Redis


## Источники информации

* [Программирование на Perl DBI](https://www.books.ru/books/programmirovanie-na-perl-dbi-fail-pdf-595893) ; Авторы: Аллигатор Декарт, Тим Банс ; Издательство: Символ-Плюс ; Дата выхода: октябрь 2000 ; ISBN: 5-93286-013-8
* [Getting Started with Berkeley DB Transaction Processing](https://docs.oracle.com/cd/E17076_02/html/gsg_txn/C/index.html)
* [Berkeley DB Programmer's Reference Guide](https://docs.oracle.com/cd/E17076_02/html/programmer_reference/index.html)
* [Berkeley DB Installation and Build Guide](https://docs.oracle.com/cd/E17076_02/html/installation/index.html) ;  12/19/2011
* [SQLite в ОС ALT Linux wiki](https://www.altlinux.org/SQLite)
* [MemcacheDB](https://en.wikipedia.org/wiki/MemcacheDB)
* [Distributed Caching with Memcached](https://www.linuxjournal.com/article/7451)


