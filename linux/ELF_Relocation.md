---
title: ELF Relocation
description: 
published: true
date: 2023-03-16T07:59:04.648Z
tags: 
editor: markdown
dateCreated: 2023-03-16T07:41:18.626Z
---

# ELF Relocation

По умолчанию в разделяемых библиотеках включены перенаправления(relocations).
Они позволяют оптимизировать процесс поиска символов функций, чтобы не искать их заново для каждой новой функции в разделяемой библиотеке.

## Пример

main.cpp
```
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
```
#pragma once
void libcall();
```

lib.cpp
```
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
```shell
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

```bash
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

### static & anonimous namespace

Помечая функцию `print()` static или помещая её в anonimous namespace мы исключаем функцию из экспортных(global) символов и так же она будет исключена из секции relocation.
Но тогда мы её не сможем использовать отдельно из main.cpp

### -Bsymbolic

Добавив линковщику(ld) флаг `-Bsymbolic`, символы из этой библиотеки не будут добавляться в секцию relocation. Но будут добавляться системные символы(и возможно символы из других библиотек, нужно проверять). Таким образом символы из этой библиотеки не будут замещаться сторонними.

Пример:
```bash
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

Функции print больше нет в секции relocation, но она в global секции и её можно импортировать.

https://sourceware.org/binutils/docs/ld/Options.html#index-_002dBsymbolic

### --dynamic-list

Можно создать список функций, которые будут подлежать замещению.

Пример:
lib.sym
```
{
    extern "C++" {
        "call()";
    };
};
```

```bash
g++ lib.cpp -shared -I. -Wl,--dynamic-list=lib.sym -o libso.so
readelf --syms --relocs --use-dynamic --demangle libso.so
```

https://sourceware.org/binutils/docs/ld/Options.html#index-_002d_002ddynamic_002dlist_003ddynamic_002dlist_002dfile
https://sourceware.org/binutils/docs/ld/VERSION.html

## Links
https://habr.com/ru/post/106107/
https://www.intezer.com/blog/malware-analysis/executable-and-linkable-format-101-part-3-relocations/
https://reverseengineering.stackexchange.com/questions/1992/what-is-plt-got
https://maskray.me/blog/2021-05-16-elf-interposition-and-bsymbolic