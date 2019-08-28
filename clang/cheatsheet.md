# Функции

<тип> <название>(<тип аргумента> <имя аргумента>);
```c
void matx(int x);

```

Объявление аргументов без указания имен

```c
void matx(int, char);

void matx(int x, char y)
{
    return x - y;
}
```

Пост объявление аргументов

```c
void matx(x, y)
int x;
char y;
{
    return x - y;
}
```

# Структуры

Объявление структуры
```c
struct name_struct {
    char *key;          // field
    unsigned long dflt; // field
};
```

Создание структуры, но создавать еще структуру по её имени нельзя `struct name_struct struct2` не сработает
```c
struct {
    char *key;          // field
    unsigned long dflt; // field
} name_struct;

name_struct.dflt = 5;
```

Создание и объявление структуры, можно иcпользовать одинаковые названия
```c
struct name_struct {
    char *key;          // field
    unsigned long dflt; // field
} name_struct;

name_struct.dflt = 5;
struct name_struct struct2
struct2.key = "hello";
```

Объявление структуры как типа. Нельзя сразу создать структуру
```c
typedef struct {
    char *key;          // field
    unsigned long dflt; // field
} name_struct;

name_struct struct2;
struct2.dflt = 6;
```