# Замечания по кроссплатформенному программированию на Windows

При создании программного обеспечения (ПО)
для информационно-управляющих систем (ИУС) на платформе Microsoft Windows
следует держать в голове тот факт , что востребованность исходного кода будет во-многом
зависеть от того, можно ли его в дальнейшем использовать и на других платформах.
Тут речь идёт о понятии кроссплатформенности процесса разработки, а в качестве альтернативных
для Windows платформ чаще всего рассматривают Linux и macOS и другие
[Unix'оподобные](https://ru.wikipedia.org/wiki/Unix-%D0%BF%D0%BE%D0%B4%D0%BE%D0%B1%D0%BD%D0%B0%D1%8F_%D0%BE%D0%BF%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F_%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0).
операционные системы (ОС).
В этом плане бо́льший приоритет лежит на переносимости в пределах однотипности аппаратного обеспечения: т.е. на мобильных
платформах и встроенном ПО нужно рассматривать вопросы переносимости приложений между
[Android](https://ru.wikipedia.org/wiki/Android)
и [Apple iOS](https://ru.wikipedia.org/wiki/IOS) экосистемами.

## Среды разработки

Отчасти [среды разработки были уже рассмотрены](),
больше как редакторы, отладчики и др.

### MS Visual Studio Community

### MS Visual Studio Code

### QtCreator


## Cygwin

## MinGW

Минималистичная среда GNU для разарботки в Windows (MinGW)
например интегрирована в редактор-интегрированную среду Code::Blocks.

Есть инсталляционный исполняемый файл доступный для
[загрузки со страницы проекта](https://www.codeblocks.org/downloads/binaries/).

Есть [инструкция от активного разработчика из проекта FLTK](https://www.fltk.org/articles.php?L1820)
по установке Code::Blocks с его GUI-билиотекой.

## PTSource Developer Platform

https://ptsource.github.io/Developer-Platform/

## Подсистема для приложений на базе UNIX

[Subsystem for UNIX-based Applications](https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%B4%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0_%D0%B4%D0%BB%D1%8F_%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B9_%D0%BD%D0%B0_%D0%B1%D0%B0%D0%B7%D0%B5_UNIX)

## Windows Subsystem for Linux

[Windows Subsystem for Linux](https://ru.wikipedia.org/wiki/Windows_Subsystem_for_Linux)


## Источники

* [Кроссплатформенность](https://ru.wikipedia.org/wiki/%D0%9A%D1%80%D0%BE%D1%81%D1%81%D0%BF%D0%BB%D0%B0%D1%82%D1%84%D0%BE%D1%80%D0%BC%D0%B5%D0%BD%D0%BD%D0%BE%D1%81%D1%82%D1%8C)
* [Учебник. Создание кроссплатформенных проектов C++ в Visual Studio](https://docs.microsoft.com/ru-ru/cpp/build/get-started-linux-cmake?view=msvc-160&viewFallbackFrom=vs-2019) ; Дата: 12.08.2021 ; Чтение занимает 4 мин
* Кое-что по Javascript: [Фреймворки и библиотеки для кроссплатформенной разработки десктопных програм](https://habr.com/ru/post/528614/) ; автор: [DanFleishman](https://habr.com/ru/users/DanFleishman/) ; Дата/время: 17 ноября 2020 в 22:25
* Разработка на Free Pascal : [Multiplatform Programming Guide/ru](https://wiki.lazarus.freepascal.org/Multiplatform_Programming_Guide/ru)
* [Article #1820: Installation CodeBlocks with FLTK on Windows 2021-12-01](https://www.fltk.org/articles.php?L1820)


