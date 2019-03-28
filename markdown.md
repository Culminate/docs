# MARKDOWN
Чтобы смотреть markdown файлы есть [расширение](https://chrome.google.com/webstore/detail/markdown-preview-plus/febilkbfcbhebfnokafefeacimjdckgl)
Либо сконвертировать статический сайт mkdocs пакетом в debian
[Список генераторов статических сайтов](https://www.staticgen.com/)
## HEADERS

	Заголовок 1 уровня
	===
	Заголовок 2 уровня
	---
	# Заголовок 1 Уровня
	## Заголовок 2 Уровня
	### Заголовок 3 Уровня
	#### Заголовок 4 Уровня
	##### Заголовок 5 Уровня
	###### Заголовок 6 Уровня
	
>Заголовок 1 уровня
>=
>Заголовок 2 уровня 
>-
># Заголовок 1 Уровня
>## Заголовок 2 Уровня
>### Заголовок 3 Уровня
>#### Заголовок 4 Уровня
>##### Заголовок 5 Уровня
>###### Заголовок 6 Уровня

## LISTS
### Unordered

`*` or `-` or `+`

    * item 1             - item 1             + item 1
        * subitem 1           - subitem 1          + subitem 1
    * item 2             - item 2             + item 2
    * item 3             - item 3             + item 3

* item 1
    * subitem 1
* item 2
* item 3
    
### Ordered
    1. item 1
        1. subitem 1
    2. item 2
    3. item 3
    
1. item 1
    1. subitem 1
2. item 2
5. item 3

### Tasklists

    - [x] Задача 1
    - [ ] Задача 2
    - [ ] Задача 3
    
- [x] Задача 1
- [ ] Задача 2
- [ ] Задача 3

need markdown-cheklist extension https://github.com/FND/markdown-checklist
mkdocs.yml

    markdown_extensions:
           - markdown_checklist:

## EMPHASIS
### Italic
    *курсив*
    _также курсив_

*курсив*
_также курсив_

### Bold

    **Этот жирный текст**
    __Этот также может быть жирным__
    
**Этот жирный текст**
__Этот также может быть жирным__

### Combined
    *Вы **можете** комбинировать их*
    
*Вы **можете** комбинировать их*

    ***Жирный курсив***
    ___Это тоже может бять жирным курсивом___

***Жирный курсив***
___Это тоже может бять жирным курсивом___

### Strikethrough

    ~~Зачёркнутый~~
    
~~Зачёркнутый~~

### Highlighted

    ==Подсвеченный==
==Подсвеченный==

## Horizontal Rules

    ***
    ---
    ___
    
***

---
___

## Blockquotes

    > Цитата
    >> Цитата 2 уровня
    >>> Цитата 3 уровня
    
> Цитата
>> Цитата 2 уровня
>>> Цитата 3 уровня

## CODE
### Inline code

    `void timer_callback(timer_t *timer)`
    
`void timer_callback(timer_t *timer)`

### Block code "fences"
    ```c++ // syntax highlighting
    void timer_callback(int *timer)
    ```
```c++
void timer_callback(int *timer)
```

## Tables
    | Option | Description |
    | ------ | ----------- |
    | data   | path to data files to supply the data that will be passed into templates. |
    | engine | engine to be used for processing templates. Handlebars is the default. |
    | ext    | extension to be used for dest files. |
    
| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

### Aligned text

    | Option | Description |
    | :------:| -----------:|
    | data   | path to data files to supply the data that will be passed into templates. |
    | engine | engine to be used for processing templates. Handlebars is the default. |
    | ext    | extension to be used for dest files. |
| Option | Description |
| :------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |