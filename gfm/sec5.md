Up: [Readme.md](../Readme.md),  Prev: [Section 4](sec4.md), Next: [Section 6](sec6.md)

# Свойства

Система GObject предоставляет свойства.
Свойства - это значения, хранящиеся в экземплярах, которые являются потомками GObject, и они доступны другим экземплярам.
К ним можно обращаться по их именам.

Например, GtkWindow имеет свойства "title", "default-width", "default-height" и другие.
Строка "title" - это имя свойства.
Имя свойства - это строка, которая начинается с буквы, за которой следуют буквы, цифры, тире ('-') или подчеркивание ('_').
Тире и подчеркивание используются в качестве разделителей, но их нельзя смешивать.
Использование тире более эффективно, чем подчеркивания.
Например, "value", "double" и "double-value" - корректные имена свойств.
"\_value" или "-value" - некорректные.

Свойства имеют различные типы значений.
Тип свойства "title" - строка.
Тип "default-width" и "default-height" - целое число.

Свойства устанавливаются и получаются с помощью функций, определенных в GObject.

- Свойства могут быть установлены несколькими функциями GObject.
Часто используются `g_object_new` и `g_object_set`.
- Свойства могут быть получены несколькими функциями GObject.
Часто используется `g_object_get`.

Функции выше принадлежат GObject, но они могут быть использованы для любого объекта-потомка GObject.
Ниже приведен пример GtkWindow, который является объектом-потомком GObject.

Экземпляр создается и его свойства устанавливаются с помощью `g_object_new`.

~~~C
GtkWindow *win;
win = g_object_new (GTK_TYPE_WINDOW, "title", "Hello", "default-width", 800, "default-height", 600, NULL);
~~~

Пример выше создает экземпляр GtkWindow и устанавливает свойства.

- Свойство "title" устанавливается в "Hello".
- Свойство "default-width" устанавливается в 800.
- Свойство "default-height" устанавливается в 600.

Последний параметр `g_object_new` - это `NULL`, который обозначает конец списка свойств.

Если вы уже создали экземпляр GtkWindow и хотите установить его заголовок, вы можете использовать `g_object_set`.

~~~C
GtkWindow *win;
win = g_object_new (GTK_TYPE_WINDOW, NULL);
g_object_set (win, "title", "Good bye", NULL);
~~~

Вы можете получить значение свойства с помощью `g_object_get`.

~~~C
GtkWindow *win;
char *title;
int width, height;

win = g_object_new (GTK_TYPE_WINDOW, "title", "Hello", "default-width", 800, "default-height", 600, NULL);
g_object_get (win, "title", &title, "default-width", &width, "default-height", &height, NULL);
g_print ("%s, %d, %d\n", title, width, height);
g_free (title);
~~~

Остаток этого раздела посвящен реализации свойств в потомке GObject.
Он разделен на две части.

- Регистрация свойства
- Определение методов класса `set_property` и `get_property` для дополнения `g_object_set` и `g_object_get`.

## GParamSpec

GParamSpec - это фундаментальный объект.
GParamSpec и GObject не имеют отношения родитель-потомок.
GParamSpec содержит информацию о параметрах.
"ParamSpec" - это сокращение от "Parameter specification" (спецификация параметра).

Например,

~~~C
double_property = g_param_spec_double ("value", "val", "Double value",
                                       -G_MAXDOUBLE, G_MAXDOUBLE, 0.0,
                                        G_PARAM_READWRITE);
~~~

Эта функция создает экземпляр GParamSpec, точнее экземпляр GParamSpecDouble.
GParamSpecDouble - это потомок GParamSpec.

Экземпляр содержит информацию:

- Тип значения - double.
Суффикс имени функции, `double` в `g_param_spec_double`, указывает на тип.
- Имя - "value".
- Краткое имя - "val".
- Описание - "Double value".
- Минимальное значение - -G\_MAXDOUBLE.
G\_MAXDOUBLE - это максимальное значение, которое может храниться в double.
Это описано в [GLib(2.68.1) Reference Manual -- G\_MAXDOUBLE and G\_MINDOUBLE](https://developer-old.gnome.org/glib/stable/glib-Basic-Types.html#G-MINDOUBLE:CAPS).
Вы можете подумать, что наименьшее значение double - это G\_MINDOUBLE, но это не так.
G\_MINDOUBLE - это минимальное положительное значение, которое может храниться в double.
- Максимальное значение - G\_MAXDOUBLE.
- Значение по умолчанию - 0.0.
- `G_PARAM_READWRITE` - это флаг.
`G_PARAM_READWRITE` означает, что параметр доступен для чтения и записи.

Для получения дополнительной информации обратитесь к справочнику API GObject.

- [GParamSpec and its subclasses](https://docs.gtk.org/gobject/index.html#classes)
- [g\_param\_spec\_double and similar functions](https://docs.gtk.org/gobject/index.html#functions)
- [GValue](https://docs.gtk.org/gobject/struct.Value.html)

GParamSpec используется для регистрации свойств GObject.
Это извлечено из tdouble.c в [src/tdouble6](../src/tdouble6).

~~~C
#define PROP_DOUBLE 1
static GParamSpec *double_property = NULL;

static void
t_double_class_init (TDoubleClass *class) {
  GObjectClass *gobject_class = G_OBJECT_CLASS (class);

  gobject_class->set_property = t_double_set_property;
  gobject_class->get_property = t_double_get_property;
  double_property = g_param_spec_double ("value", "val", "Double value", -G_MAXDOUBLE, G_MAXDOUBLE, 0.0, G_PARAM_READWRITE);
  g_object_class_install_property (gobject_class, PROP_DOUBLE, double_property);
}
~~~

Переменная `double_property` статическая.
Экземпляр GParamSpec присваивается `double_property`.

Функция `g_object_class_install_property` устанавливает свойство.
Она должна быть вызвана после переопределения методов `set_property` и `get_property`.
Эти методы будут объяснены позже.
Аргументы - это класс TDoubleClass, PROP\_DOUBLE (идентификатор свойства) и экземпляр GParamSpec.
Идентификатор свойства используется для идентификации свойства в `tdouble.c`.
Это положительное целое число.

## Переопределение методов класса set\_property и get\_property

Значения свойств различаются от экземпляра к экземпляру.
Поэтому значение сохраняется в каждом экземпляре объекта.

Функция `g_object_set` принимает значение в качестве аргумента и сохраняет его.
Но как `g_object_set` узнает, в какой экземпляр сохранять?
Она компилируется до создания объекта.
Таким образом, она вообще не знает, куда сохранить значение.
Эта часть должна быть запрограммирована автором объекта с помощью переопределения.

Функция `g_object_set` сначала проверяет свойство и значение, затем создает GValue (общее значение) из значения.
И затем она вызывает функцию, на которую указывает `set_property` в классе.
Посмотрите на диаграмму ниже.

![Overriding `set_property` class method](../image/class_property1.png)

Элемент `set_property` в классе GObjectClass указывает на `g_object_do_set_property` в программе GObject, которая создается компиляцией `gobject.c`.
Часть GObjectClass структуры TDoubleClass (она такая же, как TDoubleClass, потому что TDoubleClass не имеет собственной области) инициализируется копированием содержимого GObjectClass.
Поэтому `set_property` в классе TDoubleClass указывает на `g_object_do_set_property` в программе GObject.
Но `g_object_do_set_property` не сохраняет значение в экземпляр TDouble.
Автор объекта TDouble создает функцию `t_double_set_property` в `tdouble.c`.
И присваивает адрес `t_double_set_property` элементу `set_property` в классе TDoubleClass.
Это показано красной кривой на диаграмме.
В результате `g_object_set` вызывает `t_double_set_property` вместо `g_object_do_set_property` (красная пунктирная кривая), и значение будет сохранено в экземпляре TDouble.
Посмотрите на функцию `t_double_class_init` выше.
Она изменяет элемент `gobject_class->set_property` так, чтобы он указывал на функцию `t_double_set_property`.
Функция `g_object_set` смотрит на TDoubleClass и вызывает функцию, на которую указывает элемент `set_property`.

Программа `t_double_set_property` и `t_double_get_property` будет показана позже.

## GValue

GValue - это общее значение.
GValue состоит из типа и значения.

Тип может быть любым GType.
Таблица ниже показывает некоторые GType, но не все.

|GType           |C type|type name |примечания                |
|:---------------|:-----|:---------|:-------------------------|
|G\_TYPE\_CHAR   |char  |gchar     |                          |
|G\_TYPE\_BOOLEAN|int   |gboolean  |                          |
|G\_TYPE\_INT    |int   |gint      |                          |
|G\_TYPE\_FLOAT  |float |gfloat    |                          |
|G\_TYPE\_DOUBLE |double|gdouble   |                          |
|G\_TYPE\_STRING |      |gchararray|C-строка с нулевым окончанием|
|G\_TYPE\_PARAM  |      |GParam    |GParamSpec                |
|G\_TYPE\_OBJECT |      |GObject   |                          |
|G\_TYPE\_VARIANT|      |GVariant  |                          |

Если тип GValue `value` - это `G_TYPE_DOUBLE`, `value` можно получить с помощью функции `g_value_get_double`.

~~~C
GValue value;
value = ... ... ... (объект GValue присвоен. Его тип - double.)
double v;
v = g_value_get_double (&value);
~~~

И наоборот, вы можете установить GValue `value` с помощью `g_value_set_double`.

~~~C
g_value_set_double (value, 123.45);
~~~

Обратитесь к [GObject API Reference -- GValue](https://docs.gtk.org/gobject/struct.Value.html) для получения дополнительной информации.

## t\_double\_set\_property и t\_double\_get\_property

`g_object_set` создает GValue из значения свойства, переданного в качестве аргумента.
И вызывает функцию, на которую указывает `set_property` в классе.
Функция объявлена в структуре GObjectClass.

~~~C
struct  _GObjectClass
{
  ... ... ...
  ... ... ...
  /* overridable methods */
  void       (*set_property)    (GObject        *object,
                                 guint           property_id,
                                 const GValue   *value,
                                 GParamSpec     *pspec);
  void       (*get_property)    (GObject        *object,
                                 guint           property_id,
                                 GValue         *value,
                                 GParamSpec     *pspec);
  ... ... ...
  ... ... ...
};
~~~

`t_double_set_property` просто получает значение из GValue `value` и сохраняет его в экземпляр TDouble.

~~~C
1 static void
2 t_double_set_property (GObject *object, guint property_id, const GValue *value, GParamSpec *pspec) {
3   TDouble *self = T_DOUBLE (object);
4 
5   if (property_id == PROP_DOUBLE)
6     self->value = g_value_get_double (value);
7   else
8     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
9 }
~~~

- 3: Приводит `object` к объекту TDouble `self`.
- 6: Устанавливает `self->value`.
Присваиваемое значение получается с помощью функции `g_value_get_double`.

Аналогично, `t_double_get_property` сохраняет `self->value` в GValue.

~~~C
 1 static void
 2 t_double_get_property (GObject *object, guint property_id, GValue *value, GParamSpec *pspec) {
 3   TDouble *self = T_DOUBLE (object);
 4 
 5   if (property_id == PROP_DOUBLE)
 6     g_value_set_double (value, self->value);
 7   else
 8     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 9 
10 }
~~~

## Сигнал notify

GObject испускает сигнал "notify" при установке свойства.
Когда вы подключаете сигнал "notify" к вашему обработчику, вы можете указать деталь, которая является именем свойства.
Деталь добавляется к имени сигнала с разделителем "::".

~~~C
g_signal_connect (G_OBJECT (d1), "notify::value", G_CALLBACK (notify_cb), NULL);
~~~

Если вы не указываете детали, обработчик вызывается при установке любых свойств.
Поэтому обычно деталь устанавливается.

Сигнал notify не означает, что значение свойства изменилось.
Он испускается даже если устанавливается то же самое значение.
Вы можете захотеть испускать сигнал notify только когда свойство действительно изменяется.
В этом случае вы определяете GPramSpec с флагом `G_PARAM_EXPLICIT_NOTIFY`.
Тогда сигнал notify не испускается автоматически.
Вместо этого вы вызываете функцию `g_object_notify_by_pspec` для явного испускания сигнала "notify", когда значение свойства действительно изменяется.

Возможно создать setter и getter для свойства.
Но если вы просто устанавливаете элемент экземпляра в вашем setter, сигнал notify не испускается.

~~~C
void
t_double_set_value (TDouble *self, double value) {
  g_return_if_fail (T_IS_DOUBLE (self));

  self->value = value; /* Просто устанавливается d->value. Сигнал "notify" не испускается. */
}
~~~

Пользователи будут озадачены, если захотят поймать сигнал "notify".
Одно решение - использовать `g_object_set` в вашем setter.
Тогда сигнал notify будет испущен, даже если пользователь использует функцию setter.

~~~C
void
t_double_set_value (TDouble *d, double value) {
  g_return_if_fail (T_IS_DOUBLE (d));

  g_object_set (d, "value", value, NULL); /* Используется g_object_set. Сигнал "notify" будет испущен. */
}
~~~

Другое решение - использовать `g_object_notify_by_pspec` для явного испускания сигнала.
В любом случае, если вы создаете setter для вашего свойства, будьте осторожны с сигналом notify.

## Определение более чем одного свойства

Если вы определяете более одного свойства, используйте массив идентификаторов свойств.
Вам будет полезно посмотреть исходные файлы Gtk, такие как `gtklabel.c`.
GtkLabel имеет 18 свойств.

Есть пример в директории [src/tdouble6](../src/tdouble6).

## Упражнение

Создайте объект TInt.
Он похож на TDouble, но тип значения - int.
Определите сигнал "div-by-zero" и свойство "value".

Сравните ваш ответ с файлами в директории [src/tint](../src/tint).

Up: [Readme.md](../Readme.md),  Prev: [Section 4](sec4.md), Next: [Section 6](sec6.md)
