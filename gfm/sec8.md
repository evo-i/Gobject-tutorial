Вверх: [Readme.md](../Readme.md),  Назад: [Раздел 7](sec7.md), Далее: [Раздел 9](sec9.md)

# Дочерний класс расширяет функцию родителя

Примером в этом разделе является объект TNumStr.
TNumStr является дочерним объектом TStr.
TNumStr содержит строку и тип строки, который может быть `t_int`, `t_double` или `t_none`.

- t\_int: строка выражает целое число
- t\_double: строка выражает число с плавающей точкой (double)
- t\_none: строка не выражает ни одно из двух чисел выше.

Строка типа t\_int или t\_double называется числовой строкой.
Это не общепринятая терминология, она используется только в этом учебнике.
Короче говоря, числовая строка - это строка, которая выражает число.
Например, "0", "-100" и "123.456" являются числовыми строками.
Это строки, которые выражают числа.

- "0" является строкой.
Это массив символов, элементы которого '0' и '\0'.
Она выражает 0, что является целым нулем.
- "-100" является строкой, состоящей из '-', '1', '0', '0' и '\0'.
Она выражает целое число -100.
- "123.456" состоит из '1', '2', '3', '.', '4', '5', '6' и '\0'.
Она выражает вещественное число (тип double) 123.456.

Числовая строка - это именно такая специфическая строка.

## Проверка числовой строки

Перед определением TNumStr нам нужен способ проверки числовой строки.

Числовая строка включает:

- Целое число.
Например, "0", "100", "-10" и "+20".
- Число с плавающей точкой.
Например, "0.1", "-10.3", "+3.14", ".05" "1." и "0.0".

Нужно быть осторожным, что "0" и "0.0" различны.
Потому что их типы различны.
Тип "0" - целое число, а тип "0.0" - double.
Точно так же "1" - это целое число, а "1." - это double.

".5" и "0.5" - одно и то же.
Оба являются double, и их значения равны 0.5.

Проверка числовой строки является своего рода лексическим анализом.
Для лексического анализа часто используется диаграмма состояний и матрица состояний.

Числовая строка - это последовательность символов, которая удовлетворяет следующим условиям:

1. '+' или '-' может быть первым символом. Он может быть опущен.
2. за ним следует последовательность цифр.
3. за ней следует точка.
4. за ней следует последовательность цифр.

Вторая часть может быть опущена.
Например, ".56" или "-.56" являются корректными.

Третья и четвертая части могут быть опущены.
Например, "12" или "-23" являются корректными.

Четвертая часть может быть опущена.
Например, "100." является корректной.

Существует шесть состояний.

- 0 - начальная точка.
- 1 - состояние после '+' или '-'.
- 2 - в середине первой последовательности цифр (целая часть).
- 3 - состояние после десятичной точки.
- 4 - конец строки, и строка является int.
- 5 - конец строки, и строка является double.
- 6 - ошибка. Строка не выражает число.

Входные символы:

- 0: '+' или '-'
- 1: цифра ('0' - '9')
- 2: точка '.'
- 3: конец строки '\0'
- 4: другие символы

Диаграмма состояний выглядит следующим образом.

![state diagram of a numeric string](../image/state_diagram.png)

Матрица состояний:

|state＼input|0 |1 |2 |3 |4 |
|:-----------|:-|:-|:-|:-|:-|
|0           |1 |2 |3 |6 |6 |
|1           |6 |2 |3 |6 |6 |
|2           |6 |2 |3 |4 |6 |
|3           |6 |3 |6 |5 |6 |

Эта диаграмма состояний не поддерживает числа с плавающей точкой в стиле "1.23e5" (десятичная запись с экспонентой).
Если бы это поддерживалось, диаграмма состояний была бы более сложной.
(Однако это была бы хорошая практика для ваших навыков программирования.)

## Заголовочный файл

Заголовочный файл TNumStr - это [`tnumstr.h`](../src/tstr/tnumstr.h).
Он находится в каталоге `src/tstr`.

~~~C
 1 #pragma once
 2 
 3 #include <glib-object.h>
 4 #include "tstr.h"
 5 #include "../tnumber/tnumber.h"
 6 
 7 #define T_TYPE_NUM_STR  (t_num_str_get_type ())
 8 G_DECLARE_FINAL_TYPE (TNumStr, t_num_str, T, NUM_STR, TStr)
 9 
10 /* type of the numeric string */
11 enum {
12   t_none,
13   t_int,
14   t_double
15 };
16 
17 /* get the type of the TNumStr object */
18 int
19 t_num_str_get_string_type (TNumStr *self);
20 
21 /* setter and getter */
22 void
23 t_num_str_set_from_t_number (TNumStr *self, TNumber *num);
24 
25 // TNumStr can have any string, which is t_none, t_int or t_double type.
26 // If the type is t_none, t_num_str_get_t_number returns NULL.
27 // It is good idea to call t_num_str_get_string_type and check the type in advance. 
28 
29 TNumber *
30 t_num_str_get_t_number (TNumStr *self);
31 
32 /* create a new TNumStr instance */
33 TNumStr *
34 t_num_str_new_with_tnumber (TNumber *num);
35 
36 TNumStr *
37 t_num_str_new (void);
~~~

- 8: Макрос `G_DECLARE_FINAL_TYPE` для класса TNumStr.
Это дочерний класс TStr и финальный тип класса.
- 11-15: Эти три enum-данные определяют тип строки TNumStr.
  - `t_none`: Строка не сохранена или строка не является числовой строкой.
  - `t_int`: Строка выражает целое число
  - `t_double`: Строка выражает вещественное число, которое является типом double в языке C.
- 18-19: Публичная функция `t_num_str_get_string_type` возвращает тип строки, которую имеет объект TStrNum.
Возвращаемое значение - `t_none`, `t_int` или `t_double`.
- 22-30: Сеттер и геттер из/в объект TNumber.
- 33-37: Функции для создания новых объектов TNumStr.

## C файл

C файл TNumStr - это [`tnumstr.c`](../src/tstr/tnumstr.c).
Он находится в каталоге `src/tstr`.

~~~C
  1 #include <stdlib.h>
  2 #include <ctype.h>
  3 #include "tnumstr.h"
  4 #include "tstr.h"
  5 #include "../tnumber/tnumber.h"
  6 #include "../tnumber/tint.h"
  7 #include "../tnumber/tdouble.h"
  8 
  9 struct _TNumStr {
 10   TStr parent;
 11   int type;
 12 };
 13 
 14 G_DEFINE_TYPE(TNumStr, t_num_str, T_TYPE_STR)
 15 
 16 static int
 17 t_num_str_string_type (const char *string) {
 18   const char *t;
 19   int stat, input;
 20   /* state matrix */
 21   int m[4][5] = {
 22     {1, 2, 3, 6, 6},
 23     {6, 2, 3, 6, 6},
 24     {6, 2, 3, 4, 6},
 25     {6, 3, 6, 5, 6}
 26   };
 27 
 28   if (string == NULL)
 29     return t_none;
 30   stat = 0;
 31   for (t = string; ; ++t) {
 32     if (*t == '+' || *t == '-')
 33       input = 0;
 34     else if (isdigit (*t))
 35       input = 1;
 36     else if (*t == '.')
 37       input = 2;
 38     else if (*t == '\0')
 39       input = 3;
 40     else
 41       input = 4;
 42 
 43     stat = m[stat][input];
 44 
 45     if (stat >= 4 || *t == '\0')
 46       break;
 47   }
 48   if (stat == 4)
 49     return t_int;
 50   else if (stat == 5)
 51     return t_double;
 52   else
 53     return t_none;
 54 }
 55 
 56 /* This function overrides t_str_set_string. */
 57 /* And it also changes the behavior of setting the "string" property. */
 58 /* On TStr => just set the "string" property */
 59 /* On TNumStr => set the "string" property and set the type of the string. */
 60 static void
 61 t_num_str_real_set_string (TStr *self, const char *s) {
 62   T_STR_CLASS (t_num_str_parent_class)->set_string (self, s);
 63   T_NUM_STR (self)->type = t_num_str_string_type(s);
 64 }
 65 
 66 static void
 67 t_num_str_init (TNumStr *self) {
 68   self->type = t_none;
 69 }
 70 
 71 static void
 72 t_num_str_class_init (TNumStrClass *class) {
 73   TStrClass *t_str_class = T_STR_CLASS (class);
 74 
 75   t_str_class->set_string = t_num_str_real_set_string;
 76 }
 77 
 78 int
 79 t_num_str_get_string_type (TNumStr *self) {
 80   g_return_val_if_fail (T_IS_NUM_STR (self), -1);
 81 
 82   return self->type;
 83 }
 84 
 85 /* setter and getter */
 86 void
 87 t_num_str_set_from_t_number (TNumStr *self, TNumber *num) {
 88   g_return_if_fail (T_IS_NUM_STR (self));
 89   g_return_if_fail (T_IS_NUMBER (num));
 90 
 91   char *s;
 92 
 93   s = t_number_to_s (T_NUMBER (num));
 94   t_str_set_string (T_STR (self), s);
 95   g_free (s);
 96 }
 97 
 98 TNumber *
 99 t_num_str_get_t_number (TNumStr *self) {
100   g_return_val_if_fail (T_IS_NUM_STR (self), NULL);
101 
102   char *s = t_str_get_string(T_STR (self));
103   TNumber *tnum;
104 
105   if (self->type == t_int)
106     tnum = T_NUMBER (t_int_new_with_value (atoi (s)));
107   else if (self->type == t_double)
108     tnum = T_NUMBER (t_double_new_with_value (atof (s)));
109   else
110     tnum = NULL;
111   g_free (s);
112   return tnum;
113 }
114 
115 /* create a new TNumStr instance */
116 
117 TNumStr *
118 t_num_str_new_with_tnumber (TNumber *num) {
119   g_return_val_if_fail (T_IS_NUMBER (num), NULL);
120 
121   TNumStr *numstr;
122 
123   numstr = t_num_str_new ();
124   t_num_str_set_from_t_number (numstr, num);
125   return numstr;
126 }
127 
128 TNumStr *
129 t_num_str_new (void) {
130   return T_NUM_STR (g_object_new (T_TYPE_NUM_STR, NULL));
131 }
~~~

- 9-12: Структура TNumStr имеет родительский член "TStr" и член типа int "type".
Таким образом, экземпляр TNumStr содержит строку, которая размещена в приватной области родителя, и тип.
- 14: Макрос `G_DEFINE_TYPE`.
- 16- 54: Функция `t_num_str_string_type` проверяет заданную строку и возвращает `t_int`, `t_double` или `t_none`.
Если строка равна NULL или является нечисловой строкой, будет возвращено `t_none`.
Алгоритм проверки объясняется в первом подразделе "Проверка числовой строки".
- 60-64: Функция `t_num_str_real_set_string` устанавливает строку TNumStr и ее тип.
Это тело метода класса, на который указывает член `set_string` структуры класса.
Метод класса инициализируется в функции инициализации класса `t_num_str_class_init`.
- 66-69: Функция инициализации экземпляра `t_num_str_init` устанавливает тип в `t_none`,
потому что функция инициализации родителя установила указатель `priv->string` в NULL.
- 71-76: Функция инициализации класса `t_num_str_class_init` присваивает `t_num_str_real_set_string` члену `set_string`.
Поэтому функция `t_str_set_string` вызывает `t_num_str_real_set_string`, которая устанавливает не только строку, но и тип.
Функция `g_object_set` также вызывает её и устанавливает как строку, так и тип.
- 78-83: Публичная функция `t_num_str_get_string_type` возвращает тип строки.
- 86-113: Сеттер и геттер.
Сеттер устанавливает числовую строку из объекта TNumber.
А геттер возвращает объект TNumber.
- 117-131: Эти две функции создают экземпляры TNumStr.

## Дочерний класс расширяет функцию родителя.

TNumStr является дочерним классом TStr, поэтому он имеет все публичные функции TStr.

- `TStr *t_str_concat (TStr *self, TStr *other)`
- `void t_str_set_string (TStr *self, const char *s)`
- `char *t_str_get_string (TStr *self)`

Когда вы хотите установить строку для экземпляра TNumStr, вы можете использовать функцию `t_str_set_string`.

```c
TNumStr *ns = t_num_str_new ();
t_str_set_string (T_STR (ns), "123.456");
```

TNumStr расширяет функцию `t_str_set_string` и устанавливает не только строку, но и тип для экземпляра TNumStr.

```c
int t;
t = t_num_str_get_string_type (ns);
if (t == t_none) g_print ("t_none\n");
if (t == t_int) g_print ("t_int\n");
if (t == t_double) g_print ("t_double\n");
// => t_double appears on your display
```

TNumStr добавляет некоторые публичные функции.

- `int t_num_str_get_string_type (TNumStr *self)`
- `void t_num_str_set_from_t_number (TNumStr *self, TNumber *num)`
- `TNumber *t_num_str_get_t_number (TNumStr *self)`

Дочерний класс расширяет родительский класс.
Таким образом, дочерний класс более специфичен и богаче, чем родительский.

## Компиляция и выполнение

В каталоге `src/tstr` находятся `main.c`, `test1.c` и `test2.c`.
Две программы `test1.c` и `test2.c` генерируют `_build/test1` и `_build/test2` соответственно.
Они тестируют `tstr.c` и `tnumstr.c`.
Если есть ошибки, появятся сообщения.
В противном случае ничего не появится.

Программа `main.c` генерирует `_build/tnumstr`.
Она показывает, как работают TStr и TNumStr.

Компиляция выполняется обычным способом.
Сначала измените текущий каталог на `src/tstr`.

~~~
$ cd src/tstr
$ meson setup _build
$ ninja -C _build
$ _build/test1
$ _build/test2
$ _build/tnumstr
String property is set to one.
"one" and "two" is "onetwo".
123 + 456 + 789 = 1368
TNumStr => TNumber => TNumStr
123 => 123 => 123
-45 => -45 => -45
+0 => 0 => 0
123.456 => 123.456000 => 123.456000
+123.456 => 123.456000 => 123.456000
-123.456 => -123.456000 => -123.456000
.456 => 0.456000 => 0.456000
123. => 123.000000 => 123.000000
0.0 => 0.000000 => 0.000000
123.4567890123456789 => 123.456789 => 123.456789
abc => (null) => abc
(null) => (null) => (null)
~~~

Последняя часть `main.c` представляет собой преобразование между TNumStr и TNumber.
Между ними есть некоторые различия по следующим двум причинам.

- числа с плавающей точкой округляются.
Это не точное значение, когда значение имеет длинные цифры.
TNumStr "123.4567890123456789" преобразуется в TNumber 123.456789.
- Существует два или более строковых представления для чисел с плавающей точкой.
TNumStr ".456" преобразуется в TNumber x, а x преобразуется в TNumStr "0.456000".

Трудно сравнивать два экземпляра TNumStr.
Лучший способ - сравнивать экземпляры TNumber, преобразованные из TNumStr.

Вверх: [Readme.md](../Readme.md),  Назад: [Раздел 7](sec7.md), Далее: [Раздел 9](sec9.md)
