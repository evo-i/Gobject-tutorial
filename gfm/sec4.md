Up: [Readme.md](../Readme.md),  Prev: [Section 3](sec3.md), Next: [Section 5](sec5.md)

# Сигналы

## Сигналы

Сигналы обеспечивают средство коммуникации между объектами.
Сигналы испускаются, когда что-то происходит или завершается.

Шаги для программирования сигнала показаны ниже.

1. Зарегистрировать сигнал.
Сигнал принадлежит объекту, поэтому регистрация выполняется в функции инициализации класса объекта.
2. Написать обработчик сигнала.
Обработчик сигнала - это функция, которая вызывается при испускании сигнала.
3. Подключить сигнал и обработчик.
Сигналы подключаются к обработчикам с помощью `g_connect_signal` или функций этого семейства.
4. Испустить сигнал.

Шаги первый и четвёртый выполняются на объекте, которому принадлежит сигнал.
Шаг третий обычно выполняется вне объекта.

Процесс работы с сигналами сложен, и требуется много времени, чтобы объяснить все возможности.
Содержание этого раздела ограничено минимальными вещами для написания простого сигнала и не обязательно точно.
Если вам нужна точная информация, обратитесь к справочнику API GObject.
Есть четыре части, которые описывают сигналы.

- [Type System Concepts -- signals](https://docs.gtk.org/gobject/concepts.html#signals)
- [Funcions (g\_signal\_XXX series)](https://docs.gtk.org/gobject/#functions)
- [Funcions Macros (g\_signal\_XXX series)](https://docs.gtk.org/gobject/#function_macros)
- [GObject Tutorial -- How to create and use signals](https://docs.gtk.org/gobject/tutorial.html#how-to-create-and-use-signals)

## Регистрация сигнала

Примером в этом разделе является сигнал, испускаемый при делении на ноль.
Во-первых, нам нужно определить имя сигнала.
Имя сигнала состоит из букв, цифр, дефиса (`-`) и подчёркивания (`_`).
Первый символ имени должен быть буквой.
Итак, строка "div-by-zero" подходит для имени сигнала.

Существует четыре функции для регистрации сигнала.
Мы будем использовать [`g_signal_new`](https://docs.gtk.org/gobject/func.signal_new.html) для сигнала "div-by-zero".

~~~C
guint
g_signal_new (const gchar *signal_name,
              GType itype,
              GSignalFlags signal_flags,
              guint class_offset,
              GSignalAccumulator accumulator,
              gpointer accu_data,
              GSignalCMarshaller c_marshaller,
              GType return_type,
              guint n_params,
              ...);
~~~

Требуется много объяснений для каждого параметра.
В настоящее время я просто покажу вам вызов функции `g_signal_new`, извлечённый из `tdouble.c`.

~~~C
t_double_signal =
g_signal_new ("div-by-zero",
              G_TYPE_FROM_CLASS (class),
              G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
              0 /* class offset.Subclass cannot override the class handler (default handler). */,
              NULL /* accumulator */,
              NULL /* accumulator data */,
              NULL /* C marshaller. g_cclosure_marshal_generic() will be used */,
              G_TYPE_NONE /* return_type */,
              0     /* n_params */
              );
~~~

- `t_double_signal` - это статическая переменная типа guint.
Тип guint такой же, как unsigned int.
Ей присваивается идентификатор сигнала, возвращаемый функцией `g_signal_new`.
- Второй параметр - это тип (GType) объекта, которому принадлежит сигнал.
`G_TYPE_FROM_CLASS (class)` возвращает тип, соответствующий классу (`class` - это указатель на класс объекта).
- Третий параметр - это флаг сигнала.
Потребуется много страниц, чтобы объяснить этот флаг.
Поэтому я хочу пока опустить их.
Приведённый выше аргумент может использоваться во многих случаях.
Определение описано в [GObject API Reference -- SignalFlags](https://docs.gtk.org/gobject/flags.SignalFlags.html).
- Возвращаемый тип - G_TYPE_NONE, что означает, что обработчик сигнала не возвращает значение.
- `n_params` - это количество параметров.
Этот сигнал не имеет параметров, поэтому это ноль.

Эта функция находится в функции инициализации класса (`t_double_class_init`).

Вы можете использовать другие функции, такие как `g_signal_newv`.
См. [GObject API Reference](https://docs.gtk.org/gobject/func.signal_newv.html) для получения подробной информации.

## Обработчик сигнала

Обработчик сигнала - это функция, которая вызывается при испускании сигнала.
Обработчик имеет два параметра.

- Экземпляр, которому принадлежит сигнал
- Указатель на пользовательские данные, которые передаются при подключении сигнала.

Сигналу "div-by-zero" не нужны пользовательские данные.

~~~C
void div_by_zero_cb (TDouble *self, gpointer user_data) { ... ... ...}
~~~

Первый аргумент `self` - это экземпляр, на котором испускается сигнал.
Вы можете опустить второй параметр.

~~~C
void div_by_zero_cb (TDouble *self) { ... ... ...}
~~~

Если сигнал имеет параметры, параметры находятся между экземпляром и пользовательскими данными.
Например, обработчик сигнала "window-added" на GtkApplication:

~~~C
void window_added (GtkApplication* self, GtkWindow* window, gpointer user_data);
~~~

Второй аргумент `window` является параметром сигнала.
Сигнал "window-added" испускается, когда новое окно добавляется в приложение.
Параметр `window` указывает на вновь добавленное окно.
См. [GTK API reference](https://docs.gtk.org/gtk4/signal.Application.window-added.html) для получения дополнительной информации.

Обработчик сигнала "div-by-zero" просто показывает сообщение об ошибке.

~~~C
static void
div_by_zero_cb (TDouble *self, gpointer user_data) {
  g_print ("\nError: division by zero.\n\n");
}
~~~

## Подключение сигнала

Сигнал и обработчик подключаются с помощью функции [`g_signal_connect`](https://docs.gtk.org/gobject/func.signal_connect.html).

~~~C
g_signal_connect (self, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
~~~

- `self` - это экземпляр, которому принадлежит сигнал.
- Второй аргумент - это имя сигнала.
- Третий аргумент - это обработчик сигнала.
Он должен быть приведён к типу с помощью `G_CALLBACK`.
- Последний аргумент - это пользовательские данные.
Сигналу не нужны пользовательские данные, поэтому присваивается NULL.

## Испускание сигнала

Сигналы испускаются на объекте.
Ниже приведена часть `tdouble.c`.

~~~C
TDouble *
t_double_div (TDouble *self, TDouble *other) {
... ... ...
  if ((! t_double_get_value (other, &value)))
    return NULL;
  else if (value == 0) {
    g_signal_emit (self, t_double_signal, 0);
    return NULL;
  }
  return t_double_new (self->value / value);
}
~~~

Если делитель равен нулю, сигнал испускается.
[`g_signal_emit`](https://docs.gtk.org/gobject/func.signal_emit.html) имеет три параметра.

- Первый параметр - это экземпляр, который испускает сигнал.
- Второй параметр - это идентификатор сигнала.
Идентификатор сигнала - это значение, возвращаемое функцией `g_signal_new`.
- Третий параметр - это детализация.
Сигнал "div-by-zero" не имеет детализации, поэтому аргумент равен нулю.
Детализация не объясняется в этом разделе, но обычно вы можете поставить ноль в качестве третьего аргумента.
Если вы хотите узнать подробности, обратитесь к [GObject API Reference -- Signal Detail](https://docs.gtk.org/gobject/concepts.html#the-detail-argument).

Если сигнал имеет параметры, они являются четвёртым и последующими аргументами.

## Пример

Пример программы находится в [src/tdouble3](../src/tdouble3).

tdouble.h

~~~C
 1 #pragma once
 2 
 3 #include <glib-object.h>
 4 
 5 #define T_TYPE_DOUBLE  (t_double_get_type ())
 6 G_DECLARE_FINAL_TYPE (TDouble, t_double, T, DOUBLE, GObject)
 7 
 8 /* getter and setter */
 9 gboolean
10 t_double_get_value (TDouble *self, double *value);
11 
12 void
13 t_double_set_value (TDouble *self, double value);
14 
15 /* arithmetic operator */
16 /* These operators create a new instance and return a pointer to it. */
17 TDouble *
18 t_double_add (TDouble *self, TDouble *other);
19 
20 TDouble *
21 t_double_sub (TDouble *self, TDouble *other);
22 
23 TDouble *
24 t_double_mul (TDouble *self, TDouble *other);
25 
26 TDouble *
27 t_double_div (TDouble *self, TDouble *other);
28 
29 TDouble *
30 t_double_uminus (TDouble *self);
31 
32 /* create a new TDouble instance */
33 TDouble *
34 t_double_new (double value);
~~~

tdouble.c

~~~C
  1 #include "tdouble.h"
  2 
  3 static guint t_double_signal;
  4 
  5 struct _TDouble {
  6   GObject parent;
  7   double value;
  8 };
  9 
 10 G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)
 11 
 12 static void
 13 t_double_class_init (TDoubleClass *class) {
 14   t_double_signal = g_signal_new ("div-by-zero",
 15                                  G_TYPE_FROM_CLASS (class),
 16                                  G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
 17                                  0 /* class offset.Subclass cannot override the class handler (default handler). */,
 18                                  NULL /* accumulator */,
 19                                  NULL /* accumulator data */,
 20                                  NULL /* C marshaller. g_cclosure_marshal_generic() will be used */,
 21                                  G_TYPE_NONE /* return_type */,
 22                                  0     /* n_params */
 23                                  );
 24 }
 25 
 26 static void
 27 t_double_init (TDouble *self) {
 28 }
 29 
 30 /* getter and setter */
 31 gboolean
 32 t_double_get_value (TDouble *self, double *value) {
 33   g_return_val_if_fail (T_IS_DOUBLE (self), FALSE);
 34 
 35   *value = self->value;
 36   return TRUE;
 37 }
 38 
 39 void
 40 t_double_set_value (TDouble *self, double value) {
 41   g_return_if_fail (T_IS_DOUBLE (self));
 42 
 43   self->value = value;
 44 }
 45 
 46 /* arithmetic operator */
 47 /* These operators create a new instance and return a pointer to it. */
 48 #define t_double_binary_op(op) \
 49   if (! t_double_get_value (other, &value)) \
 50     return NULL; \
 51   return t_double_new (self->value op value);
 52 
 53 TDouble *
 54 t_double_add (TDouble *self, TDouble *other) {
 55   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 56   g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
 57   double value;
 58 
 59   t_double_binary_op (+)
 60 }
 61 
 62 TDouble *
 63 t_double_sub (TDouble *self, TDouble *other) {
 64   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 65   g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
 66   double value;
 67 
 68   t_double_binary_op (-)
 69 }
 70 
 71 TDouble *
 72 t_double_mul (TDouble *self, TDouble *other) {
 73   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 74   g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
 75   double value;
 76 
 77   t_double_binary_op (*)
 78 }
 79 
 80 TDouble *
 81 t_double_div (TDouble *self, TDouble *other) {
 82   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 83   g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
 84   double value;
 85 
 86   if ((! t_double_get_value (other, &value)))
 87     return NULL;
 88   else if (value == 0) {
 89     g_signal_emit (self, t_double_signal, 0);
 90     return NULL;
 91   }
 92   return t_double_new (self->value / value);
 93 }
 94 TDouble *
 95 t_double_uminus (TDouble *self) {
 96   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 97 
 98   return t_double_new (-self->value);
 99 }
100 
101 TDouble *
102 t_double_new (double value) {
103   TDouble *d;
104 
105   d = g_object_new (T_TYPE_DOUBLE, NULL);
106   d->value = value;
107   return d;
108 }
~~~

main.c

~~~C
 1 #include <glib-object.h>
 2 #include "tdouble.h"
 3 
 4 static void
 5 div_by_zero_cb (TDouble *self, gpointer user_data) {
 6   g_printerr ("\nError: division by zero.\n\n");
 7 }
 8 
 9 static void
10 t_print (char *op, TDouble *d1, TDouble *d2, TDouble *d3) {
11   double v1, v2, v3;
12 
13   if (! t_double_get_value (d1, &v1))
14     return;
15   if (! t_double_get_value (d2, &v2))
16     return;
17   if (! t_double_get_value (d3, &v3))
18     return;
19 
20   g_print ("%lf %s %lf = %lf\n", v1, op, v2, v3);
21 }
22 
23 int
24 main (int argc, char **argv) {
25   TDouble *d1, *d2, *d3;
26   double v1, v3;
27 
28   d1 = t_double_new (10.0);
29   d2 = t_double_new (20.0);
30   if ((d3 = t_double_add (d1, d2)) != NULL) {
31     t_print ("+", d1, d2, d3);
32     g_object_unref (d3);
33   }
34 
35   if ((d3 = t_double_sub (d1, d2)) != NULL) {
36     t_print ("-", d1, d2, d3);
37     g_object_unref (d3);
38   }
39 
40   if ((d3 = t_double_mul (d1, d2)) != NULL) {
41     t_print ("*", d1, d2, d3);
42     g_object_unref (d3);
43   }
44 
45   if ((d3 = t_double_div (d1, d2)) != NULL) {
46     t_print ("/", d1, d2, d3);
47     g_object_unref (d3);
48   }
49 
50   g_signal_connect (d1, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
51   t_double_set_value (d2, 0.0);
52   if ((d3 = t_double_div (d1, d2)) != NULL) {
53     t_print ("/", d1, d2, d3);
54     g_object_unref (d3);
55   }
56 
57   if ((d3 = t_double_uminus (d1)) != NULL && (t_double_get_value (d1, &v1)) && (t_double_get_value (d3, &v3))) {
58     g_print ("-%lf = %lf\n", v1, v3);
59     g_object_unref (d3);
60   }
61 
62   g_object_unref (d1);
63   g_object_unref (d2);
64 
65   return 0;
66 }
67 
~~~

Измените текущий каталог на src/tdouble3 и введите следующее.

~~~
$ meson setup _build
$ ninja -C _build
~~~

Затем исполняемый файл `tdouble` будет создан в каталоге `_build`.
Выполните его.

~~~
$ _build/tdouble
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error: division by zero.

-10.000000 = -10.000000
~~~

## Обработчик сигнала по умолчанию

Вы, возможно, подумали, что было странно, что сообщение об ошибке было установлено в `main.c`.
Действительно, ошибка происходит в `tdouble.c`, поэтому сообщение должно управляться самим `tdouble.c`.
Система GObject имеет обработчик сигнала по умолчанию, который устанавливается в самом объекте.
Обработчик сигнала по умолчанию также называется "обработчик по умолчанию" или "метод-обработчик объекта".

Вы можете установить обработчик по умолчанию с помощью `g_signal_new_class_handler`.

~~~c
guint
g_signal_new_class_handler (const gchar *signal_name,
                            GType itype,
                            GSignalFlags signal_flags,
                            GCallback class_handler, /*default signal handler */
                            GSignalAccumulator accumulator,
                            gpointer accu_data,
                            GSignalCMarshaller c_marshaller,
                            GType return_type,
                            guint n_params,
                            ...);
~~~

Отличие от `g_signal_new` в четвёртом параметре.
`g_signal_new` устанавливает обработчик по умолчанию с помощью смещения указателя на функцию в структуре класса.
Если объект является производным, у него есть своя собственная область класса, поэтому вы можете установить обработчик по умолчанию с помощью `g_signal_new`.
Но объект финального типа не имеет своей собственной области класса, поэтому невозможно установить обработчик по умолчанию с помощью `g_signal_new`.
Вот почему мы используем `g_signal_new_class_handler`.

Файл C `tdouble.c` изменён следующим образом.
Добавлена функция `div_by_zero_default_cb` и `g_signal_new_class_handler` заменяет `g_signal_new`.
Обработчик сигнала по умолчанию не имеет параметра `user_data`.
Параметр `user_data` устанавливается в функциях семейства `g_signal_connect`, когда пользователь подключает свой собственный обработчик сигнала к сигналу.
Обработчик сигнала по умолчанию управляется экземпляром, а не пользователем.
Поэтому никакие пользовательские данные не передаются в качестве аргумента.

~~~C
 1 static void
 2 div_by_zero_default_cb (TDouble *self) {
 3   g_printerr ("\nError: division by zero.\n\n");
 4 }
 5 
 6 static void
 7 t_double_class_init (TDoubleClass *class) {
 8   t_double_signal =
 9   g_signal_new_class_handler ("div-by-zero",
10                               G_TYPE_FROM_CLASS (class),
11                               G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
12                               G_CALLBACK (div_by_zero_default_cb),
13                               NULL /* accumulator */,
14                               NULL /* accumulator data */,
15                               NULL /* C marshaller */,
16                               G_TYPE_NONE /* return_type */,
17                               0     /* n_params */
18                               );
19 }
~~~

`g_signal_connect` и `div_by_zero_cb` удалены из `main.c`.

Скомпилируйте и выполните его.

~~~
$ cd src/tdouble4; _build/tdouble
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error: division by zero.

-10.000000 = -10.000000
~~~

Исходный файл находится в каталоге [src/tdouble4](../src/tdouble4).

Если вы хотите подключить свой обработчик (предоставленный пользователем обработчик) к сигналу, вы все ещё можете использовать `g_signal_connect`.
Добавьте следующее в `main.c`.

~~~C
static void
div_by_zero_cb (TDouble *self, gpointer user_data) {
  g_print ("\nError happens in main.c.\n");
}

int
main (int argc, char **argv) {
... ... ...
  g_signal_connect (d1, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
... ... ...
}
~~~

Тогда будут вызваны и предоставленный пользователем обработчик, и обработчик по умолчанию, когда сигнал будет испущен.
Скомпилируйте и выполните его, затем на вашем дисплее появится следующее.

~~~
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error happens in main.c.

Error: division by zero.

-10.000000 = -10.000000
~~~

Это говорит нам, что предоставленный пользователем обработчик вызывается первым, затем вызывается обработчик по умолчанию.
Если вы хотите, чтобы ваш обработчик вызывался после обработчика по умолчанию, вы можете использовать `g_signal_connect_after`.
Добавьте строки ниже в `main.c` снова.

~~~C
static void
div_by_zero_after_cb (TDouble *self, gpointer user_data) {
  g_print ("\nError has happened in main.c and an error message has been displayed.\n");
}

int
main (int argc, char **argv) {
... ... ...
  g_signal_connect_after (d1, "div-by-zero", G_CALLBACK (div_by_zero_after_cb), NULL);
... ... ...
}
~~~

Скомпилируйте и выполните его, затем:

~~~
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error happens in main.c.

Error: division by zero.

Error has happened in main.c and an error message has been displayed.

-10.000000 = -10.000000
~~~

Исходные файлы находятся в [src/tdouble5](../src/tdouble5).

## Флаг сигнала

Порядок, в котором вызываются обработчики, описан в [GObject API Reference -- Sigmal emission](https://docs.gtk.org/gobject/concepts.html#signal-emission).

Порядок зависит от флага сигнала, который устанавливается в `g_signal_new` или `g_signal_new_class_handler`.
Есть три флага, которые относятся к порядку вызова обработчиков.

- `G_SIGNAL_RUN_FIRST`: обработчик по умолчанию вызывается перед любым предоставленным пользователем обработчиком.
- `G_SIGNAL_RUN_LAST`: обработчик по умолчанию вызывается после обычного предоставленного пользователем обработчика (не подключённого с помощью `g_signal_connect_after`).
- `G_SIGNAL_RUN_CLEANUP`: обработчик по умолчанию вызывается после любого предоставленного пользователем обработчика.

`G_SIGNAL_RUN_LAST` является наиболее подходящим во многих случаях.

Другие флаги сигналов описаны в [GObject API Reference](https://docs.gtk.org/gobject/flags.SignalFlags.html).

Up: [Readme.md](../Readme.md),  Prev: [Section 3](sec3.md), Next: [Section 5](sec5.md)
