---
title: ELF Relocation
description: 
published: true
date: 2023-03-16T13:24:13.606Z
tags: 
editor: markdown
dateCreated: 2023-03-16T07:41:18.626Z
---

# ELF Relocation

По умолчанию в разделяемых библиотеках включены перенаправления(relocations).
Они позволяют оптимизировать процесс поиска символов функций, чтобы не искать их заново для каждой новой функции в разделяемой библиотеке.

## Пример

main.cpp
```cpp
#include <stdio.h>
#include <lib.h>

void print()
{
	printf("call from main\n");
}

int main()
{
	libcall();
	return 0;
}
```

lib.h
```cpp
#pragma once
void libcall();
```

lib.cpp
```cpp
#include <lib.h>
#include <stdio.h>

void print()
{
	printf("call from lib\n");
}

void libcall()
{
	print();
}
```

Компиляция
```
# (GCC) 12.2.1 20230201
g++ lib.cpp -shared -I. -o libso.so
g++ main.cpp -I. -L. -lso -Wl,-rpath,\$ORIGIN -o main
```

Запуск
```
$ ./main
call from main
```

Вызвалась функция из main.

## Пояснение

Это произошло потому что по умолчанию при создании библиотеки создаётся секция relocation.

```
$ readelf --syms --relocs --use-dynamic --demangle libso.so
...
'PLT' relocation section at offset 0x570 contains 24 bytes:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000004000  000200000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
000000004008  000600000007 R_X86_64_JUMP_SLO 0000000000001119 print + 0

Symbol table for image contains 8 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
...
     6: 0000000000001119    22 FUNC    GLOBAL DEFAULT   12 print()
     7: 000000000000112f    12 FUNC    GLOBAL DEFAULT   12 libcall()
```

В данном случае функция print оказалась в секции `.rela.plt`(relocation PLT).
Вкратце это означает, что функция print будет замещена ранее найденной функцией(в данном случае функцией print из main.cpp).

## Варианты решения

### -Bsymbolic

Добавив линковщику(ld) флаг `-Bsymbolic`, символы из этой библиотеки не будут добавляться в секцию relocation. Но будут добавляться системные символы(и возможно символы из других библиотек, нужно проверять). Таким образом символы из этой библиотеки не будут замещаться сторонними.

Пример:
```
$ g++ lib.cpp -shared -I. -Wl,-Bsymbolic -o libso.so
$ readelf --syms --relocs --use-dynamic --demangle libso.so
...
'PLT' relocation section at offset 0x570 contains 24 bytes:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000004000  000200000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0

Symbol table for image contains 8 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
...
     6: 0000000000001119    22 FUNC    GLOBAL DEFAULT   12 print()
     7: 000000000000111f    12 FUNC    GLOBAL DEFAULT   12 libcall()
```

> Функции `print()` больше нет в секции relocation, но она в global секции и её можно импортировать.{.is-info}

https://sourceware.org/binutils/docs/ld/Options.html#index-_002dBsymbolic

### --dynamic-list

Можно создать список функций, которые будут подлежать замещению.

Пример:
Создадим файл версий see VERSION
В нём перечислим функции. Работают шаблоны `*` - все символы, `?` - один символ.
Для корректного деманглинга C++ нужно помещать функции в секцию extern "C++". Символы скобок являются управляющими(todo уточнить для чего они) из-за них всё имя нужно оборачивать в кавычки `""`.
lib.sym
```cpp
{
    extern "C++" {
    		example_namespace::*;
        "example_function(int)";
        "libcall()";
    };
};
```

```
$ g++ lib.cpp -shared -I. -Wl,--dynamic-list=lib.sym -o libso.so
$ readelf --syms --relocs --use-dynamic --demangle libso.so
...
'PLT' relocation section at offset 0x570 contains 24 bytes:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000004000  000200000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0

Symbol table for image contains 8 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     6: 0000000000001109    22 FUNC    GLOBAL DEFAULT   12 print()
     7: 000000000000111f    12 FUNC    GLOBAL DEFAULT   12 libcall()
```

> В данном случае функции из библиотеки не были добавлены в секцию relocation. `libcall()` добавляется в relocation в `main`.{.is-warning}

https://sourceware.org/binutils/docs/ld/Options.html#index-_002d_002ddynamic_002dlist_003ddynamic_002dlist_002dfile
https://sourceware.org/binutils/docs/ld/VERSION.html

### --version-script=lib.ver

Можно создать список функций, которые будут экпортироваться(GLOBAL). Все local функции не добавляются в секцию relocation. Но так же их нельзя использовать вне библиотеки

Пример:
Создадим файл версиий see VERSION
И определим что будет помещаться в global и local секции
lib.ver
```cpp
{
global:
    extern "C++" {
        "call()";
    };

local: *;
};
```

```
$ g++ lib.cpp -shared -I. -Wl,--version-script=lib.ver -o libso.so
$ readelf --syms --relocs --use-dynamic --demangle libso.so
...
'PLT' relocation section at offset 0x550 contains 24 bytes:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000004000  000200000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0

Symbol table for image contains 7 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
...
     6: 000000000000111f    12 FUNC    GLOBAL DEFAULT   12 libcall()
```

Доступной функцией в GLOBAL секции осталась только `libcall()`.
> Функцию `print()` не сможем использовать отдельно из main.cpp.{.is-warning}

https://sourceware.org/binutils/docs/ld/Options.html#index-version-script_002c-symbol-versions
https://sourceware.org/binutils/docs/ld/VERSION.html

### static & anonymous namespace

Помечая функцию `print()` static или помещая её в anonymous namespace мы исключаем функцию из экспортных(global) символов и так же она будет исключена из секции relocation.
> Функцию `print()` не сможем использовать отдельно из main.cpp.{.is-warning}

### -fvisibility=hidden

Этот флаг компиляции заставляет скрыть все функции из global секции, так же если бы они были помечены static или anonymous namespace, кроме тех функций которые помечены специальным атрибутом. Для gcc это `__attribute__ ((visibility ("default")))`.

> Этот флаг действует только на те функции, которые были компилированы с этим флагом. Например у вас есть статическая библиотека, компилированная без флага `-fvisibility=hidden` и вы связываете её со своей библиотекой, то символы из статической библиотеки будут в global секции.{.is-warning}

https://gcc.gnu.org/wiki/Visibility

## Links
https://habr.com/ru/post/106107/
https://www.intezer.com/blog/malware-analysis/executable-and-linkable-format-101-part-3-relocations/
https://reverseengineering.stackexchange.com/questions/1992/what-is-plt-got
https://maskray.me/blog/2021-05-16-elf-interposition-and-bsymbolic
https://fasterthanli.me/series/making-our-own-executable-packer/part-11