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

### BerkleyDB (Oracle DBM, GDBM, LMDB, QDBM,...)


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


### Ретро БД DBase (xbase), Clipper, FoxPro

На рубеже 80-х-90-х годов прошлого века были популярны однофайловые
однопользовательские СУБД для ПК. Как правило для них использовались DBF-файлы.
Сначала использовали системы DBase-III, IV ;
Они были медленными, т.к. использовали интерпретаторы.
Потом появился быстрый компилятор Clipper создающий EXE файлы.
Потом Microsoft 
создала удобный продукт Visual FoxPro (в наши дни не развивается).

Чтобы посмотреть какое ПО Debian/Ubuntu всё ещё поддерживает старые DBF-файлы
достаточно делать запросы к пакетному менеджеру:

	apt search xbase


## клиент-серверные СУБД

Наличие языка SQL способствовало разделению ИУС на две (три) части (звена):

1. Приложение с пользовательским интерфейсом на клиенте - frontend.
2. Сервер (web) приложений (может остустсвовать) - backend
3. Сервер баз данных, работающих с языком SQL.


### MySQL

### PostgreSQL


## распределённые СУБД больших данных (NoSQL)

### MongoDB

### Redis


## Источники информации



