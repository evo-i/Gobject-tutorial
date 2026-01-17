Up: [Readme.md](../Readme.md),  Prev: [Section 1](sec1.md), Next: [Section 3](sec3.md)

# GObject

## Класс и экземпляр

Экземпляр GObject создается с помощью функции `g_object_new`.
GObject имеет не только экземпляры, но и классы.

- Класс GObject создается при первом вызове `g_object_new`.
И существует только один класс GObject.
- Экземпляр GObject создается при каждом вызове `g_object_new`.
Таким образом, может существовать два или более экземпляров GObject.

В широком смысле GObject означает объект, который включает свой класс и экземпляры.
В узком смысле GObject — это определение структуры C.

~~~C
typedef struct _GObject  GObject;
struct  _GObject
{
  GTypeInstance  g_type_instance;
  
  /*< private >*/
  guint          ref_count;  /* (atomic) */
  GData         *qdata;
};
~~~

Функция `g_object_new` выделяет память размером с GObject, инициализирует память и возвращает указатель на память.
Память является экземпляром GObject.

Точно так же класс GObject — это память, выделенная `g_object_new`, и его структура определена с помощью GObjectClass.
Следующее извлечено из `gobject.h`.
Но вам не нужно знать детали структуры сейчас.

~~~C
struct  _GObjectClass
{
  GTypeClass   g_type_class;

  /*< private >*/
  GSList      *construct_properties;

  /*< public >*/
  /* seldom overridden */
  GObject*   (*constructor)     (GType                  type,
                                 guint                  n_construct_properties,
                                 GObjectConstructParam *construct_properties);
  /* overridable methods */
  void       (*set_property)		(GObject        *object,
                                         guint           property_id,
                                         const GValue   *value,
                                         GParamSpec     *pspec);
  void       (*get_property)		(GObject        *object,
                                         guint           property_id,
                                         GValue         *value,
                                         GParamSpec     *pspec);
  void       (*dispose)			(GObject        *object);
  void       (*finalize)		(GObject        *object);
  /* seldom overridden */
  void       (*dispatch_properties_changed) (GObject      *object,
					     guint	   n_pspecs,
					     GParamSpec  **pspecs);
  /* signals */
  void	     (*notify)			(GObject	*object,
					 GParamSpec	*pspec);

  /* called when done constructing */
  void	     (*constructed)		(GObject	*object);

  /*< private >*/
  gsize		flags;

  gsize         n_construct_properties;

  gpointer pspecs;
  gsize n_pspecs;

  /* padding */
  gpointer	pdummy[3];
};
~~~

Программы для GObject включены в исходные файлы GLib.
Вы можете скачать исходные файлы GLib со [страницы загрузки GNOME](https://download.gnome.org/sources/glib/).

В каталоге [src/misc](../src/misc) в репозитории руководства по GObject есть примеры программ.
Вы можете скомпилировать их следующим образом:

~~~
$ cd src/misc
$ meson setup _build
$ ninja -C _build
~~~

Одна из программ — это `example1.c`.
Её код выглядит следующим образом.

~~~C
 1 #include <glib-object.h>
 2 
 3 int
 4 main (int argc, char **argv) {
 5   GObject *instance1, *instance2;
 6   GObjectClass *class1, *class2;
 7   
 8   instance1 = g_object_new (G_TYPE_OBJECT, NULL);
 9   instance2 = g_object_new (G_TYPE_OBJECT, NULL);
10   g_print ("The address of instance1 is %p\n", instance1);
11   g_print ("The address of instance2 is %p\n", instance2);
12 
13   class1 = G_OBJECT_GET_CLASS (instance1);
14   class2 = G_OBJECT_GET_CLASS (instance2);
15   g_print ("The address of the class of instance1 is %p\n", class1);
16   g_print ("The address of the class of instance2 is %p\n", class2);
17 
18   g_object_unref (instance1);
19   g_object_unref (instance2);
20 
21   return 0;
22 }
23 
~~~

- 5-6: `instance1` и `instance2` — это указатели, которые указывают на экземпляры GObject.
`class1` и `class2` указывают на класс экземпляров.
- 8-11: Функция `g_object_new` создает экземпляр GObject.
Экземпляр GObject — это фрагмент памяти, который имеет структуру GObject (`struct _GObject`).
Аргумент `G_TYPE_OBJECT` — это тип GObject.
Этот тип отличается от типа языка C, такого как `char` или `int`.
Существует *система типов*, которая является базовой системой системы GObject.
Каждый тип данных, такой как GObject, должен быть зарегистрирован в системе типов.
Система типов имеет серию функций для регистрации.
Если одна из функций вызывается, то система типов определяет значение типа `GType` для объекта и возвращает его вызывающей стороне.
`GType` — это беззнаковое длинное целое число на моем компьютере, но это зависит от аппаратного обеспечения.
`g_object_new` выделяет память размером с GObject и возвращает указатель на начальный адрес памяти.
После создания эта программа отображает адреса экземпляров.
- 13-16: Макрос `G_OBJECT_GET_CLASS` возвращает указатель на класс аргумента.
Следовательно, `class1` указывает на класс `instance1`, а `class2` указывает на класс `instance2` соответственно.
Отображаются адреса двух классов.
- 18-19: `g_object_unref` будет объяснено в следующем подразделе.
Он уничтожает экземпляры, и память освобождается.

Теперь выполним программу.

~~~
$ cd src/misc; _build/example1
The address of instance1 is 0x55895eaf7ad0
The address of instance2 is 0x55895eaf7af0
The address of the class of instance1 is 0x55895eaf7880
The address of the class of instance2 is 0x55895eaf7880
~~~

Расположения двух экземпляров `instance1` и `instance2` различны.
Каждый экземпляр имеет свою собственную память.
Расположения двух классов `class1` и `class2` одинаковы.
Два экземпляра GObject используют один и тот же класс.

![Class and Instance](../image/class_instance.png)

## Счётчик ссылок

Экземпляр GObject имеет свою собственную память.
Они выделяются системой при создании.
Если он становится бесполезным, память должна быть освобождена.
Однако как мы можем определить, что он бесполезен?
Система GObject предоставляет счётчик ссылок для решения этой проблемы.

Экземпляр создается и используется другим экземпляром или основной программой.
То есть на экземпляр ссылаются.
Если на экземпляр ссылаются A и B, то количество ссылок равно двум.
Это число называется *счётчиком ссылок*.
Давайте рассмотрим такой сценарий: 

- A вызывает `g_object_new` и владеет экземпляром G.
A ссылается на G, поэтому счётчик ссылок G равен 1.
- B также хочет использовать G.
B вызывает `g_object_ref` и увеличивает счётчик ссылок на 1.
Теперь счётчик ссылок равен 2.
- A больше не использует G.
A вызывает `g_object_unref` и уменьшает счётчик ссылок на 1.
Теперь счётчик ссылок равен 1.
- B больше не использует G.
B вызывает `g_object_unref` и уменьшает счётчик ссылок на 1.
Теперь счётчик ссылок равен 0.
- Поскольку счётчик ссылок равен нулю, G знает, что никто на него не ссылается.
G начинает процесс завершения самостоятельно.
G исчезает, и память освобождается.

Программа `example2.c` основана на приведенном выше сценарии.

~~~C
 1 #include <glib-object.h>
 2 
 3 static void
 4 show_ref_count (GObject *instance) {
 5   if (G_IS_OBJECT (instance))
 6     /* Users should not use ref_count member in their program. */
 7     /* This is only for demonstration. */
 8     g_print ("Reference count is %d.\n", instance->ref_count);
 9   else
10     g_print ("Instance is not GObject.\n");
11 }
12 
13 int
14 main (int argc, char **argv) {
15   GObject *instance;
16   
17   instance = g_object_new (G_TYPE_OBJECT, NULL);
18   g_print ("Call g_object_new.\n");
19   show_ref_count (instance);
20   g_object_ref (instance);
21   g_print ("Call g_object_ref.\n");
22   show_ref_count (instance);
23   g_object_unref (instance);
24   g_print ("Call g_object_unref.\n");
25   show_ref_count (instance);
26   g_object_unref (instance);
27   g_print ("Call g_object_unref.\n");
28   g_print ("Now the reference count is zero and the instance is destroyed.\n");
29   g_print ("The instance memories are possibly returned to the system.\n");
30   g_print ("Therefore, the access to the same address may cause a segmentation error.\n");
31   
32   return 0;
33 }
~~~

Теперь выполним программу.

~~~
$ cd src/misc; _build/example2
bash: cd: src/misc: No such file or directory
Call g_object_new.
Reference count is 1.
Call g_object_ref.
Reference count is 2.
Call g_object_unref.
Reference count is 1.
Call g_object_unref.
Now the reference count is zero and the instance is destroyed.
The instance memories are possibly returned to the system.
Therefore, the access to the same address may cause a segmentation error.
~~~

`example2` показывает:

- `g_object_new` создает новый экземпляр GObject и устанавливает его счётчик ссылок на 1.
- `g_object_ref` увеличивает счётчик ссылок на 1.
- `g_object_unref` уменьшает счётчик ссылок на 1.
Если счётчик ссылок падает до нуля, экземпляр уничтожает себя.

## Процесс инициализации и уничтожения

Фактический процесс инициализации и уничтожения GObject очень сложен.
Следующее является упрощенным описанием без подробностей.

Инициализация

1. Регистрирует тип GObject в системе типов.
Это делается в процессе инициализации GLib до вызова функции `main`.
(Если компилятор — gcc, то `__attribute__ ((constructor))` используется для квалификации функции инициализации.
Смотрите [руководство GCC](https://gcc.gnu.org/onlinedocs/gcc-10.2.0/gcc/Common-Function-Attributes.html#Common-Function-Attributes).)
2. Выделяет память для структур GObjectClass и GObject.
3. Инициализирует память структуры GObjectClass.
Эта память будет классом GObject.
4. Инициализирует память структуры GObject.
Эта память будет экземпляром GObject.

Этот процесс инициализации выполняется при первом вызове функции `g_object_new`.
При втором и последующих вызовах `g_object_new` она выполняет только два процесса: (1) выделение памяти для структуры GObject (2) инициализация памяти.
`g_object_new` возвращает указатель, который указывает на экземпляр (память, выделенную для структуры GObject).

Уничтожение

1. Уничтожает экземпляр GObject. Память для экземпляра освобождается.

Тип GObject является статическим типом.
Статический тип никогда не уничтожает свой класс.
Поэтому, даже если уничтоженный экземпляр является последним экземпляром, класс остается.

Когда вы пишете код для определения дочернего объекта GObject, важно понимать процесс выше.
Подробный процесс будет объяснен в последующих разделах.

Up: [Readme.md](../Readme.md),  Prev: [Section 1](sec1.md), Next: [Section 3](sec3.md)
