# Языки программирования и двоичный код

## Виды двоичного кода

### Станки ЧПУ и 3Д принтеры

Для управления станками с числовым программным управлением и 3D-устройствами печати используют
т.н. двоичный G-код, в котором прописаны команды движения механических частей, включение различных
двигателей, остановка и др.
Код для станков ЧПУ записывается в файл с расширением (суффиксом) cnc.

### Двоичный код

Такой код, который записывается набором битов.
Такое представление используется для хранения, передачи
и обработки машинным способом в комбинационных логических схемах (мгновенное преобразование без предыстории),
в секвенциональных схемах (есть предыстория состояния в триггерах).
Биты двоичного кода группируются минимум в байты.
Байтами например кодируют символы английского алфавита и ряд команд управления устройствами передачи информации.
Такой код сохранился до сих пор как краеугольный камень и используется как база для других кодов,
называется он [ASCII](https://ru.wikipedia.org/wiki/ASCII).
Над кодом ASCII для совместимости используют код символов мирового языка переменной длинны
[Юникод](https://ru.wikipedia.org/wiki/%D0%AE%D0%BD%D0%B8%D0%BA%D0%BE%D0%B4).
Двоичный код составляет не только представление текста, но и других информационных содержимых: изображения,
звука, и прочих мультимедия потоков...
Двоичный код исполняется процессорами, и сохраняется в различных исполнимых файлах: 

* Portable Executable (PE)
* Executable and Linkable Format (ELF)


### GNU Binutils

Пакет содержит важные утилиты для работы с исполнимыми файлами, библиотеками,
и т.д.

* **ldd** - просмотр ссылок исполнимых файлов на библиотеки.
* **strings** - просмотр ASCII символов внутри любых файлов
* **ld** - компоновщик объектных файлов в исполнимые программы и динамических библиотек - не вызывается напрямую, как правило. Также называется редактором связей.
* **as** - GNU Assembler (синтаксис AT&T)
* **ar** - архиватор объектных файлов в статические библиотеки (используется довольно редко). Также на его базе строят пакеты ОС (rpm, deb файлы).
* **objdump** - подробно показывает поля объектных (и исполнимых) файлов.
* **readelf** - даёт представление о содержимом исполнимого файла (формат .ELF).

Пример просмотра полей заголовка исполняемого файла `str1`:

```
readelf --syms str1

Symbol table '.dynsym' contains 8 entries:
   Чис:    Знач           Разм Тип     Связ   Vis      Индекс имени
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5 (2)

...

objdump --syms str1

str1:     формат файла elf64-x86-64

SYMBOL TABLE:
0000000000000238 l    d  .interp	0000000000000000              .interp
0000000000000254 l    d  .note.ABI-tag	0000000000000000              .note.ABI-tag
0000000000000274 l    d  .note.gnu.build-id	0000000000000000              .note.gnu.build-id
0000000000000298 l    d  .gnu.hash	0000000000000000              .gnu.hash
00000000000002b8 l    d  .dynsym	0000000000000000              .dynsym
0000000000000378 l    d  .dynstr	0000000000000000              .dynstr

...

```

Важным среди прочих полей - является точка входа с меткой main.

При сборке программы с отладочной информацией получаем такой результат:

```

objdump -d str1

str1:     формат файла elf64-x86-64


Дизассемблирование раздела .init:

0000000000000528 <_init>:
 528:	48 83 ec 08          	sub    $0x8,%rsp
 52c:	48 8b 05 b5 0a 20 00 	mov    0x200ab5(%rip),%rax        # 200fe8 <__gmon_start__>
 533:	48 85 c0             	test   %rax,%rax
 536:	74 02                	je     53a <_init+0x12>
 538:	ff d0                	callq  *%rax
 53a:	48 83 c4 08          	add    $0x8,%rsp
 53e:	c3                   	retq

...

```

Смотрим переносимый двоичный код платформ (.Net/Mono) собранный из языка C#.

```
objdump -d /usr/lib/keepass2/KeePass.exe|head -20

/usr/lib/keepass2/KeePass.exe:     формат файла pei-i386


Дизассемблирование раздела .text:

00402000 <.text>:
  402000:	10 33                	adc    %dh,(%ebx)
  402002:	31 00                	xor    %eax,(%eax)
  402004:	00 00                	add    %al,(%eax)
  402006:	00 00                	add    %al,(%eax)
  402008:	48                   	dec    %eax
  402009:	00 00                	add    %al,(%eax)
  40200b:	00 02                	add    %al,(%edx)
  40200d:	00 05 00 e0 19 22    	add    %al,0x2219e000
  402013:	00 00                	add    %al,(%eax)
  402015:	19 0f                	sbb    %ecx,(%edi)
  402017:	00 09                	add    %cl,(%ecx)
  402019:	00 00                	add    %al,(%eax)
  40201b:	00 f2                	add    %dh,%dl

```





Т.е. это ни что иное как код ассемблера (синтаксис AT&T) с комментариями.

Код делится на сегменты, данные, исполнимая часть ...

```
readelf --segments str1

Тип файла ELF — DYN (Совм. исп. объектный файл)
Entry point 0x580
There are 9 program headers, starting at offset 64

Заголовки программы:
  Тип            Смещ.              Вирт.адр           Физ.адр
                 Рзм.фйл            Рзм.пм              Флаги  Выравн
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000001f8 0x00000000000001f8  R      0x8
  INTERP         0x0000000000000238 0x0000000000000238 0x0000000000000238
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x00000000000008c0 0x00000000000008c0  R E    0x200000
  LOAD           0x0000000000000db0 0x0000000000200db0 0x0000000000200db0
                 0x0000000000000260 0x0000000000000268  RW     0x200000
  DYNAMIC        0x0000000000000dc0 0x0000000000200dc0 0x0000000000200dc0
                 0x00000000000001f0 0x00000000000001f0  RW     0x8
  NOTE           0x0000000000000254 0x0000000000000254 0x0000000000000254

...

```



## Языки программирования

Программирование - процесс создания человеком текста, понятного ему и другим людям
из которого можно машинным способом создать код для машины или файл для
перепрограммируемых вентлильных матриц (FPGA).

Список знакомых актуальных языков программирования по алфавиту:

* Bash, Shell
* C
* C# (‘c#’, ‘cs’)
* C++ (‘c++’, ‘cpp’, ‘cxx’, ‘cc’)
* Clipper, DBase, FoxPro...
* Clojure
* CMake
* CSS
* D
* Diff (‘patch’, ‘diff’)
* Erlang
* F# (‘f#’, ‘fs’)
* Fortran
* Go
* Haskell
* HTML, XML
* Java
* JavaScript (‘js’, ‘javascript’)
* Lisp, Scheme
* Lua
* Objective C (‘objc’, ‘objectivec’)
* Pascal, Delphi
* Perl
* PHP
* Python
* Ruby
* Rust
* Scala
* SQL
* TeX
* Vala

Эволюция языков программирования ведёт начало от фикцации двоичного кода под запись в файлы.
Сначала, программа для исполнения создавалась как есть: т.е. в ячейки памяти или
устройства постоянного хранения данных (например, [перфокарты](https://ru.wikipedia.org/wiki/%D0%9F%D0%B5%D1%80%D1%84%D0%BE%D0%BA%D0%B0%D1%80%D1%82%D0%B0)) забивались числовые коды:

* операций процессора
* регистров
* адресов ячеек памяти
* констант
* и др. виды кода

### Язык ассемблера и макроподстановки

Человеку заучивать коды операций и регистров процессора довольно сложно, поэтому придумали 
вместо того, чтобы вписывать в память эти все числа - использовать запись сокращёнными словами (т.н. мнемониками)
которые запомнить гораздо проще. Например, сложить - ADD (Addition), скопировать (перенести) в регистр значение - MOV (Move).
Также не удобно рассчитывать для адресной арифметики смещения и длины переменных.
Вместо этого переменные в тексте программы на ассемблере отмечают **метками** и вписывают в операнды инструкций
не переменные, а эти самые метки и смещения.

Современные популярные реализации Ассемблера:

* GNU Assembler (gas) из комплекта binutils. Имеет синтаксис AT&T.
* NetWide AsseMbler (NASM) . Классический Intel-синтаксиса ассмблер.
* Fast (Flat) AsseMbler (FASM) -  быстрый плоский ассемблер для ПК.

### Первые переносимые языки прикладного программирования

* 1954 - [Fortran](https://ru.wikipedia.org/wiki/Фортран) - математические научные вычисления. **Долгожитель**:
	- [GNU Fortran Compiler](https://gcc.gnu.org/fortran/) - в основном комплекте сборного компилятора GNU
	- [Nvidia HPC Fortran Compiler](https://developer.nvidia.com/hpc-compilers) - массивные параллельные вычисления
	- [Intel® Fortran Compiler](https://software.intel.com/content/www/us/en/develop/tools/oneapi/components/fortran-compiler.html) - ПО от флагмана процессорного рынка
	- Последний стандарт датируется 2018 годом
* 1958 - Algol - универсальный ЯП при разработке Джона Бекуса
* 1959 - Cobol - малопопулярный язык банковских транзакций. Доступен в виде [GnuCobol](https://gnucobol.sourceforge.io/) - но как сторонний проект. Ранее: open-cobol .
* 1964 - Basic ( Beginner’s All-purpose Symbolic Instruction Code) - язык общего назначения для начинающих непрограммистов. Стоял в школах и на домашних компьютерах (в ПЗУ). Современные варианты: VB и [Basiс-256](https://ru.wikipedia.org/wiki/BASIC-256).
* 1969 - Началась разработка первого и образцового ОО-языка программирования [SmallTalk](https://ru.wikipedia.org/wiki/Smalltalk).
* 1970 - Pascal - придумал Никлаус Вирт. Для обучения студентов ВУЗов быть программистами. Есть современная реализация Free Pascal Compiler со следующими возможностями:
	- TUI среда fp-ide совместимая с Turbo Pascal, Borland Pascal имеет встроенный компилятор
	- fpc - сама утилита компиляции, совместимая с аналогами ранее названых сред и Object Pascal, Delphi
	- Free Vision - реализация коммерческой TUI-библиотеки Turbo Vision
	- Free Component Library - открытая библиотека, совместимая с Delphi-коммерческими библиотеками
* 1972 - С - язык для переноса Unix
	- gcc - идёт с комплектом GNU Collection Compiler, как основной
	- clang - на базе LLVM
	- tcc (Tiny C Compiler) - позволяет интерпретацию Си
	- [sdcc](https://ru.wikipedia.org/wiki/Small_Device_C_Compiler) - для кода микроконтроллеров
* 1972 - Prolog (Programming Logically) - язык декларативного программирования
* 1972 - Представлена первая реализация (3 года спустя) языка SmallTalk. Сейчас есть версия GNU Smalltalk (gst).
* 1977 - Borne Shell - классическая оболочка UNIX. Современная реализация: bash
* 1983 - C++ - надмножество C.
* 1983 - Ada - первые реализации концепции языка параллельного программирования. До сих пор идёт в комплекте с GCC под именем GNAT
* 1983 - Objective-C - язык из Next перешедший в Apple
* 1987 - Perl - универсальный скриптовой язык, не только для составления отчётов . Активная 5 версия, готовится 7.
* 1987 - Erlang/OTP - функциональный язык ПП для телекома.
* 1988 - TCL - один из первых скриптовых языков
* 1990 - Haskell - функциональный ЯП - хорошо распараллеливает код
* 1991 - Python
* 1993 - Lua - простой язык, встраивается в прогаммы удобно
* 1995 - Java
* 1995 - PHP  - популярен для вставок HTML
* 1995 - Ruby
* 1996 - R - язык для анализа данных и обработки статистики - мощная платформа
* 1996 - OCaml - мультипарадигменный язык
* 2000 - D - Си-подобный ООП язык. Используются с Solaris-10 для трассировки (D-Trace) системных вызовов.
* 2002 - Scratch - язык программирования для обучения детей, с возрастом вхождени ниже Basic
* 2004 - Groovy - фунциональный ЯП на платформе Java
* 2005 - [F#](https://fsharp.org/) - язык функционального программирования от Microsoft доступен в Linux
* 2006 - Windows PowerShell - современная командная оболочка Microsoft , доступна в исходных кодах
* 2006 - BASIC256 - современный Basic для настольных и мобильных программ (есть в проекте F-Droid)
* 2007 - Vala - C# в среде GNOME
* 2007 - Closure - язык Lisp поверх виртуальной машины Java
* 2009 - Go - системный язык программирования, задуман быть заменой Си. Разрабатывается Google и Робом Пайком и Кеном Томпсоном из Bell Labs - лаборатория Lucent / AT&T.
* 2010 - Rust - **системный** ЯП придуман организацией Mozilla, рассматривается Microsoft
* 2011 - Dart - замена JS от Google . Им занимается Джонатан Шварц - бросил свой Perl
* 2011 - Kotlin - для разработки Android приложений, продвигает JetBrains - лучше Java по мнению многих.
* Ada
* 2012 - Elixir - надстройка на Erlang. Реализует похожий синтаксис Ruby, Closure.
* 2014 - Swift - новая технология Apple: позволяет запускать код, как интерпретируемый скрипт. Развитие Objective-C
* 2015 - Raku - 6-я версия Perl выродилась в отдельный язык

### Язык программирования Си

Язык был придуман Б. Керниганом и Д. Ритчи для переноса кода ОС Unix на разную архитектуру (тогда PDP-10 - 11)
в 1972 году.
До этого, получается, два года Unix разрабатывался на Assembler  и каждый раз при смене архитектуры
переписывался.
В настоящее время существует стандарт C11. Си называют Ассемблером высокого уровня. Я бы дал такое определение - 
язык близкий к тому, что происходит на самом деле с вычислительным процессом (двоичным кодом), но с 
минимальными переносимыми между компьютерными архитектурами возможностям.
Код на Си используется для системых вызовов ядра, самого ядра, библиотеки Си и библиотеки математических
операций с плавающей точкой.

### Objective-C

В начале 1980х годов Брэдом Коксом и Томом Лов в компании Productivity Products International (PPI) был
разработан [Objective-C](https://ru.wikipedia.org/wiki/Objective-C) как язык совместимый с ANSI C.
На становление языка повлиял Smalltalk, т.к. в последнем была концепция повторного использования кода
и объектно-ориентированная парадигма, которая повлияла и на C++. Работы начинались как препроцессор языка Си,
а позже была сформирована компонентная библиотека классов Objective-C.

В 1995 году права на язык были приобретены компанией Next, основанную Стивом Джобсом в период
временного изгнания из Apple. А позже сама Apple Computer купила Next со всеми её правами
на интеллектуальную собственность.

Интересная особенность языка - парадигма объектно ориентированного программирования, когда части программы
классы (объекты) посылают друг-другу сообщения вызывая методы, меняющие их отдельные состояния и всю
внутреннюю вычислительную
среду процесса в целом.

Сейчас Objective-C на экосистеме Apple уступает пальму первенства языку Swift, но всё
же ещё очень популярен для разработки приложений macOS и iOS и где то даже не заменим.
Разработка кода ведётся программистами с использованием интегрированной среды Xcode.

В 1988 году интерес к языку проявляют участники проекта GNU и включают компонент френтенда Obj-C
в сборный компилятор GCC.
Библиотека классов NextStep была реализованна, как GNUStep и жива до сих пор.
Например, она применяется для оконной среды WindowMaker и также ряда приложений под неё в Linux.
Некодна, WindowsMaker был средой по-умолчанию в дистрибутиве Debian, позже заменился на GNOME.
Интересная история, что в компании Next совместно с SUN Microsystems разрабатывали
на базе NextStep в 1994 году OpenStep, которая не получила дальнейшего развития.


### C++

Придуман Бёрном Страуструпом. В начале назывался Си с классами.
По сути современный Си++ - это надмножество Си и не только за счёт введения ООП и классов.
Сегодня C++ по популярности превзошёл своего предшественника C.
На нём создано львиное количество ПО. Причём , поскольку это ПО также как и код 
C - компилируется заранее - то и размер и скорость его работы - достаточно высоки,
особенно если использовать профилировщик.
На C++ написано множество приложений с графическими интерфейсными компонентами (виджетами):

* FLTK
* Qt
* wxWidgets
* FOX
* xForms

...



### C# - один из наследников языков C/C++

Код, созданный на C# транслируют сначала в 
двоичный байт код.
Потом этот байт-код подвергается [JIT-компиляции](https://ru.wikipedia.org/wiki/JIT-%D0%BA%D0%BE%D0%BC%D0%BF%D0%B8%D0%BB%D1%8F%D1%86%D0%B8%D1%8F) .
Выполняется это всё в т.н. среде [Common Language Runtime](https://ru.wikipedia.org/wiki/Common_Language_Runtime) .

На Windows - среда называется "платформа .Net".
В Linux - это ПО Mono, разработанное соавтором проекта GNOME - Мигелем Де Икасо.
В настоящее время он работает в основанной им компании [Xamarin](https://ru.wikipedia.org/wiki/Xamarin),
которую приобрела Microsoft, и в дальнейшем будут продолжать проводиться работы по
сближению продуктов .Net и Mono , которые в итоге оба будут лицензироваться по 
варианту [M.I.T.](https://ru.wikipedia.org/wiki/%D0%9B%D0%B8%D1%86%D0%B5%D0%BD%D0%B7%D0%B8%D1%8F_MIT).
Выдающимся достижением здесь является инструмент Xamarin.Forms,
позволяющий создавать пользовательский интерфейс из набора визуальных элементов, описываемых на языке разметки XAML.

Можно работать в C# в режиме интерпретатора [REPL](https://ru.wikipedia.org/wiki/REPL).
Для этого есть интерпретатор `csharp` (пакет mono-csharp-shell).

В Debian/Ubuntu устанавливаем базовую функциональность компилятора:

	sudo apt install mono-mcs

Пишем текст C#  в файл `hello_console.cs`:

```csharp
using System;

public class HelloWorld
{
    public static void Main(string[] args)
    {
        Console.WriteLine ("Hello Mono World");
    }
}
```

Компилируем:

	mcs hello_console.cs

Запускаем и получаем результат прогона программы на текстовую консоль:

	./hello_console.exe
	Hello Mono World


Для того, чтобы разрабатывать настольные GUI-приложения ставим пакет разработки под Mono:

	sudo apt install mono-devel

После этого можно составить такой простейший файл (пусть будет `hello_win.cs`)
для демонстрации оконной системы:

```csharp
using System;
using System.Windows.Forms;

public class HelloWorld : Form
{
    static public void Main ()
    {
        Application.Run (new HelloWorld ());
    }

    public HelloWorld ()
    {
        Text = "Hello Mono World";
    }
}
```

Компилируем с подключением динамической библиотеки Windows.Forms:

	mcs -r:System.Windows.Forms.dll hello_win.cs

Запускаем:

	./hello_win.exe

открывается графическое приложение с заголовком окна "Hello Mono World".

Для разработки Веб приложений на платформе .Net/Mono используем [XSP](https://en.wikipedia.org/wiki/XSP_(software)) сервер:

	sudo apt install mono-xsp4

Напишем текст динамической страницы, который отобразит календарь 
в файл `hello.aspx`:

```asp
<%@ Page Language="C#" %>
<html>
<head>
	<meta charset="utf-8">
	<title>Sample Calendar</title>

</head>
<asp:calendar showtitle="true" runat="server">
</asp:calendar>

```

Запускаем вебсервер для отображения динамической ASPX страницы:


```sh
xsp4 --port 9000
xsp4
Listening on address: 0.0.0.0
Root directory: /home/dron/languages/CSharp
Listening on port: 9000 (non-secure)
Hit Return to stop the server.
```

Открываем веб-браузер по адресу: http://localhost:9000/hello.aspx

Будет отображён календарь:

```
<	Сентябрь 2021	>
Пн	Вт	Ср	Чт	Пт	Сб	Вс
30	31	1	2	3	4	5
6	7	8	9	10	11	12
13	14	15	16	17	18	19
20	21	22	23	24	25	26
27	28	29	30	1	2	3
4	5	6	7	8	9	10
```

Нажимаем в консоли клавишу *<Enter>*.

Также можно поставить примеры web-приложений из комплекта Mono:

	sudo apt install asp.net-examples

после этого их можно смотреть по адресу: http://localhost:8084/samples/

Для разработки красивых приложений с видом как в Линукс-дистрибутивах
можно использовать кроссплатформенные виджеты библиотеку GTK:

	sudo apt install gtk-sharp3-examples monodoc-gtk3.0-manual libgtk3.0-cil-dev gtk-sharp3

В файл `hello_gtk.cs` запишем код на языке C#:

```csharp
using Gtk;
using System;

class Hello
{
    static void Main ()
    {
        Application.Init ();

        Window window = new Window ("Hello Mono World");
        window.Show ();

        Application.Run ();
    }
}
```

Собираем с пакетом GTK-3 для языка C#:

	mcs hello_gtk.cs -pkg:gtk-sharp-3.0

Запускаем командой `./hello_gtk.exe` или `mono hello_gtk.exe` .
Также как и в примере выше с Winows.Forms - запускается окно с тем же заголовком.
Чтобы закрыть приложение - мало нажать на соответствующий системный виджет:
придётся ещё в консоли нажать клавишу *Ctrl-C*.

Если технология mono вас так увлекла, то имеет смысл поставить полный пакет:

	sudo apt install mono-complete


### Современные компилируемые языки программирования

#### Go-lang

Зародился язык Go в Google.
Цель - решить на уровне синтаксиса и комплектных библиотек
задачу писать многопотчный код.


Практиковаться можно [в режиме онлайн](https://repl.it/languages/go)


Установка в Debian/Ubuntu будет такой:

```sh
apt-get update
apt-get dist-upgrade
apt-get install golang  gccgo gccgo-*-doc
```

Пакеты курируются:

* golang\* - компанией [Google](https://golang.org/project) при сопровождении вместе с волонтёрами проекта
* gccgo\*  - проектом [GNU](https://gcc.gnu.org/onlinedocs/gccgo/) и фондом СПО

Посмотреть номер версии:

	go version

Получить справку по использованию программы go:

	go help

Переменные:

* GOPATH - каталог с файлами разработки Go. Обычно значение: $HOME/go. С подкаталогами:
	- $GOPATH/src - сюда кладут исходники
	- GOBIN - туда собираются исполняемые процессором в ОС модули = $GOPATH/bin

Пример простейшей программы.

Создаём файл main.go с примерно таким содержимым:

```go
package main

import "fmt"

func main() {
        fmt.Println("Hello, World")
}
```

Запустить на исполнение программу на исходном тексте:

	go run исходный_файл

Например, запустим программу main.go, написанную в текстовом редкаторе :

	go run main.go
	Hello, World


Сборка программы:

	go build Программа.go

на выходе получаем исполняемый файл Программа.

Для нашего примера:

	go build main.go
	./main
	Hello, World


#### Rust

Язык системного программирования придуманный Mozilla, но поддерживаемый Microsoft,
рядом корпораций и силами сообщества.
По задумке Rust должен стать лучше Си, т.к. он делает упор на
большую безопасность при работе с памятью, что обеспечивается встроенными в язык механизмами контроля ссылок.


В Debian/Ubuntu ставим пакеты:

	sudo apt install rustc
	sudo apt install cargo


Проверяем версии:

	rustc --version
	cargo --version

cargo   создаёт каталог с файловой инфраструктурой проекта, позволяет использовать заготовки и собирать части в исполняемые модули и запускать их.
Пример создания одноимённой папки проекта hello-rust:

	cargo new hello-rust --bin

В подкаталоге src, папки проекта правим код.
Файлам исходных текстов языка Rust принято давать расширение \*.rs

Для сборки используем команду в виде:

	cargo build

Для сборки используем команду в виде:

	cargo run

В CentOS 8 ищем пакет, предоставляющий компилятор языка Rust: 

	yum provides /usr/bin/rustc

По найденному пакету (rust,  в данном случае) запускаем установку:

	yum install rust

Ищем пакет, предоставляющий сборочное средство и менеджер пакетов языка Rust - cargo: 

	yum provides /usr/bin/cargo

По найденному пакету (cargo,  в данном случае) запускаем установку:

	yum install cargo

Переходим в каталог проектов и создаём первый, например, с именем hello-rust:

	cargo new hello-rust --bin

Получаем одноимённый каталог (hello-rust из примера), с базой git и подкаталогом src исходным кодом.


### Интерпретируемые языки программирования


#### Basic

На платформе .Net/Mono работает Visual Basic.
Установка на Debian/Ubuntu:

	sudo apt install mono-vbnc

Пишем код программы по [образцу](https://www.tutlane.com/tutorial/visual-basic/vb-hello-world-program)
в файл `Program.vb`:

```basic
Imports System

Module Module1
    Sub Main()
        Console.WriteLine("Hello World!")
        Console.WriteLine("Press Enter Key to Exit.")
        Console.ReadLine()
    End Sub
End Module
```

Компилируем:

	vbnc Program.vb

Запускаем:

	mono Program.exe

Из популярных в наше время реализаций, ещё хотелось бы упомянуть [BASIC-256](https://ru.wikipedia.org/wiki/BASIC-256),
который доступен как в виде настольных приложений и довольно неплохо документирован.

Из мобильных реализаций, например [X11-Basic в репозитории F-Droid](https://f-droid.org/ru/packages/net.sourceforge.x11basic/).

#### Perl

Язык программирования Perl создан лингвистом Larry Wall'ом. Он удобен для программирования в разных парадигмах: алгоритмической, объектно-ориентированнной, структурированной и т.д. Весьма полезны в нём регулярные выражения.

Особенностью языка является различная интерпретация его конструкций в зависимости от контекста
и в чём то схожесть с естественным языком.

Пример условного выражения:

```perl
# если условие истинное - завершить скрипт с кодом 1
exit 1 if $a > $b;
```

Или тоже самое в типичном программистском стиле:

```perl
# если условие истинное - завершить скрипт с кодом 1
if($a > $b){
	exit 1 ;
}
```

Т.е. язык довольно вариативен в том плане,
что своим слоганом избрал [There’s More Than One Way To Do It](https://ru.wikipedia.org/wiki/TMTOWTDI), или по-русски:
«Есть больше одного способа сделать это», это придумал Ларри Уолл.

"Простые вещи должны быть простыми, а сложные вещи должны быть возможными" - это любит повторять на
своих лекциях Джонатан Шварц.

Современный рабочий perl в своём развитии остановился на версии 5. Но он используется, как системное средство во многих
базовых частях Unix'оподобных дистрибутивов. Например, в системах установки ОС и управления пакетами.

Язык имеет большую библиотеку [Comprehensive Perl Archive Network](https://ru.wikipedia.org/wiki/CPAN) с модулями
позволяющими с помощью perl-скритов делать всё, что угодно.

В наши дни дальнейшим развитием Perl-5 видится проект Perl-7, который пока не "выкачен на публику".
А Perl-6 внезапно для себя стал новым языком [Racku](https://ru.wikipedia.org/wiki/Raku).

#### Python

В настоящее время превзошёл своего конкурента Perl по популярности.
Версия 2.x.y объявлена, как не поддерживаемая.
Не рекомендуется её использовать, т.к. отсутствует поддержка Unicode - т.е. могут возникнуть проблемы
с русским языком.
Рабочая - версия ветки 3.\*
Поэтому интерпретатор:

	/usr/bin/python3

Среда разработки в комплекте:

	/usr/bin/idle

Или `idle3`, если в ОС всё ещё поддерживается параллельно разработка на 2-й версии.

IDLE - Integrated Development & Learning Environment, основана на библиотеке [TkInter](https://ru.wikipedia.org/wiki/Tkinter) .




#### R

Язык и проект с большим количеством модулей
для анализа данных, классической статистики, инфографики.
В идеале конкурирует с Python/NumPy...

Практиковаться тут можно интерактивно на [Repl.it](https://repl.it/languages/rlang)

Все пакеты для языка программирования R начинаются с "r-" в Debian/Ubuntu.

Обязательно надо выполнить команду чтобы как-то начать работать:

	sudo apt-get install r-base r-base-dev

После этого можно с сайта rstudio.com поставить rstudio-desktop с Qt-интерфейсом или rstudio-server.
Там будет консоль.

Основной пакет r-base-core содержит  интерпретатор /usr/bin/R . Это консоль для языка R.

Посмотреть остальные пакеты можно командой:

	sudo apt-cache pkgnames r-


Конфигурационные файлы R-проекта берутся из пакета r-base-core: /etc/R/Rprofile.site

Настройки пользователя: ~/.Rprofile

Синтаксис консоли R:

	R[опции]

Среди опций:

* -e выражение - выполнить выражение в командной строке. Работает как калькулятор. Например: R -e '2\*2'
* -f файл - выполнить инструкции из файл'а.
* -h, --help - получить справку
* --version - получить версию пакета и консоли 
* -q, --quiet - работать без лишних сообщений
* --verbose - наоборот - информировать пользователя

Т.о. в начале каждого файла со скриптом должно быть:

	#!/usr/bin/R -f


Если запускать консоль R интерактивно, то доступны следующие функции:

	help.start()

Запускает Web-сервер и открывает на него браузер - можно листать справку.

	demo()

Запускает пакеты демонстраций.

	demo(graphics)

запускает демонстрацию графики.



R Commander ставится командой:

	apt install r-cran-rcmdr


Запускаем R-консоль:

	/usr/bin/R

В ней запускается R Commander так:

	library(Rcmdr)


Будут установлены дополнительные модули для работы в подкаталог $HOME/R


В RStudio, которая скачивается отдельно есть консоль.
Можно вводить команды.
Например: 1+1

Можно создавать в среде RStudio файлы. 
Чтобы начать редактирование - введите  Ctrl-Shift-N.
И набирайте пустой файл.
Его можно можно запускать (Ctrl-Enter) или соотв. кнопка.
Это запускает текущую строку, либо выделение.

Построение графиков
Для построение графиков хороша библиотека ggplot2 (пакет).
Ставим в Ubuntu/Debian:

	sudo apt install r-cran-ggplot2



#### Erlang/OTP



## Источники

* [G-код](https://ru.wikipedia.org/wiki/G-code)
* [Числовое программное управление](https://ru.wikipedia.org/wiki/%D0%A7%D0%B8%D1%81%D0%BB%D0%BE%D0%B2%D0%BE%D0%B5_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%BD%D0%BE%D0%B5_%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5)
* [Компьютер](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BC%D0%BF%D1%8C%D1%8E%D1%82%D0%B5%D1%80)
* [Компьютерная память](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BC%D0%BF%D1%8C%D1%8E%D1%82%D0%B5%D1%80%D0%BD%D0%B0%D1%8F_%D0%BF%D0%B0%D0%BC%D1%8F%D1%82%D1%8C)
* [Постоянное запоминающее устройство](https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%BD%D0%BE%D0%B5_%D0%B7%D0%B0%D0%BF%D0%BE%D0%BC%D0%B8%D0%BD%D0%B0%D1%8E%D1%89%D0%B5%D0%B5_%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D0%BE)
* [Компьютерная программа](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BC%D0%BF%D1%8C%D1%8E%D1%82%D0%B5%D1%80%D0%BD%D0%B0%D1%8F_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0)
* [Двоичный код](https://ru.wikipedia.org/wiki/%D0%94%D0%B2%D0%BE%D0%B8%D1%87%D0%BD%D1%8B%D0%B9_%D0%BA%D0%BE%D0%B4)
* [Машинный код](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D1%88%D0%B8%D0%BD%D0%BD%D1%8B%D0%B9_%D0%BA%D0%BE%D0%B4)
* [Регистр процессора](https://ru.wikipedia.org/wiki/%D0%A0%D0%B5%D0%B3%D0%B8%D1%81%D1%82%D1%80_%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%BE%D1%80%D0%B0)
* [Арифметико-логическое устройство](https://ru.wikipedia.org/wiki/%D0%90%D1%80%D0%B8%D1%84%D0%BC%D0%B5%D1%82%D0%B8%D0%BA%D0%BE-%D0%BB%D0%BE%D0%B3%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5_%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D0%BE)
* [Язык ассемблера](https://ru.wikipedia.org/wiki/%D0%AF%D0%B7%D1%8B%D0%BA_%D0%B0%D1%81%D1%81%D0%B5%D0%BC%D0%B1%D0%BB%D0%B5%D1%80%D0%B0)
* [Низкоуровневый язык программирования](https://ru.wikipedia.org/wiki/%D0%9D%D0%B8%D0%B7%D0%BA%D0%BE%D1%83%D1%80%D0%BE%D0%B2%D0%BD%D0%B5%D0%B2%D1%8B%D0%B9_%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)
* [Инструкция (информатика)](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%86%D0%B8%D1%8F_(%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%82%D0%B8%D0%BA%D0%B0))
* [Макрос в языке ассемблера](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D0%BA%D1%80%D0%BE%D1%81_%D0%B2_%D1%8F%D0%B7%D1%8B%D0%BA%D0%B5_%D0%B0%D1%81%D1%81%D0%B5%D0%BC%D0%B1%D0%BB%D0%B5%D1%80%D0%B0)
* [Комбинационная логика](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BC%D0%B1%D0%B8%D0%BD%D0%B0%D1%86%D0%B8%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F_%D0%BB%D0%BE%D0%B3%D0%B8%D0%BA%D0%B0)
* [Секвенциальная логика](https://ru.wikipedia.org/wiki/%D0%A1%D0%B5%D0%BA%D0%B2%D0%B5%D0%BD%D1%86%D0%B8%D0%B0%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F_%D0%BB%D0%BE%D0%B3%D0%B8%D0%BA%D0%B0)
* [Триггер](https://ru.wikipedia.org/wiki/Триггер)
* [Исполняемый файл](https://ru.wikipedia.org/wiki/%D0%98%D1%81%D0%BF%D0%BE%D0%BB%D0%BD%D1%8F%D0%B5%D0%BC%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB)
* [Executable and Linkable Format](https://ru.wikipedia.org/wiki/Executable_and_Linkable_Format)
* [Portable Executable](https://ru.wikipedia.org/wiki/Portable_Executable)
* [GNU Binutils](https://en.wikipedia.org/wiki/GNU_Binutils)
* [История языков программирования](https://ru.wikipedia.org/wiki/%D0%98%D1%81%D1%82%D0%BE%D1%80%D0%B8%D1%8F_%D1%8F%D0%B7%D1%8B%D0%BA%D0%BE%D0%B2_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)
* [Исходный код](https://ru.wikipedia.org/wiki/%D0%98%D1%81%D1%85%D0%BE%D0%B4%D0%BD%D1%8B%D0%B9_%D0%BA%D0%BE%D0%B4)
* [Алгол](https://ru.wikipedia.org/wiki/%D0%90%D0%BB%D0%B3%D0%BE%D0%BB)
* [Джон Бэкус](https://ru.wikipedia.org/wiki/%D0%91%D1%8D%D0%BA%D1%83%D1%81,_%D0%94%D0%B6%D0%BE%D0%BD)
* [Кобол](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%B1%D0%BE%D0%BB)
* [Байт-код](https://ru.wikipedia.org/wiki/%D0%91%D0%B0%D0%B9%D1%82-%D0%BA%D0%BE%D0%B4)
* [JIT-компиляция](https://ru.wikipedia.org/wiki/JIT-%D0%BA%D0%BE%D0%BC%D0%BF%D0%B8%D0%BB%D1%8F%D1%86%D0%B8%D1%8F)
* [Сверхвысокоуровневый язык программирования](https://ru.wikipedia.org/wiki/%D0%A1%D0%B2%D0%B5%D1%80%D1%85%D0%B2%D1%8B%D1%81%D0%BE%D0%BA%D0%BE%D1%83%D1%80%D0%BE%D0%B2%D0%BD%D0%B5%D0%B2%D1%8B%D0%B9_%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)
* [Высокоуровневый язык программирования](https://ru.wikipedia.org/wiki/%D0%92%D1%8B%D1%81%D0%BE%D0%BA%D0%BE%D1%83%D1%80%D0%BE%D0%B2%D0%BD%D0%B5%D0%B2%D1%8B%D0%B9_%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)
* [Метапрограммирование](https://ru.wikipedia.org/wiki/%D0%9C%D0%B5%D1%82%D0%B0%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)
* [Макрокоманда](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D0%BA%D1%80%D0%BE%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D0%B0)
* [Макропроцессор](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D0%BA%D1%80%D0%BE%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%BE%D1%80)
* [Транслятор](https://ru.wikipedia.org/wiki/%D0%A2%D1%80%D0%B0%D0%BD%D1%81%D0%BB%D1%8F%D1%82%D0%BE%D1%80)
* [.NET Framework](https://ru.wikipedia.org/wiki/.NET_Framework)
* [Common Language Runtime](https://ru.wikipedia.org/wiki/Common_Language_Runtime) - двоичный код .Net платформы
* [Common Intermediate Language](https://ru.wikipedia.org/wiki/Common_Intermediate_Language) - высокоуровневый ассемблер
* [Microsoft Visual Studio](https://ru.wikipedia.org/wiki/Microsoft_Visual_Studio)
* [Проект Mono](https://ru.wikipedia.org/wiki/Mono)
* [Интегрированная среда разработки](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%B0%D1%8F_%D1%81%D1%80%D0%B5%D0%B4%D0%B0_%D1%80%D0%B0%D0%B7%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B8)
* [Gambas](https://en.wikipedia.org/wiki/Gambas)
* [Scala (язык программирования)](https://ru.wikipedia.org/wiki/Scala_(%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F))
* [Ruby](https://ru.wikipedia.org/wiki/Ruby)
* [Erlang](https://ru.wikipedia.org/wiki/Erlang)
* [Perl](https://ru.wikipedia.org/wiki/Perl)
* [JS](https://ru.wikipedia.org/wiki/JavaScript)
* [Kotlin](https://ru.wikipedia.org/wiki/Kotlin) - язык от JetBrains для платформы Java
* [Объектно-ориентированное программирование](https://ru.wikipedia.org/wiki/%D0%9E%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%BD%D0%BE-%D0%BE%D1%80%D0%B8%D0%B5%D0%BD%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%BE%D0%B5_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)
* [Лисп](https://ru.wikipedia.org/wiki/%D0%9B%D0%B8%D1%81%D0%BF)
* [Пролог (язык программирования)](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%BE%D0%BB%D0%BE%D0%B3_(%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F))

## Литература

* [C# Notes for Professionals book](https://books.goalkicker.com/CSharpBook/)
* [Xamarin.Forms Notes for Professionals book](https://books.goalkicker.com/XamarinFormsBook/)
* [.NET Framework Notes for Professionals book](https://books.goalkicker.com/DotNETFrameworkBook/)


## Видео

* [How to Inspect Compiled Binaries (binutils, objdump)](https://youtu.be/bWMIpHVRFUo)
 
## Аудио

* [Perlcast: Podcasting Perl](http://www.perlcast.com/2005/)
* [FLOSS Weekly](https://twit.tv/shows/floss-weekly)
	- [4. Chromatic](https://twit.tv/shows/floss-weekly/episodes/4) ;  May 2nd 2006 - про Perl6
	- [9.  Randal L. Schwartz](https://twit.tv/shows/floss-weekly/episodes/9) ; Hosted by Chris DiBona, Leo Laporte; Perl 6, Parrot, and partying with Linus with Perl author and developer, Randal L. Schwartz ; Jul 14th 2006

