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

* 1954 - Fortran - математические научные вычисления
* 1958 - Algol - универсальный ЯП при разработке Джона Бекуса
* 1959 - Cobol - 
* 1970 - Pascal - придумал Никлаус Вирт. Есть современная реализация Free Pascal Compiler со следующими возможностями:
	- TUI среда fp-ide совместимая с Turbo Pascal, Borland Pascal имеет встроенный компилятор
	- fpc - сама утилита компиляции, совместимая с аналогами ранее названых сред и Object Pascal, Delphi
	- Free Vision - реализация коммерческой TUI-библиотеки Turbo Vision
	- Free Component Library - открытая библиотека, совместимая с Delphi-коммерческими библиотеками
* 1972 - С - язык для переноса Unix
	- gcc - идёт с комплектом GNU Collection Compiler, как основной
	- clang - на базе LLVM
	- tcc - позволяет интерпретацию Си
	- sdcc - для кода микроконтроллеров
* 1972 - Prolog - язык декларативного программирования
* 1977 - Borne Shell - классическая оболочка UNIX
* 1983 - C++ - надмножество C
* 1983 - Ada - до сих пор идёт в комплекте с GCC
* 1983 - Objective-C - язык из Next перешедший в Apple
* 1987 - Perl - универсальный скриптовой язык, не только для составления отчётов . Активная 5 версия, готовится 7.
* 1987 - Erlang/OTP - функциональный язык для телекома.
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


Objective-C - 

C++ - 

### C# - один из наследников языков C/C++

Код, созданный на C# транслируют сначала в 
двоичный байт код.
Потом этот байт-код подвергается [JIT-компиляции](https://ru.wikipedia.org/wiki/JIT-%D0%BA%D0%BE%D0%BC%D0%BF%D0%B8%D0%BB%D1%8F%D1%86%D0%B8%D1%8F) .



### Современные компилируемые языки программирования

Go-lang

Rust

### Интерпретируемые языки программирования

Perl
Python
R
...


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



## Видео

* [How to Inspect Compiled Binaries (binutils, objdump)](https://youtu.be/bWMIpHVRFUo)
 



