---
title: cpp
description: 
published: 1
date: 2025-04-16T16:16:10.766Z
tags: 
editor: markdown
dateCreated: 2025-01-09T22:05:04.537Z
---

# C++
# Приведение типов

https://habr.com/ru/post/106294/

## Tabs {.tabset}

### static_cast

`Type static_cast<Type> (object);`

`static_cast` преобразует выражения одного статического типа в объекты и значения другого статического типа. Поддерживается преобразование численных типов, указателей и ссылок по иерархии наследования как вверх, так и вниз. Проверка производится на уровне компиляции, так что в случае ошибки сообщение будет получено в момент сборки приложения или библиотеки.

### dynamic_cast

`Type& dynamic_cast<Type&> (object);`
`Type* dynamic_cast<Type*> (object);`

Используется для динамического приведения типов во время выполнения. В случае неправильного приведения типов для ссылок вызывается исключительная ситуация `std::bad_cast`, а для указателей будет возвращен 0. Использует систему RTTI (Runtime Type Information). Безопасное приведение типов по иерархии наследования, в том числе для виртуального наследования.

Основное назначение `dynamic_cast` - защита от нисходящего приведения типов в тех случаях, когда это лишено смысла. Например, если у вас есть ссылка или указатель на базовый класс, но вы точно знаете, что на самом деле она указывает на экземпляр производного типа, вы можете выполнить приведение и работать с производным типом.
Это называется нисходящее приведение. Но если там будет объект базового типа, лишенный своей производной части, такое приведение лишено смысла и приведет к неопределенному поведению.

### const_cast

`Type const_cast<Type> (object);`

Пожалуй самое простое приведение типов. Снимает cv qualifiers — const и volatile, то есть константность и отказ от оптимизации компилятором переменной. Это преобразование проверяется на уровне компиляции и в случае ошибки приведения типов будет выдано сообщение.

### reinterpret_cast

`Type reinterpret_cast<Type> (object);`

Приведение типов без проверки. `reinterpret_cast` — непосредственное указание компилятору. Применяется только в случае полной уверенности программиста в собственных действиях. Не снимает константность и `volatile`. применяется для приведения указателя к указателю, указателя к целому и наоборот.

### C-style cast

`Type (Type*) object;`

Си-шный метод приведения типов. Пожалуй самый нежелательный способ приведения типов. Страуструп пишет:
«Например, что это значит выражение — x = (T)y;. Мы не знаем. Это зависит от типа T, типов x и y. T может быть названием типа, typedef или может быть параметр template-а. Может быть, х и у являются скалярными переменными и Т представляет собой значение преобразования. Может быть, х объекта класса, производного от класса Y и Т — нисходящее преобразование. По этой причине программист может не знать, что он делает на самом деле.»
Вторая причина нежелательного использования приведения типов в C-style — трудоемкость процесса поиска мест приведения типов.


## Reinterpret_cast vs. C-style cast

https://stackoverflow.com/a/7832003

C-style cast похоже на reinterpret_cast, но оно также «пробует» сначала static_cast и может отбрасывать квалификацию cv (в то время как static_cast и reinterpret_cast не могут) и выполнять преобразования без учета контроля доступа. (see 5.4/4 in C++11 standard)

```cpp
#include <iostream>

using namespace std;

class A { int x; };
class B { int y; };

class C : A, B { int z; };

int main()
{
  C c;

  // just type pun the pointer to c, pointer value will remain the same
  // only it's type is different.
  B *b1 = reinterpret_cast<B *>(&c);

  // perform the conversion with a semantic of static_cast<B*>(&c), disregarding
  // that B is an unaccessible base of C, resulting pointer will point
  // to the B sub-object in c.
  B *b2 = (B*)(&c);

  cout << "reinterpret_cast:\t" << b1 << "\n";
  cout << "C-style cast:\t\t" << b2 << "\n";
  cout << "no cast:\t\t" << &c << "\n";
}
```

output:

```c
reinterpret_cast:  0xbfd84e78
C-style cast:      0xbfd84e7c
no cast:           0xbfd84e78
```

обратите внимание, что значение, созданное reinterpret_cast, точно такое же, как адрес 'c', в то время как C-style cast привело к правильному смещению указателя.

# Умные указатели

## Tabs {.tabset}

### std::unique_ptr

Это умный указатель, который владеет и управляет другим объектом через указатель и удаляет этот объект, когда unique_ptr выходит за пределы области видимости.

Объект удаляется с использованием асоциированного удаления (associated deleter), когда происходит одно из следующих событий:

- управляющий объект unique_ptr уничтожается;
- управляющему объекту unique_ptr назначается другой указатель с помощью operator= или reset().

https://en.cppreference.com/w/cpp/memory/unique_ptr

### std::shared_ptr

В отличие от `std::unique_ptr`, который предназначен для единоличного владения и управления переданным ему ресурсом/объектом, `std::shared_ptr` предназначен для случаев, когда несколько умных указателей совместно владеют одним динамически выделенным ресурсом.

Объект уничтожается, а его память освобождается, когда происходит одно из следующих событий:
- последний оставшийся shared_ptr, владеющий объектом, уничтожается;
- последнему оставшемуся shared_ptr, владеющему объектом, назначается другой указатель с помощью operator= или reset().

https://en.cppreference.com/w/cpp/memory/shared_ptr

### std::weak_ptr

`std::weak_ptr` — это умный указатель, который содержит ссылку, не являющуюся владельцем («слабую»), на объект, которым управляет `std::shared_ptr`. Его необходимо преобразовать в `std::shared_ptr`, чтобы получить доступ к указанному объекту.

`std::weak_ptr` моделирует временное владение: когда к объекту нужно получить доступ, только если он существует, и он может быть удален в любое время кем-то другим, `std::weak_ptr` используется для отслеживания объекта и преобразуется в `std::shared_ptr` для временного владения. Если в это время исходный `std::shared_ptr` уничтожается, время жизни объекта продлевается до тех пор, пока не будет уничтожен временный `std::shared_ptr`.

Еще одно применение `std::weak_ptr` — прерывание циклической зависимости, сформированных объектами, управляемыми `std::shared_ptr`. Если такой цикл осиротел (т. е. в цикле нет внешних общих указателей), счетчик ссылок `std::shared_ptr` не может достичь нуля, и происходит утечка памяти. Чтобы этого не произошло, один из указателей в цикле можно сделать `std::weak_ptr` («слабым»).

https://en.cppreference.com/w/cpp/memory/weak_ptr

## Пример использования

TODO

# Категории выражений
## l-values & r-values

В языке C++ все переменные являются l-values. l-value — это значение, которое имеет свой собственный адрес в памяти. Поскольку все переменные имеют адреса, то они все являются l-values. l от слова «left», так как только значения l-values могут находиться в левой стороне в операциях присваивания (в противном случае, мы получим ошибку). Например, стейтмент 9 = 10; вызовет ошибку компилятора, так как 9 не является l-value. Число 9 не имеет своего адреса в памяти и, таким образом, мы ничего не можем ему присвоить (9 = 9 и ничего здесь не изменить).

Противоположностью l-value является r-value. r-value — это значение, которое не имеет постоянного адреса в памяти. Примерами могут быть единичные числа (например, 7, которое имеет значение 7) или выражения (например, 3 + х, которое имеет значение х плюс 3).

https://pvs-studio.com/ru/blog/terms/6517/

## RVO & NRVO
https://pvs-studio.ru/ru/blog/terms/6516/

## Универсальные ссылки

https://habr.com/ru/post/157961/

TODO

# Многопоточность

https://en.cppreference.com/w/cpp/atomic/atomic
https://habr.com/ru/post/517918/

- std::atomic
- std::mutex
- std::condition_variable
- std::counting_semaphore
- std::binary_semaphore
- асинхронное программирование
    - std::promise
    - std::future
    - std::async
    - std::packaged_task
-	linux
    - rdlock rwlock
    - spinlock
    - semaphore
    - barrier

## Неблокирующая многопоточность

https://habr.com/ru/post/328348/
https://habr.com/ru/post/195948/

# Mutable

https://habr.com/ru/company/infopulse/blog/341264/

TODO

# RAII (Resource Acquisition Is Initialization)

https://habr.com/ru/post/150069/

TODO

# Шаблоны

## Частичная специализация шаблонов & SFINAE

TODO

## std::forward

https://pvs-studio.com/ru/blog/terms/6515/

TODO

# Минимальный размер класса и виртуального класса

Минимальный размер класса равен 1. По стандарту размер класса(структуры) не может быть равен 0, т.к. будет нарушаться расчёт адреса в памяти по смещению.

Минимальный размер виртуального клсса равен размеру указателя в системе(на 64x это 8). Т.к. Это размер vtable(виртуальной таблицы).

https://www.stroustrup.com/bs_faq2.html#sizeof-empty

# IPC

https://opensource.com/article/19/4/interprocess-communication-linux-networking
https://opensource.com/article/19/4/interprocess-communication-linux-storage
https://opensource.com/article/19/4/interprocess-communication-linux-channels
https://www.boost.org/doc/libs/1_74_0/doc/html/interprocess.html

# Нововведения стандартов

https://github.com/AnthonyCalandra/modern-cpp-features
https://en.cppreference.com/w/cpp/compiler_support

## C++14

- бинарные литералы;
- атрибут deprecated;
- цифровые разделители;
- автоматическое определение возвращаемого типа функции — вывод типов;
- relaxed constexpr функции;
- шаблоны переменных;
- стандартные пользовательские литералы;
- std::make_unique().

## C++17

- идентификатор препроцессора __has_include для проверки доступности дополнительных заголовочных файлов;
- if-стейтменты, которые обрабатываются во время компиляции;
- инициализаторы в стейтментах if и switch;
- встроенные переменные;
- fold-выражения;
- вложенные пространства имен теперь можно определять как пространство имен X::Y;
- удаление std::auto_ptr и других устаревших типов;
- static_assert больше не требует параметра в виде текстового сообщения;
- std::any;
- std::byte;
- std::filesystem;
- std::optional;
- std::shared_ptr теперь может управлять массивами C-style (но через std::make_shared() их по-прежнему нельзя создавать);
- std::size;
- триграфы были удалены;
- UTF-8 (u8) символьные литералы.

## C++20

- https://en.cppreference.com/w/cpp/language/default_comparisons

TODO

# links
[A guide to understanding Linux software libraries in C](https://opensource.com/article/21/2/linux-software-libraries)
[Getting started with OpenSSL: Cryptography basics](https://opensource.com/article/19/6/cryptography-basics-openssl-part-1)
[How to use OpenSSL: Hashes, digital signatures, and more](https://opensource.com/article/19/6/cryptography-basics-openssl-part-2)
[How to use Protobuf for data interchange](https://opensource.com/article/19/10/protobuf-data-interchange)
[Spinlocks and Read-Write Locks](http://locklessinc.com/articles/locks/)
[Ускоряем std::shared_mutex в 10 раз](https://habr.com/ru/post/328362/)
[https://betterprogramming.pub/c-memory-pool-and-small-object-allocator-8f27671bd9ee](https://betterprogramming.pub/c-memory-pool-and-small-object-allocator-8f27671bd9ee)
[Миграция на поседневный С++17](https://ps-group.github.io/cxx/cxx17)