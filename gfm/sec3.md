Up: [Readme.md](../Readme.md),  Prev: [Section 2](sec2.md), Next: [Section 4](sec4.md)

# Система типов и процесс регистрации

GObject является базовым объектом.
Обычно мы не используем GObject сам по себе.
Потому что GObject очень простой и недостаточен для самостоятельного использования в большинстве ситуаций.
Вместо этого мы используем производные объекты GObject, такие как множество видов GtkWidget.
Можно даже сказать, что такая возможность наследования является наиболее важной особенностью GObject.

Этот раздел описывает, как определить дочерний объект GObject.

## Соглашение об именовании

Примером для этого раздела является объект, представляющий вещественное число.
Он не очень полезен, потому что у нас уже есть тип double в языке C для представления вещественных чисел.
Однако я думаю, что этот пример неплох для изучения техники определения дочернего объекта.

Во-первых, вам нужно знать соглашение об именовании.
Имя объекта состоит из пространства имен и имени.
Например, "GObject" состоит из пространства имен "G" и имени "Object".
"GtkWidget" состоит из пространства имен "Gtk" и имени "Widget".
Пусть пространство имен будет "T", а имя будет "Double" для нового объекта.
В этом руководстве мы используем "T" в качестве пространства имен для всех создаваемых объектов.

TDouble — это имя объекта.
Это дочерний объект GObject.
Он представляет вещественное число, и тип числа — double.
У него есть несколько полезных функций.

## Определение TDoubleClass и TDouble

Когда мы говорим "тип", это может быть тип в системе типов или тип языка C.
Например, GObject — это имя типа в системе типов.
А char, int или double — это типы языка C.
Когда значение слова "тип" ясно из контекста, мы просто называем его "тип".
Но если это неоднозначно, мы называем его "тип C" или "тип в системе типов".

Объект TDouble имеет класс и экземпляр.
Тип C класса — TDoubleClass.
Его структура выглядит так:

~~~C
typedef struct _TDoubleClass TDoubleClass;
struct _TDoubleClass {
  GObjectClass parent_class;
};
~~~

\_TDoubleClass — это имя тега структуры C, а TDoubleClass — это "struct \_TDoubleClass".
TDoubleClass — это вновь созданный тип C.

- Используйте typedef для определения типа класса.
- Первый член структуры должен быть структурой класса родителя.

TDoubleClass не нуждается в собственных членах.

Тип C экземпляра TDouble — это TDouble.

~~~C
typedef struct _TDouble TDouble;
struct _TDouble {
  GObject parent;
  double value;
};
~~~

Это похоже на структуру класса.

- Используйте typedef для определения типа экземпляра.
- Первый член структуры должен быть структурой экземпляра родителя.

TDouble имеет свой собственный член "value".
Это значение экземпляра TDouble.

Соглашение о написании кода, приведенное выше, должно соблюдаться всегда.

## Процесс создания дочернего объекта GObject

Процесс создания типа TDouble аналогичен процессу создания GObject.

1. Регистрирует тип TDouble в системе типов.
2. Система типов выделяет память для TDoubleClass и TDouble.
3. Инициализирует TDoubleClass.
4. Инициализирует TDouble.

## Регистрация

Обычно регистрация выполняется удобными макросами, такими как `G_DECLARE_FINAL_TYPE` и `G_DEFINE_TYPE`.
Вы можете использовать `G_DEFINE_FINAL_TYPE` для финального класса типа вместо `G_DEFINE_TYPE` начиная с версии GLib 2.70.
Таким образом, вам не нужно заботиться о деталях регистрации.
Но в этом руководстве важно понимать систему типов GObject, поэтому я хочу сначала показать вам регистрацию без макроса.

Существует два типа типов: статический и динамический.
Статический тип не уничтожает свой класс даже после того, как все экземпляры были уничтожены.
Динамический тип уничтожает свой класс, когда последний экземпляр был уничтожен.
Тип GObject является статическим, и тип его производных объектов также является статическим.
Функция `g_type_register_static` регистрирует тип статического объекта.
Следующий код извлечен из `gtype.h` в исходных файлах Glib.

~~~C
GType
g_type_register_static (GType           parent_type,
                        const gchar     *type_name,
                        const GTypeInfo *info,
                        GTypeFlags      flags);
~~~

Параметры выше:

- parent_type: Родительский тип.
- type_name: Имя типа.
Например, "TDouble".
- info: Информация о типе.
Структура `GTypeInfo` будет объяснена ниже.
- flags: Флаг.
Если тип является абстрактным типом или абстрактным типом значения, то установите их флаг.
В противном случае установите его в ноль.

Поскольку система типов поддерживает отношение родитель-потомок типа, `g_type_register_static` имеет параметр родительского типа.
И система типов также хранит информацию о типе.
После регистрации `g_type_register_static` возвращает тип нового объекта.

Структура `GTypeInfo` определена следующим образом.

~~~C
typedef struct _GTypeInfo  GTypeInfo;

struct _GTypeInfo
{
  /* interface types, classed types, instantiated types */
  guint16                class_size;

  GBaseInitFunc          base_init;
  GBaseFinalizeFunc      base_finalize;

  /* interface types, classed types, instantiated types */
  GClassInitFunc         class_init;
  GClassFinalizeFunc     class_finalize;
  gconstpointer          class_data;

  /* instantiated types */
  guint16                instance_size;
  guint16                n_preallocs;
  GInstanceInitFunc      instance_init;

  /* value handling */
  const GTypeValueTable  *value_table;
};
~~~

Эта структура должна быть создана до регистрации.

- class_size: Размер класса.
Например, размер класса TDouble — `sizeof (TDoubleClass)`.
- base_init, base_finalize: Эти функции инициализируют/завершают динамические члены класса.
Во многих случаях они не нужны и им присваивается NULL.
Для получения дополнительной информации см. [GObject API Reference -- BaseInitFunc](https://docs.gtk.org/gobject/callback.BaseInitFunc.html)
и [GObject API Reference -- ClassInitFunc](https://docs.gtk.org/gobject/callback.ClassInitFunc.html).
- class_init: Инициализирует статические члены класса.
Назначьте вашу функцию инициализации класса члену `class_init`.
По соглашению, имя — `<пространство имен>_<имя>_class_init`, например, `t_double_class_init`.
- class_finalize: Завершает класс.
Поскольку производный тип GObject является статическим, он не имеет функции завершения.
Назначьте NULL члену `class_finalize`.
- class_data: Пользовательские данные, передаваемые функциям инициализации/завершения класса.
Обычно присваивается NULL.
- instance_size: Размер экземпляра.
Например, размер экземпляра TDouble — `sizeof (TDouble)`.
- n_preallocs: Это игнорируется. Использовалось старой версией Glib.
- instance_init: Инициализирует члены экземпляра.
Назначьте вашу функцию инициализации экземпляра члену `instance_init`.
По соглашению, имя — `<пространство имен>_<имя>_init`, например, `t_double_init`.
- value_table: Обычно это полезно только для фундаментальных типов.
Если тип является производным от GObject, назначьте NULL.

Эта информация хранится в системе типов и используется при создании или уничтожении объекта.
Class\_size и instance\_size используются для выделения памяти для класса и экземпляра.
Функции Class\_init и instance\_init вызываются при инициализации класса или экземпляра.

Программа на C `example3.c` показывает, как использовать `g_type_register_static`.

~~~C
 1 #include <glib-object.h>
 2 
 3 #define T_TYPE_DOUBLE  (t_double_get_type ())
 4 
 5 typedef struct _TDouble TDouble;
 6 struct _TDouble {
 7   GObject parent;
 8   double value;
 9 };
10 
11 typedef struct _TDoubleClass TDoubleClass;
12 struct _TDoubleClass {
13   GObjectClass parent_class;
14 };
15 
16 static void
17 t_double_class_init (TDoubleClass *class) {
18 }
19 
20 static void
21 t_double_init (TDouble *self) {
22 }
23 
24 GType
25 t_double_get_type (void) {
26   static GType type = 0;
27   GTypeInfo info;
28 
29   if (type == 0) {
30     info.class_size = sizeof (TDoubleClass);
31     info.base_init = NULL;
32     info.base_finalize = NULL;
33     info.class_init = (GClassInitFunc)  t_double_class_init;
34     info.class_finalize = NULL;
35     info.class_data = NULL;
36     info.instance_size = sizeof (TDouble);
37     info.n_preallocs = 0;
38     info.instance_init = (GInstanceInitFunc)  t_double_init;
39     info.value_table = NULL;
40     type = g_type_register_static (G_TYPE_OBJECT, "TDouble", &info, 0);
41   }
42   return type;
43 }
44 
45 int
46 main (int argc, char **argv) {
47   GType dtype;
48   TDouble *d;
49 
50   dtype = t_double_get_type (); /* or dtype = T_TYPE_DOUBLE */
51   if (dtype)
52     g_print ("Registration was a success. The type is %lx.\n", dtype);
53   else
54     g_print ("Registration failed.\n");
55 
56   d = g_object_new (T_TYPE_DOUBLE, NULL);
57   if (d)
58     g_print ("Instantiation was a success. The instance address is %p.\n", d);
59   else
60     g_print ("Instantiation failed.\n");
61   g_object_unref (d); /* Releases the object d. */
62 
63   return 0;
64 }
65 
~~~

- 16-22: Функция инициализации класса и функция инициализации экземпляра.
Аргумент `class` указывает на структуру класса, а аргумент `self` указывает на структуру экземпляра.
Здесь они ничего не делают, но необходимы для регистрации.
- 24-43: Функция `t_double_get_type`.
Эта функция возвращает тип объекта TDouble.
Имя функции всегда `<пространство имен>_<имя>_get_type`.
И макрос `<ПРОСТРАНСТВО_ИМЕН>_TYPE_<ИМЯ>` (все символы в верхнем регистре) заменяется этой функцией.
Посмотрите на строку 3.
`T_TYPE_DOUBLE` — это макрос, заменяемый на `t_double_get_type ()`.
Эта функция имеет статическую переменную `type` для хранения типа объекта.
При первом вызове этой функции `type` равен нулю.
Затем она вызывает `g_type_register_static` для регистрации объекта в системе типов.
При втором или последующих вызовах функция просто возвращает `type`, потому что статической переменной `type` было присвоено ненулевое значение функцией `g_type_register_static`, и она сохраняет это значение.
- 30-40: Устанавливает структуру `info` и вызывает `g_type_register_static`.
- 45-64: Главная функция.
Получает тип объекта TDouble и отображает его.
Функция `g_object_new` используется для создания экземпляра объекта.
Справочник GObject API говорит, что функция возвращает указатель на экземпляр GObject, но на самом деле она возвращает gpointer.
Gpointer — это то же самое, что `void *`, и может быть присвоен указателю, который указывает на любой тип.
Таким образом, утверждение `d = g_object_new (T_TYPE_DOUBLE, NULL);` корректно.
Если бы функция `g_object_new` возвращала `GObject *`, необходимо было бы приводить возвращаемый указатель.
После создания она показывает адрес экземпляра.
Наконец, экземпляр освобождается и уничтожается функцией `g_object_unref`.

`example3.c` находится в каталоге [src/misc](../src/misc).

Выполните его.

~~~
$ cd src/misc; _build/example3
Registration was a success. The type is 56414f164880.
Instantiation was a success. The instance address is 0x56414f167010.
~~~

## Макрос G_DEFINE_TYPE

Регистрация, описанная выше, всегда выполняется с одним и тем же алгоритмом.
Поэтому ее можно определить как макрос, такой как `G_DEFINE_TYPE`.

`G_DEFINE_TYPE` делает следующее:

- Объявляет функцию инициализации класса.
Ее имя — `<пространство имен>_<имя>_class_init`.
Например, если имя объекта `TDouble`, то это `t_double_class_init`.
Это объявление, а не определение.
Вам нужно определить ее.
- Объявляет функцию инициализации экземпляра.
Ее имя — `<пространство имен>_<имя>_init`.
Например, если имя объекта `TDouble`, то это `t_double_init`.
Это объявление, а не определение.
Вам нужно определить ее.
- Определяет статическую переменную, указывающую на родительский класс.
Ее имя — `<пространство имен>_<имя>_parent_class`.
Например, если имя объекта `TDouble`, то это `t_double_parent_class`.
- Определяет функцию `<пространство имен>_<имя>_get_type ()`.
Например, если имя объекта `TDouble`, то это `t_double_get_type`.
Регистрация выполняется в этой функции, как в предыдущем подразделе.

Использование этого макроса сокращает строки программы.
См. следующий пример `example4.c`, который работает так же, как `example3.c`.

~~~C
 1 #include <glib-object.h>
 2 
 3 #define T_TYPE_DOUBLE  (t_double_get_type ())
 4 
 5 typedef struct _TDouble TDouble;
 6 struct _TDouble {
 7   GObject parent;
 8   double value;
 9 };
10 
11 typedef struct _TDoubleClass TDoubleClass;
12 struct _TDoubleClass {
13   GObjectClass parent_class;
14 };
15 
16 G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)
17 
18 static void
19 t_double_class_init (TDoubleClass *class) {
20 }
21 
22 static void
23 t_double_init (TDouble *self) {
24 }
25 
26 int
27 main (int argc, char **argv) {
28   GType dtype;
29   TDouble *d;
30 
31   dtype = t_double_get_type (); /* or dtype = T_TYPE_DOUBLE */
32   if (dtype)
33     g_print ("Registration was a success. The type is %lx.\n", dtype);
34   else
35     g_print ("Registration failed.\n");
36 
37   d = g_object_new (T_TYPE_DOUBLE, NULL);
38   if (d)
39     g_print ("Instantiation was a success. The instance address is %p.\n", d);
40   else
41     g_print ("Instantiation failed.\n");
42   g_object_unref (d);
43 
44   return 0;
45 }
46 
~~~

Благодаря `G_DEFINE_TYPE` мы освобождены от написания утомительного кода, такого как `GTypeInfo` и `g_type_register_static`.
Одна важная вещь, на которую нужно обратить внимание, — следовать соглашению об именовании функций инициализации.

Выполните его.

~~~
$ cd src/misc; _build/example4
Registration was a success. The type is 564b4ff708a0.
Instantiation was a success. The instance address is 0x564b4ff71400.
~~~

Вы можете использовать `G_DEFINE_FINAL_TYPE` вместо `G_DEFINE_TYPE` для финальных классов типа начиная с версии GLib 2.70.

## Макрос G_DECLARE_FINAL_TYPE

Другой полезный макрос — это макрос `G_DECLARE_FINAL_TYPE`.
Этот макрос может использоваться для финального типа.
Финальный тип не имеет дочерних элементов.
Если тип имеет дочерние элементы, это производный тип.
Если вы хотите определить объект производного типа, используйте вместо этого `G_DECLARE_DERIVABLE_TYPE`.
Однако в большинстве случаев вы, вероятно, захотите написать объекты финального типа.

`G_DECLARE_FINAL_TYPE` делает следующее:

- Объявляет функцию `<пространство имен>_<имя>_get_type ()`.
Это только объявление.
Вам нужно определить ее.
Но вы можете использовать `G_DEFINE_TYPE`, его раскрытие включает определение функции.
Таким образом, вам фактически не нужно писать определение самостоятельно.
- Тип C объекта определяется как typedef структуры.
Например, если имя объекта `TDouble`, то `typedef struct _TDouble TDouble` включается в раскрытие.
Но вам нужно самостоятельно определить структуру `struct _TDouble` перед `G_DEFINE_TYPE`.
- Определяется макрос `<ПРОСТРАНСТВО_ИМЕН>_<ИМЯ>`.
Например, если объект — `TDouble`, макрос — `T_DOUBLE`.
Он будет раскрыт в функцию, которая приводит аргумент к указателю на объект.
Например, `T_DOUBLE (obj)` приводит тип `obj` к `TDouble *`.
- Определяется макрос `<ПРОСТРАНСТВО_ИМЕН>_IS_<ИМЯ>`.
Например, если объект — `TDouble`, макрос — `T_IS_DOUBLE`.
Он будет раскрыт в функцию, которая проверяет, указывает ли аргумент на экземпляр `TDouble`.
Он возвращает true, если аргумент указывает на потомка `TDouble`.
- Определяется структура класса.
Объект финального типа не нуждается в собственных членах структуры класса.
Определение похоже на строки 11-14 в `example4.c`.

Вам нужно написать определение макроса типа объекта перед `G_DECLARE_FINAL_TYPE`.
Например, если объект — `TDouble`, то

~~~C
#define T_TYPE_DOUBLE  (t_double_get_type ())
~~~

должно быть определено перед `G_DECLARE_FINAL_TYPE`.

Файл C `example5.c` использует этот макрос.
Он работает так же, как `example3.c` или `example4.c`.

~~~C
 1 #include <glib-object.h>
 2 
 3 #define T_TYPE_DOUBLE  (t_double_get_type ())
 4 G_DECLARE_FINAL_TYPE (TDouble, t_double, T, DOUBLE, GObject)
 5 
 6 struct _TDouble {
 7   GObject parent;
 8   double value;
 9 };
10 
11 G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)
12 
13 static void
14 t_double_class_init (TDoubleClass *class) {
15 }
16 
17 static void
18 t_double_init (TDouble *self) {
19 }
20 
21 int
22 main (int argc, char **argv) {
23   GType dtype;
24   TDouble *d;
25 
26   dtype = t_double_get_type (); /* or dtype = T_TYPE_DOUBLE */
27   if (dtype)
28     g_print ("Registration was a success. The type is %lx.\n", dtype);
29   else
30     g_print ("Registration failed.\n");
31 
32   d = g_object_new (T_TYPE_DOUBLE, NULL);
33   if (d)
34     g_print ("Instantiation was a success. The instance address is %p.\n", d);
35   else
36     g_print ("Instantiation failed.\n");
37 
38   if (T_IS_DOUBLE (d))
39     g_print ("d is TDouble instance.\n");
40   else
41     g_print ("d is not TDouble instance.\n");
42 
43   if (G_IS_OBJECT (d))
44     g_print ("d is GObject instance.\n");
45   else
46     g_print ("d is not GObject instance.\n");
47   g_object_unref (d);
48 
49   return 0;
50 }
51 
~~~

Выполните его.

~~~
$ cd src/misc; _build/example5
Registration was a success. The type is 5560b4cf58a0.
Instantiation was a success. The instance address is 0x5560b4cf6400.
d is TDouble instance.
d is GObject instance.
~~~

## Разделение файла на main.c, tdouble.h и tdouble.c

Теперь пришло время разделить содержимое на три файла: `main.c`, `tdouble.h` и `tdouble.c`.
Объект определяется двумя файлами: заголовочным файлом и файлом исходного кода C.

tdouble.h

~~~C
 1 #pragma once
 2 
 3 #include <glib-object.h>
 4 
 5 #define T_TYPE_DOUBLE  (t_double_get_type ())
 6 G_DECLARE_FINAL_TYPE (TDouble, t_double, T, DOUBLE, GObject)
 7 
 8 gboolean
 9 t_double_get_value (TDouble *self, double *value);
10 
11 void
12 t_double_set_value (TDouble *self, double value);
13 
14 TDouble *
15 t_double_new (double value);
~~~

- Заголовочные файлы являются публичными, т.е. они открыты для любых файлов.
Заголовочные файлы включают макросы, которые дают информацию о типе, приведение типов и проверку типов, а также публичные функции.
- 1: Директива `#pragma once` предотвращает чтение заголовочного файла компилятором два или более раз.
Она официально не определена, но широко поддерживается многими компиляторами.
- 5-6: `T_TYPE_DOUBLE` является публичным.
`G_DECLARE_FINAL_TYPE` раскрывается в публичные определения.
- 8-12: Объявления публичных функций.
Это геттер и сеттер значения объекта.
Они называются "методами экземпляра", которые используются в объектно-ориентированных языках.
- 14-15: Функция создания экземпляра объекта.

tdouble.c

~~~C
 1 #include "tdouble.h"
 2 
 3 struct _TDouble {
 4   GObject parent;
 5   double value;
 6 };
 7 
 8 G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)
 9 
10 static void
11 t_double_class_init (TDoubleClass *class) {
12 }
13 
14 static void
15 t_double_init (TDouble *self) {
16 }
17 
18 gboolean
19 t_double_get_value (TDouble *self, double *value) {
20   g_return_val_if_fail (T_IS_DOUBLE (self), FALSE);
21 
22   *value = self->value;
23   return TRUE;
24 }
25 
26 void
27 t_double_set_value (TDouble *self, double value) {
28   g_return_if_fail (T_IS_DOUBLE (self));
29 
30   self->value = value;
31 }
32 
33 TDouble *
34 t_double_new (double value) {
35   TDouble *d;
36 
37   d = g_object_new (T_TYPE_DOUBLE, NULL);
38   d->value = value;
39   return d;
40 }
~~~

- 3-6: Объявление структуры экземпляра.
Поскольку макрос `G_DECLARE_FINAL_TYPE` генерирует `typedef struct _TDouble TDouble`, имя тега структуры должно быть `_TDouble`.
- 8: Макрос `G_DEFINE_TYPE`.
- 10-16: Функции инициализации класса и экземпляра.
На данный момент они ничего не делают.
- 18-24: Геттер. Аргумент `value` — это указатель на переменную типа double.
Присваивает значение объекта (`self->value`) переменной.
Если это удается, возвращает TRUE.
Функция `g_return_val_if_fail` используется для проверки типа аргумента.
Если аргумент `self` не является типом TDouble, она выводит ошибку в журнал и немедленно возвращает FALSE.
Эта функция используется для сообщения об ошибке программиста.
Вы не должны использовать ее для ошибки времени выполнения.
См. [Glib API Reference -- Error Reporting](https://docs.gtk.org/glib/error-reporting.html) для получения дополнительной информации.
Функция `g_return_val_if_fail` не используется в статических функциях класса, которые являются приватными, потому что статические функции вызываются только из функций в том же файле, и вызывающая сторона знает тип параметров.
- 26-31: Сеттер.
Функция `g_return_if_fail` используется для проверки типа аргумента.
Эта функция не возвращает никакого значения.
Поскольку тип `t_double_set_value` — `void`, никакое значение не будет возвращено.
Поэтому мы используем `g_return_if_fail` вместо `g_return_val_if_fail`.
- 33-40: Функция создания экземпляра объекта.
У нее есть один параметр `value` для установки значения объекта.
- 37: Эта функция использует `g_object_new` для создания экземпляра объекта.
Аргумент `T_TYPE_DOUBLE` раскрывается в функцию `t_double_get_type ()`.
Если это первый вызов `t_double_get_type`, будет выполнена регистрация типа.

main.c

~~~C
 1 #include <glib-object.h>
 2 #include "tdouble.h"
 3 
 4 int
 5 main (int argc, char **argv) {
 6   TDouble *d;
 7   double value;
 8 
 9   d = t_double_new (10.0);
10   if (t_double_get_value (d, &value))
11     g_print ("t_double_get_value succesfully assigned %lf to value.\n", value);
12   else
13     g_print ("t_double_get_value failed.\n");
14 
15   t_double_set_value (d, -20.0);
16   g_print ("Now, set d (tDouble object) with %lf.\n", -20.0);
17   if (t_double_get_value (d, &value))
18     g_print ("t_double_get_value succesfully assigned %lf to value.\n", value);
19   else
20     g_print ("t_double_get_value failed.\n");
21   g_object_unref (d);
22 
23   return 0;
24 }
25 
~~~

- 2: Включает `tdouble.h`.
Это необходимо для доступа к объекту TDouble.
- 9: Создает экземпляр объекта TDouble и устанавливает `d` для указания на объект.
- 10-13: Тестирует геттер объекта.
- 15-20: Тестирует сеттер объекта.
- 21: Освобождает экземпляр `d`.

Исходные файлы находятся в [src/tdouble1](../src/tdouble1).
Измените ваш текущий каталог на указанный выше каталог и введите следующее.

~~~
$ cd src/tdouble1
$ meson setup _build
$ ninja -C _build
~~~

Затем выполните программу.

~~~
$ cd src/tdouble1; _build/example6
t_double_get_value succesfully assigned 10.000000 to value.
Now, set d (tDouble object) with -20.000000.
t_double_get_value succesfully assigned -20.000000 to value.
~~~

Этот пример очень прост.
Но любой объект имеет заголовочный файл и файл исходного кода C, как этот.
И они следуют соглашению.
Вы, вероятно, осознаете важность соглашения.
Для получения дополнительной информации обратитесь к [GObject API Reference -- Conventions](https://docs.gtk.org/gobject/concepts.html#conventions).

## Функции

Функции объектов открыты для других объектов.
Они подобны публичным методам в объектно-ориентированных языках.
Они фактически называются "методами экземпляра" в справочнике GObject API.

Естественно добавить операторы вычисления к объектам TDouble, потому что они представляют вещественные числа.
Например, `t_double_add` добавляет значение экземпляра и другого экземпляра.
Затем она создает новый экземпляр TDouble, который имеет значение их суммы.

~~~C
TDouble *
t_double_add (TDouble *self, TDouble *other) {
  g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
  g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
  double value;

  if (! t_double_get_value (other, &value))
    return NULL;
  return t_double_new (self->value + value);
}
~~~

Первый аргумент `self` — это экземпляр, которому принадлежит функция.
Второй аргумент `other` — это другой экземпляр TDouble.

Доступ к значению `self` можно получить через `self->value`, но не используйте `other->value` для получения значения `other`.
Вместо этого используйте функцию `t_double_get_value`.
Потому что `self` — это экземпляр вне `other`.
Обычно структура объекта не открыта для других объектов.
Когда объект A обращается к другому объекту B, A должен использовать публичную функцию, предоставленную B.

## Упражнение

Напишите функции объекта TDouble для вычитания, умножения, деления и изменения знака (унарный минус).
Сравните вашу программу с `tdouble.c` в каталоге [src/tdouble2](../src/tdouble2).

Up: [Readme.md](../Readme.md),  Prev: [Section 2](sec2.md), Next: [Section 4](sec4.md)
