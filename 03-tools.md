# Инструменты разработки кода

Для эффективного создания качественного кода 
используют продуктивные средства разработки:

* командные оболочки
* текстовые редакторы с подсветкой синтаксиса языка программирования
* компиляторы и компоновщики
* средства автосборки с отслеживанием зависимостей модулей
* отладчики
* профилировщики
* интегрированные рабочие окружения

## Автосборка из командной строки

Прежде всего средства автосборки из командной строки
запускают препроцессоры, компиляторы, компоновщики, ассемблеры
с огромным количеством опций и параметров.

Автосборка качественного программного продукта должна преследовать такие цели:

* компиляция исполнимых модулей из исходного кода
* выполнение запуска различных регрессионных тестов с кодом
* генерацию сопроводительной документации по коду
* создание архивов с дистрибутивом программного продукта
* развёртывание ПО, в том числе в рамках непрерывной интеграции

Также на процесс сборки зависит от переменных окружения:

* Копмиляция: `$CFLAGS` - указывают каталоги с загаловочными файлами\
опция `-Iкаталог...`, отладки `-g`, предупреждения `-W...`,\
оптимизации `-Oцифра(от 1 до 3)`
* Компоновка:
	- `$LDLIBS` - перечисляются имена библиотек (`-lимя...`)
	- `$LDFLAGS` - указывает пути нестандартного хранения библиотек\
	(`-Lкаталог...`).


### make

Система сборки проектов make появилась в Unix, но и встречается
и в системах Microsoft под именем nmake.

Основное назначение программы make - это выполнение тех правил,
которые обновляют зависимые от других файлы.
Например, компиляция объектных файлов после редактирования исходных.

Правила прописываются в файле с именем `Makefile`, но можно с помощью
опции `-f сборочный_файл.mk` задать файл с любым именем.

Утилита make использует и
[неявные правила](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html).
Посмотреть их можно командой:

	make -p


Напишем простую программу "угадай строку" в файле с именем `prpp1.c`:

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main(int argc, char* argv[]){

float ss;

printf("Enter the size of the small side: ");
scanf("%f", &ss);
printf("The size of the large side is: %f\n", M_SQRT2*ss);
return EXIT_SUCCESS;
}
```

Сборка программы происходит командой

	LDLIBS=-lm make prpp1
	cc     prpp1.c  -lm -o prpp1


Если ничего не делать и повторно набрать команду, то:

	LDLIBS=-lm make prpp1
	make: «prpp1» не требует обновления.


### CMake

CMake - Crossplatform Make - это кроссплатформенный посредник между файлом
CMakeLists.txt, который управляет генерацией самого файла сборки (Makefile).
На Unix - Makefile, а на других программных системах - это файлы проектов конкретной системы сборки.

Установка пакетов для Debian/Ubuntu по списку:

* cmake - утилита сборки cmake
* cmake-doc - комплект документации по списку
* cmake-extras - дополнительные сборки
* cmake-curses-gui - сборки/конфигурации с библиотекой Curses: утилита ccmake
* cmake-qt-gui -  сборки с библиотекой Qt: утилита cmake-gui . Там настраивается входной и выходной каталог для сборки проекта.

Аббревиатуры:

* domain-specific language (DSL) - предметно-ориентированный язык (DSL)


Можно посмотреть для каких платформ делает метасборки cmake:

```
cmake -G
CMake Error: No generator specified for -G

Generators
  Unix Makefiles               = Generates standard UNIX makefiles.
  Ninja                        = Generates build.ninja files.
  Watcom WMake                 = Generates Watcom WMake makefiles.
  CodeBlocks - Ninja           = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles  = Generates CodeBlocks project files.
  CodeLite - Ninja             = Generates CodeLite project files.
  CodeLite - Unix Makefiles    = Generates CodeLite project files.
  Sublime Text 2 - Ninja       = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                               = Generates Sublime Text 2 project files.
  Kate - Ninja                 = Generates Kate project files.
  Kate - Unix Makefiles        = Generates Kate project files.
  Eclipse CDT4 - Ninja         = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles= Generates Eclipse CDT 4.0 project files.
  KDevelop3                    = Generates KDevelop 3 project files.
  KDevelop3 - Unix Makefiles   = Generates KDevelop 3 project files.
```

Т.е. по сути, CMake инструментарий позволяет вести кроссплатформенную разработку не только в разных ОС,
но и в разных средах разработки.
Он признался проектом Qt вместо их нативного в 5-й версии qmake.

Комментарии в файлах CMakeList.txt начинаются символом решётки (`#`).

Технология

Файл `CMakeLists.txt` может быть свой на каждый подкаталог проекта.

Генерация Makefile обеспечивается командой:

        cmake .

Подразумевается, что `.` указывает на тот каталог, где лежит файл `CMakeLists.txt`

Также производится сборочное окружение командой:

        cmake --build .

Его можно выполнять в отдельном каталоге.
Главное учитывать относительные списки при указании каталогов с источниками сборок.

Сборка может происходить в отдельном каталоге.

Создаём подкаталог (пусть будет `build`) в том же каталоге, где лежит `CMakefile`.
Переходим в этот подкаталог (`build`).

В нём запускам процедуру тестирования и создания метаконфигурационных файлов для сборки:

        cmake ..

родительский каталог (`..`) в данном случае указывает месторасположение исходных файлов с текстами
программ и метафайлом `CMakeLists.txt`.

Если всё удачно, то будут в подкаталоге сборки (`build`) созданы файлы кеша, логи и сборочный файл
Unix-утилиты `make` под названием `Makefile`.

Этот же процесс можно проделать с помощью только одной команды `cmake`, чтобы избежать особенностей функционирования
конкретной оболочки + специфики команд ОС. Для этого используется опция `-E` (**E**xecute command ?  - режим команд - выполнения).

Создадим каталог сборки (`build`):

        cmake -E make_directory build

Сменить каталог (`chdir build`) и запустить в нём сборку из конфигурации собранной в каталог выше (`cmake ..`):

        cmake -E chdir build cmake ..

Все эти команды выполняет cmake  в своём процессе, а это значит, что после завершения этого процесса, управление от cmake
возвращается обратно в оболочку. Т.е. мы остаёмся в текущем каталоге.
Поэтому, чтобы выполнить команды в сборочном подкаталоге (`build` для нашего примера):

        cmake --build build


Синтаксис CMake позволяет указывать размещение проекта
в каталогах с разным назначением. Создадим эти каталоги командой cmake:

	cmake -E make_directory include
	cmake -E make_directory src

В каталоге `include` разместим файл `foo.h` с таким тривиальным содержимым:

```c++
void foo();

```

В `src` - `foo.cc` запишем определение функции `foo()`:

```c++
#include <iostream>
#include "foo.h"

void foo()
{
    std::cout << "Hello World!\n";
}
```

Остаётся для примера добавить точку входа - функцию `main()` в одноимённый файл `main.cc` в тот же подкаталог (`src`):

```c++
#include "foo.h"

int main()
{
    foo();
    return 0;
}
```

В рабочем каталоге создаём сборочный файл `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 2.4)
project(hello_world)
include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${PROJECT_SOURCE_DIR}/src)
include_directories(${PROJECT_SOURCE_DIR})
add_library(applib src/foo.cc)
add_executable(app src/main.cc)
target_link_libraries(app applib)
```

Создаём сборочный каталог:

	cmake -E make_directory build

Переходим в него и собираем `Makefile`:

```
cmake -E chdir build cmake ..
-- The C compiler identification is GNU 4.9.2
-- The CXX compiler identification is GNU 4.9.2
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/dron/languages/C++/CMake5/build
```

Окончательно собираем проект утилитой make:

```
cmake -E chdir build make
Scanning dependencies of target applib
[ 50%] Building CXX object CMakeFiles/applib.dir/src/foo.o
Linking CXX static library libapplib.a
[ 50%] Built target applib
Scanning dependencies of target app
[100%] Building CXX object CMakeFiles/app.dir/src/main.o
Linking CXX executable app
[100%] Built target app
```


Запускаем полученное приложение :

```
./build/app
Hello World!
```

### GNU Autotools

При перенесении исходного кода программ с одной машины на другую недостаточно
оказалось только правильной трансляции в нативный (объектный) код самого процессора.
Нужно было учитывать различные особенности того или иного окружения: в основном,
библиотек кода, но также вариантов компиляторов, их опций и т.д.
Т.о. чтобы правильно собрать код без экспертных знаний,
разрабатывался набор скриптов командной оболочки.
Потом к ним добавился макропроцессор m4, и появилось большое количество заготовок-шаблонов
для генерации корректных (конкретных) сборочных скриптов и make-файлов.
И вся эта конструкция оформилась в такие пакеты ПО:

* [autoconf](https://ru.wikipedia.org/wiki/Autoconf)
* automake
* libtool
* gnulib

Средства autotools не только решают вопросы  с правильными командными строками, путями и опциями сборочных утилит,
но и определяют на уровне исходного кода Си набор подключаемых файлов и макросов.

В целом ПО разрабатывемое по технологии GNU Autotools запаковывается в сжатый tar-архив.
На целевой платформе этот файл распаковывается и в образовавшемся дереве подкаталогов
(в самом верхнем уровне) выполняются такие действия:

```sh
./configure --prefix=куда_ставим
make
make install
```


### Apache Ant

Apache Ant - сборочное средство проектов на Java. 
Файл управляющий процессом автосборки и развёртывания программного продукта пишется на языке разметки XML.
Ant легко вызывается из командной строки и может интегрироваться с бесплатными и коммерческими IDE.
Популярная среда и Apache Ant тесно интегрированы друг с другом с помощью системы плагинов.
Например: [Apache IvyDE](https://ant.apache.org/ivy/ivyde/screenshots.html).

Для корректной работы Ant нужно установить переменную среды `$JAVA_HOME`.


Смотрим версию:

	apt show ant -a


Пример сборочного файла:

```xml
<?xml version = "1.0"?>
<project name = "Hello World Project" default = "info">
   <property name = "sitename" value = "www.tutorialspoint.com"/>

   <target name = "info">
      <echo>Apache Ant version is ${ant.version} - You are at ${sitename} </echo>
   </target>
</project>
```


### Прочие системы автосборки

**ninja-build** - похожа на CMake, Make используется для сборки проектов на *C++*

**scons** - сборочная система на Python написанная, но позволяет собирать проекты и на других языках.

**GYP** (от англ. Generate Your Projects) — система автоматизации сборки, созданная корпорацией Google для генерации проектов различных IDE (таких как Visual Studio Code или Xcode).

**Meson** - ещё одна система написанная на Python. Применяется для сборки разных проектов на разных языках. Находит применение для генерации ОС во встраиваемые системы.

## Интерактивная разработка

Ещё раз, но детальней: редакторы vim, emacs, командная оболочка Борна (bash, tcsh, zsh).

### Легковесы: Geany, Codelite, Code::Blocks



### QtCreator

### Среда в комплекте Python: IDLE . Дополнительная среда: Eric

### R: RStudio

### От Oracle к Apache: NetBeans

### Выходец из IBM: Eclipse

### Продукция JetBrains: IntelliJ IDEA, PyCharm

### Прочие полукоммерсанты: Sublime Text, Brackets
### Неизменный XCode от Apple
### MS Visual Studio: Community edition - переносимая среда на Electron.JS
### Notepad++ для любителей Windows


## Источники

* [Документация по CMake](https://cmake.org/documentation/)
* [CMake Tutorial](https://cmake.org/cmake-tutorial/)
* Указание на сборку и линковку библиотеки:
        - [add\_library](https://cmake.org/cmake/help/v3.0/command/add_library.html)
        - [target\_link\_libraries](https://cmake.org/cmake/help/v3.0/command/target_link_libraries.html)
* ["Hello World" with multiple source files](https://riptutorial.com/cmake/example/22391/-hello-world--with-multiple-source-files)
* [Инструментальное программное обеспечение](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%BD%D0%BE%D0%B5_%D0%BE%D0%B1%D0%B5%D1%81%D0%BF%D0%B5%D1%87%D0%B5%D0%BD%D0%B8%D0%B5)
* [CASE](https://ru.wikipedia.org/wiki/CASE) - computer-aided software engineering
* [configure script](https://en.wikipedia.org/wiki/Configure_script)
* [Apache Ant Tutorial](https://coderlessons.com/tutorials/java-tekhnologii/izuchite-apache-ant/apache-ant-tutorial) ; 16 Мая 2019 г.
* [Автоматизация сборки](https://ru.wikipedia.org/wiki/%D0%90%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F_%D1%81%D0%B1%D0%BE%D1%80%D0%BA%D0%B8)


