# Инструменты разработки кода

## Автосборка из командной строки

Прежде всего средства автосборки из командной строки
запускают препроцессоры, компиляторы, компоновщики, ассемблеры
с огромным количеством опций и параметров.
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

Комментарии в файлах CMakeList.txt начинаются символом решётки (`#`).



### GNU Autotools


### Apache Ant

### ninja-build


### scons


## Интерактивная разработка:

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



