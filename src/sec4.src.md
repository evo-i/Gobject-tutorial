# Сигналы

## Сигналы

Сигналы обеспечивают средство коммуникации между объектами.
Сигналы испускаются, когда что-то происходит или завершается.

Шаги для программирования сигнала показаны ниже.

1. Зарегистрировать сигнал.
Сигнал принадлежит объекту, поэтому регистрация выполняется в функции инициализации класса объекта.
2. Написать обработчик сигнала.
Обработчик сигнала — это функция, которая вызывается при испускании сигнала.
3. Связать сигнал и обработчик.
Сигналы связываются с обработчиками с помощью `g_connect_signal` или родственных функций.
4. Испустить сигнал.

Шаги один и четыре выполняются на объекте, к которому принадлежит сигнал.
Шаг три обычно выполняется вне объекта.

Процесс работы с сигналами сложен, и требуется много времени, чтобы объяснить все возможности.
Содержание этого раздела ограничено минимальными вещами для написания простого сигнала и не обязательно точно.
Если вам нужна точная информация, обратитесь к справочнику GObject API.
Есть четыре части, которые описывают сигналы.

- [Type System Concepts -- signals](https://docs.gtk.org/gobject/concepts.html#signals)
- [Funcions (g\_signal\_XXX series)](https://docs.gtk.org/gobject/#functions)
- [Funcions Macros (g\_signal\_XXX series)](https://docs.gtk.org/gobject/#function_macros)
- [GObject Tutorial -- How to create and use signals](https://docs.gtk.org/gobject/tutorial.html#how-to-create-and-use-signals)

## Регистрация сигнала

Пример в этом разделе — это сигнал, испускаемый при делении на ноль.
Сначала нам нужно определить имя сигнала.
Имя сигнала состоит из букв, цифр, дефиса (`-`) и подчеркивания (`_`).
Первый символ имени должен быть буквой.
Таким образом, строка "div-by-zero" подходит для имени сигнала.

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

Требуется много времени, чтобы объяснить каждый параметр.
В настоящий момент я просто покажу вам вызов функции `g_signal_new`, извлеченный из `tdouble.c`.

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

- `t_double_signal` — это статическая переменная типа guint.
Тип guint — это то же самое, что unsigned int.
Ей присваивается идентификатор сигнала, возвращаемый функцией `g_signal_new`.
- Второй параметр — это тип (GType) объекта, к которому принадлежит сигнал.
`G_TYPE_FROM_CLASS (class)` возвращает тип, соответствующий классу (`class` — это указатель на класс объекта).
- Третий параметр — это флаг сигнала.
Для объяснения этого флага необходимо много страниц.
Поэтому я хочу пока опустить их.
Приведенный выше аргумент может использоваться во многих случаях.
Определение описано в [GObject API Reference -- SignalFlags](https://docs.gtk.org/gobject/flags.SignalFlags.html).
- Тип возвращаемого значения — G_TYPE_NONE, что означает, что обработчик сигнала не возвращает значение.
- `n_params` — это количество параметров.
Этот сигнал не имеет параметров, поэтому это ноль.

Эта функция расположена в функции инициализации класса (`t_double_class_init`).

Вы можете использовать другие функции, такие как `g_signal_newv`.
См. [GObject API Reference](https://docs.gtk.org/gobject/func.signal_newv.html) для получения подробной информации.

## Обработчик сигнала

Обработчик сигнала — это функция, которая вызывается при испускании сигнала.
Обработчик имеет два параметра.

- Экземпляр, к которому принадлежит сигнал
- Указатель на пользовательские данные, которые передаются при подключении сигнала.

Сигнал "div-by-zero" не требует пользовательских данных.

~~~C
void div_by_zero_cb (TDouble *self, gpointer user_data) { ... ... ...}
~~~

Первый аргумент `self` — это экземпляр, на котором испускается сигнал.
Вы можете опустить второй параметр.

~~~C
void div_by_zero_cb (TDouble *self) { ... ... ...}
~~~

Если сигнал имеет параметры, параметры находятся между экземпляром и пользовательскими данными.
Например, обработчик сигнала "window-added" для GtkApplication:

~~~C
void window_added (GtkApplication* self, GtkWindow* window, gpointer user_data);
~~~

Второй аргумент `window` — это параметр сигнала.
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

Сигнал и обработчик связываются с помощью функции [`g_signal_connect`](https://docs.gtk.org/gobject/func.signal_connect.html).

~~~C
g_signal_connect (self, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
~~~

- `self` — это экземпляр, к которому принадлежит сигнал.
- Второй аргумент — это имя сигнала.
- Третий аргумент — это обработчик сигнала.
Он должен быть приведен с помощью `G_CALLBACK`.
- Последний аргумент — это пользовательские данные.
Сигнал не требует пользовательских данных, поэтому присваивается NULL.

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

- Первый параметр — это экземпляр, который испускает сигнал.
- Второй параметр — это идентификатор сигнала.
Идентификатор сигнала — это значение, возвращаемое функцией `g_signal_new`.
- Третий параметр — это детализация.
Сигнал "div-by-zero" не имеет детализации, поэтому аргумент равен нулю.
Детализация не объясняется в этом разделе, но обычно вы можете поставить ноль в качестве третьего аргумента.
Если вы хотите узнать подробности, обратитесь к [GObject API Reference -- Signal Detail](https://docs.gtk.org/gobject/concepts.html#the-detail-argument).

Если сигнал имеет параметры, они являются четвертым и последующими аргументами.

## Пример

Пример программы находится в [src/tdouble3](tdouble3).

tdouble.h

@@@include
tdouble3/tdouble.h
@@@

tdouble.c

@@@include
tdouble3/tdouble.c
@@@

main.c

@@@include
tdouble3/main.c
@@@

Перейдите в каталог src/tdouble3 и наберите следующее.

~~~
$ meson setup _build
$ ninja -C _build
~~~

Затем исполняемый файл `tdouble` создается в каталоге `_build`.
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

Возможно, вы подумали, что странно, что сообщение об ошибке было установлено в `main.c`.
Действительно, ошибка происходит в `tdouble.c`, поэтому сообщение должно управляться самим `tdouble.c`.
Система GObject имеет обработчик сигнала по умолчанию, который устанавливается в самом объекте.
Обработчик сигнала по умолчанию также называется "обработчик по умолчанию" или "обработчик метода объекта".

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

Отличие от `g_signal_new` в четвертом параметре.
`g_signal_new` устанавливает обработчик по умолчанию со смещением указателя на функцию в структуре класса.
Если объект является производным, он имеет свою собственную область класса, поэтому вы можете установить обработчик по умолчанию с помощью `g_signal_new`.
Но объект финального типа не имеет своей собственной области класса, поэтому невозможно установить обработчик по умолчанию с помощью `g_signal_new`.
Вот почему мы используем `g_signal_new_class_handler`.

Файл C `tdouble.c` изменен следующим образом.
Функция `div_by_zero_default_cb` добавлена, и `g_signal_new_class_handler` заменяет `g_signal_new`.
Обработчик сигнала по умолчанию не имеет параметра `user_data`.
Параметр `user_data` устанавливается в функциях семейства `g_signal_connect`, когда пользователь подключает свой собственный обработчик сигнала к сигналу.
Обработчик сигнала по умолчанию управляется экземпляром, а не пользователем.
Поэтому никакие пользовательские данные не передаются в качестве аргумента.

@@@include
tdouble4/tdouble.c div_by_zero_default_cb t_double_class_init
@@@

`g_signal_connect` и `div_by_zero_cb` удалены из `main.c`.

Скомпилируйте и выполните.

~~~
$ cd src/tdouble4; _build/tdouble
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error: division by zero.

-10.000000 = -10.000000
~~~

Исходный файл находится в каталоге [src/tdouble4](tdouble4).

Если вы хотите подключить свой обработчик (предоставленный пользователем) к сигналу, вы все еще можете использовать `g_signal_connect`.
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

Затем, когда сигнал испускается, вызываются и предоставленный пользователем обработчик, и обработчик по умолчанию.
Скомпилируйте и выполните, затем на вашем дисплее будет показано следующее.

~~~
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error happens in main.c.

Error: division by zero.

-10.000000 = -10.000000
~~~

Это говорит нам, что сначала вызывается предоставленный пользователем обработчик, затем вызывается обработчик по умолчанию.
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

Скомпилируйте и выполните, затем:

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

Исходные файлы находятся в [src/tdouble5](tdouble5).

## Флаг сигнала

Порядок вызова обработчиков описан в [GObject API Reference -- Sigmal emission](https://docs.gtk.org/gobject/concepts.html#signal-emission).

Порядок зависит от флага сигнала, который устанавливается в `g_signal_new` или `g_signal_new_class_handler`.
Существует три флага, которые относятся к порядку вызова обработчиков.

- `G_SIGNAL_RUN_FIRST`: обработчик по умолчанию вызывается перед любым предоставленным пользователем обработчиком.
- `G_SIGNAL_RUN_LAST`: обработчик по умолчанию вызывается после обычного предоставленного пользователем обработчика (не подключенного с помощью `g_signal_connect_after`).
- `G_SIGNAL_RUN_CLEANUP`: обработчик по умолчанию вызывается после любого предоставленного пользователем обработчика.

`G_SIGNAL_RUN_LAST` является наиболее подходящим во многих случаях.

Другие флаги сигналов описаны в [GObject API Reference](https://docs.gtk.org/gobject/flags.SignalFlags.html).
