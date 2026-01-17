Вверх: [Readme.md](../Readme.md),  Назад: [Раздел 8](sec8.md)

# Интерфейс

Интерфейс похож на абстрактный класс.
Интерфейс определяет виртуальные функции, которые должны быть переопределены функцией в другом создаваемом объекте.

Этот раздел содержит простой пример, TComparable.
TComparable — это интерфейс.
Он определяет функции для сравнения.
Они следующие:

- `t_comparable_cmp (self, other)`: Сравнивает `self` и `other`.
Первый аргумент `self` — это экземпляр, на котором выполняется `t_comparable_cmp`.
Второй аргумент `other` — это другой экземпляр.
Эта функция должна быть переопределена в объекте, который реализует интерфейс.
  - Если `self` равен `other`, `t_comparable_cmp` возвращает 0.
  - Если `self` больше чем `other`, `t_comparable_cmp` возвращает 1.
  - Если `self` меньше чем `other`, `t_comparable_cmp` возвращает -1.
  - Если происходит ошибка, `t_comparable_cmp` возвращает -2.
- `t_comparable_eq (self, other)`: Возвращает TRUE, если `self` равен `other`.
В противном случае возвращает FALSE.
Обратите внимание, что FALSE возвращается даже при возникновении ошибки.
- `t_comparable_gt (self, other)`: Возвращает TRUE, если `self` больше чем `other`.
В противном случае возвращает FALSE.
- `t_comparable_lt (self, other)`: Возвращает TRUE, если `self` меньше чем `other`.
В противном случае возвращает FALSE.
- `t_comparable_ge (self, other)`: Возвращает TRUE, если `self` больше или равен `other`.
В противном случае возвращает FALSE.
- `t_comparable_le (self, other)`: Возвращает TRUE, если `self` меньше или равен `other`.
В противном случае возвращает FALSE.

Числа и строки можно сравнивать.
TInt, TDouble и TStr реализуют интерфейс TComparable, поэтому они могут использовать вышеуказанные функции.
Кроме того, TNumStr может использовать эти функции, потому что является дочерним классом TStr.

Например,

~~~C
TInt *i1 = t_int_new_with_value (10);
TInt *i2 = t_int_new_with_value (20);
t_comparable_eq (T_COMPARABLE (i1), T_COMPARABLE (i2)); /* => FALSE */
t_comparable_lt (T_COMPARABLE (i1), T_COMPARABLE (i2)); /* => TRUE */
~~~

В чем разница между интерфейсом и абстрактным классом?
Виртуальные функции в абстрактном классе переопределяются функцией в его классах-потомках.
Виртуальные функции в интерфейсе переопределяются функцией в любых классах.
Сравните TNumber и TComparable.

- Функция `t_number_add` переопределяется в TIntClass и TDoubleClass.
Она не может быть переопределена в TStrClass, потому что TStr не является потомком TNumber.
- Функция `t_comparable_cmp` переопределяется в TIntClass, TDoubleClass и TStrClass.

## Интерфейс TComparable

Определение интерфейсов похоже на определение объектов.

- Используйте `G_DECLARE_INTERFACE` вместо `G_DECLARE_FINAL_TYPE`.
- Используйте `G_DEFINE_INTERFACE` вместо `G_DEFINE_TYPE`.

Теперь рассмотрим заголовочный файл.

~~~C
 1 #pragma once
 2 
 3 #include <glib-object.h>
 4 
 5 #define T_TYPE_COMPARABLE  (t_comparable_get_type ())
 6 G_DECLARE_INTERFACE (TComparable, t_comparable, T, COMPARABLE, GObject)
 7 
 8 struct _TComparableInterface {
 9   GTypeInterface parent;
10   /* signal */
11   void (*arg_error) (TComparable *self);
12   /* virtual function */
13   int (*cmp) (TComparable *self, TComparable *other);
14 };
15 
16 /* t_comparable_cmp */
17 /* if self > other, then returns 1 */
18 /* if self = other, then returns 0 */
19 /* if self < other, then returns -1 */
20 /* if error happens, then returns -2 */
21 
22 int
23 t_comparable_cmp (TComparable *self, TComparable *other);
24 
25 gboolean
26 t_comparable_eq (TComparable *self, TComparable *other);
27 
28 gboolean
29 t_comparable_gt (TComparable *self, TComparable *other);
30 
31 gboolean
32 t_comparable_lt (TComparable *self, TComparable *other);
33 
34 gboolean
35 t_comparable_ge (TComparable *self, TComparable *other);
36 
37 gboolean
38 t_comparable_le (TComparable *self, TComparable *other);
~~~

- 6: Макрос `G_DECLARE_INTERFACE`.
Последний параметр является предварительным требованием интерфейса.
Предварительным требованием для TComparable является GObject.
Таким образом, любой другой объект, кроме потомков GObject, например GVariant, не может реализовать TComparable.
Предварительное требование — это GType интерфейса или класса.
Этот макрос раскрывается в:
  - Объявление `t_comparable_get_type()`.
  - `Typedef struct _TComparableInterface TComparableInterface`
  - Макрос `T_COMPARABLE ()`. Он приводит экземпляр к типу TComparable.
  - Макрос `T_IS_COMPARABLE ()`. Он проверяет, является ли тип экземпляра `T_TYPE_COMPARABLE`.
  - Макрос `T_COMPARABLE_GET_IFACE ()`. Он получает интерфейс экземпляра, который передается в качестве аргумента.
- 8-14: Структура `TComparableInterface`.
Это похоже на структуру класса.
Первый член — это родительский интерфейс.
Родителем `TComparableInterface` является `GTypeInterface`.
`GTypeInterface` является базой для всех типов интерфейсов.
Это аналогично `GTypeClass`, который является базой для всех типов классов.
`GTypeClass` — это первый член структуры `GObjectClass`.
(См. `gobject.h`. Обратите внимание, что `GObjectClass` — это то же самое, что `struct _GObjectClass`.)
Следующий член — это указатель `arg_error` на обработчик сигнала по умолчанию для сигнала "arg-error".
Этот сигнал генерируется, когда второй аргумент функции сравнения неподходящий.
Например, если `self` — это TInt, а `other` — это TStr, оба они являются экземплярами Comparable.
Но их *нельзя* сравнить.
Это потому, что `other` не является TNumber.
Последний член `cmp` — это указатель на метод сравнения.
Это виртуальная функция, которая должна быть переопределена функцией в объекте, реализующем интерфейс.
- 22-38: Публичные функции.

C файл `tcomparable.c` следующий.

~~~C
 1 #include "tcomparable.h"
 2 
 3 static guint t_comparable_signal;
 4 
 5 G_DEFINE_INTERFACE (TComparable, t_comparable, G_TYPE_OBJECT)
 6 
 7 static void
 8 arg_error_default_cb (TComparable *self) {
 9   g_printerr ("\nTComparable: argument error.\n");
10 }
11 
12 static void
13 t_comparable_default_init (TComparableInterface *iface) {
14   /* virtual function */
15   iface->cmp = NULL;
16   /* argument error signal */
17   iface->arg_error = arg_error_default_cb;
18   t_comparable_signal =
19   g_signal_new ("arg-error",
20                 T_TYPE_COMPARABLE,
21                 G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
22                 G_STRUCT_OFFSET (TComparableInterface, arg_error),
23                 NULL /* accumulator */,
24                 NULL /* accumulator data */,
25                 NULL /* C marshaller */,
26                 G_TYPE_NONE /* return_type */,
27                 0     /* n_params */
28                 );
29 }
30 
31 int
32 t_comparable_cmp (TComparable *self, TComparable *other) {
33   g_return_val_if_fail (T_IS_COMPARABLE (self), -2);
34 
35   TComparableInterface *iface = T_COMPARABLE_GET_IFACE (self);
36   
37   return (iface->cmp == NULL ? -2 : iface->cmp (self, other));
38 }
39 
40 gboolean
41 t_comparable_eq (TComparable *self, TComparable *other) {
42   return (t_comparable_cmp (self, other) == 0);
43 }
44 
45 gboolean
46 t_comparable_gt (TComparable *self, TComparable *other) {
47   return (t_comparable_cmp (self, other) == 1);
48 }
49 
50 gboolean
51 t_comparable_lt (TComparable *self, TComparable *other) {
52   return (t_comparable_cmp (self, other) == -1);
53 }
54 
55 gboolean
56 t_comparable_ge (TComparable *self, TComparable *other) {
57   int result = t_comparable_cmp (self, other);
58   return (result == 1 || result == 0);
59 }
60 
61 gboolean
62 t_comparable_le (TComparable *self, TComparable *other) {
63   int result = t_comparable_cmp (self, other);
64   return (result == -1 || result == 0);
65 }
~~~

- 5: Макрос `G_DEFINE_INTERFACE`.
Третий параметр — это тип предварительного требования.
Этот макрос раскрывается в:
  - Объявление `t_comparable_default_init ()`.
  - Определение `t_comparable_get_type ()`.
- 7-10: `arg_error_default_cb` — обработчик сигнала по умолчанию для сигнала "arg-error".
- 12- 29: Функция `t_comparable_default_init`.
Эта функция похожа на функцию инициализации класса.
Она инициализирует структуру `TComparableInterface`.
- 15: Присваивает NULL значению `cmp`.
Таким образом, метод сравнения не работает до тех пор, пока класс реализации не переопределит его.
- 17: Устанавливает обработчик сигнала по умолчанию для сигнала "arg-error".
- 18-28: Создает сигнал "arg-error".
- 31-38: Функция `t_comparable_cmp`.
Она проверяет тип `self` на первой строке.
Если он не является сравниваемым, она записывает сообщение об ошибке и возвращает -2 (ошибка).
Если `iface->cmp` равен NULL (это означает, что метод класса не был переопределен), то возвращается -2.
В противном случае вызывается метод класса и возвращается значение, возвращенное методом класса.
- 40-65: Публичные функции.
Эти пять функций основаны на `t_comparable_cmp`.
Поэтому для них не требуется переопределение.
Например, `t_comparable_eq` просто вызывает `t_comparable_cmp`.
И возвращает TRUE, если `t_comparable_cmp` возвращает ноль.
В противном случае возвращает FALSE.

Эта программа использует сигнал для передачи пользователю информации об ошибке типа аргумента.
Эта ошибка обычно является программной ошибкой, а не ошибкой времени выполнения.
Использование сигнала для сообщения о программной ошибке — не лучший способ.
Лучший способ — использовать `g_return_if_fail`.
Причина, по которой я создал этот сигнал, заключается в том, что я хотел показать, как реализовать сигнал в интерфейсах.

## Реализация интерфейса

TInt, TDouble и TStr реализуют TComparable.
Сначала рассмотрим TInt.
Заголовочный файл такой же, как и раньше.
Реализация написана в C файле.

`tint.c` следующий.

~~~C
  1 #include "../tnumber/tnumber.h"
  2 #include "../tnumber/tint.h"
  3 #include "../tnumber/tdouble.h"
  4 #include "tcomparable.h"
  5 
  6 enum {
  7   PROP_0,
  8   PROP_INT,
  9   N_PROPERTIES
 10 };
 11 
 12 static GParamSpec *int_properties[N_PROPERTIES] = {NULL, };
 13 
 14 struct _TInt {
 15   TNumber parent;
 16   int value;
 17 };
 18 
 19 static void t_comparable_interface_init (TComparableInterface *iface);
 20 
 21 G_DEFINE_TYPE_WITH_CODE (TInt, t_int, T_TYPE_NUMBER,
 22                          G_IMPLEMENT_INTERFACE (T_TYPE_COMPARABLE, t_comparable_interface_init))
 23 
 24 static int
 25 t_int_comparable_cmp (TComparable *self, TComparable *other) {
 26   if (! T_IS_NUMBER (other)) {
 27     g_signal_emit_by_name (self, "arg-error");
 28     return -2;
 29   }
 30 
 31   int i;
 32   double s, o;
 33 
 34   s = (double) T_INT (self)->value;
 35   if (T_IS_INT (other)) {
 36     g_object_get (other, "value", &i, NULL);
 37     o = (double) i;
 38   } else
 39     g_object_get (other, "value", &o, NULL);
 40   if (s > o)
 41     return 1;
 42   else if (s == o)
 43     return 0;
 44   else if (s < o)
 45     return -1;
 46   else /* This can't happen. */
 47     return -2;
 48 }
 49 
 50 static void
 51 t_comparable_interface_init (TComparableInterface *iface) {
 52   iface->cmp = t_int_comparable_cmp;
 53 }
 54 
 55 static void
 56 t_int_set_property (GObject *object, guint property_id, const GValue *value, GParamSpec *pspec) {
 57   TInt *self = T_INT (object);
 58 
 59   if (property_id == PROP_INT)
 60     self->value = g_value_get_int (value);
 61   else
 62     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 63 }
 64 
 65 static void
 66 t_int_get_property (GObject *object, guint property_id, GValue *value, GParamSpec *pspec) {
 67   TInt *self = T_INT (object);
 68 
 69   if (property_id == PROP_INT)
 70     g_value_set_int (value, self->value);
 71   else
 72     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 73 }
 74 
 75 static void
 76 t_int_init (TInt *self) {
 77 }
 78 
 79 /* arithmetic operator */
 80 /* These operators create a new instance and return a pointer to it. */
 81 #define t_int_binary_op(op) \
 82   int i; \
 83   double d; \
 84   if (T_IS_INT (other)) { \
 85     g_object_get (T_INT (other), "value", &i, NULL); \
 86     return  T_NUMBER (t_int_new_with_value (T_INT(self)->value op i)); \
 87   } else { \
 88     g_object_get (T_DOUBLE (other), "value", &d, NULL); \
 89     return  T_NUMBER (t_int_new_with_value (T_INT(self)->value op (int) d)); \
 90   }
 91 
 92 static TNumber *
 93 t_int_add (TNumber *self, TNumber *other) {
 94   g_return_val_if_fail (T_IS_INT (self), NULL);
 95 
 96   t_int_binary_op (+)
 97 }
 98 
 99 static TNumber *
100 t_int_sub (TNumber *self, TNumber *other) {
101   g_return_val_if_fail (T_IS_INT (self), NULL);
102 
103   t_int_binary_op (-)
104 }
105 
106 static TNumber *
107 t_int_mul (TNumber *self, TNumber *other) {
108   g_return_val_if_fail (T_IS_INT (self), NULL);
109 
110   t_int_binary_op (*)
111 }
112 
113 static TNumber *
114 t_int_div (TNumber *self, TNumber *other) {
115   g_return_val_if_fail (T_IS_INT (self), NULL);
116 
117   int i;
118   double d;
119 
120   if (T_IS_INT (other)) {
121     g_object_get (T_INT (other), "value", &i, NULL);
122     if (i == 0) {
123       g_signal_emit_by_name (self, "div-by-zero");
124       return NULL;
125     } else
126       return  T_NUMBER (t_int_new_with_value (T_INT(self)->value / i));
127   } else {
128     g_object_get (T_DOUBLE (other), "value", &d, NULL);
129     if (d == 0) {
130       g_signal_emit_by_name (self, "div-by-zero");
131       return NULL;
132     } else
133       return  T_NUMBER (t_int_new_with_value (T_INT(self)->value / (int)  d));
134   }
135 }
136 
137 static TNumber *
138 t_int_uminus (TNumber *self) {
139   g_return_val_if_fail (T_IS_INT (self), NULL);
140 
141   return T_NUMBER (t_int_new_with_value (- T_INT(self)->value));
142 }
143 
144 static char *
145 t_int_to_s (TNumber *self) {
146   g_return_val_if_fail (T_IS_INT (self), NULL);
147 
148   int i;
149 
150   g_object_get (T_INT (self), "value", &i, NULL); 
151   return g_strdup_printf ("%d", i);
152 }
153 
154 static void
155 t_int_class_init (TIntClass *class) {
156   TNumberClass *tnumber_class = T_NUMBER_CLASS (class);
157   GObjectClass *gobject_class = G_OBJECT_CLASS (class);
158 
159   /* override virtual functions */
160   tnumber_class->add = t_int_add;
161   tnumber_class->sub = t_int_sub;
162   tnumber_class->mul = t_int_mul;
163   tnumber_class->div = t_int_div;
164   tnumber_class->uminus = t_int_uminus;
165   tnumber_class->to_s = t_int_to_s;
166 
167   gobject_class->set_property = t_int_set_property;
168   gobject_class->get_property = t_int_get_property;
169   int_properties[PROP_INT] = g_param_spec_int ("value", "val", "Integer value", G_MININT, G_MAXINT, 0, G_PARAM_READWRITE);
170   g_object_class_install_properties (gobject_class, N_PROPERTIES, int_properties);
171 }
172 
173 TInt *
174 t_int_new_with_value (int value) {
175   TInt *i;
176 
177   i = g_object_new (T_TYPE_INT, "value", value, NULL);
178   return i;
179 }
180 
181 TInt *
182 t_int_new (void) {
183   TInt *i;
184 
185   i = g_object_new (T_TYPE_INT, NULL);
186   return i;
187 }
~~~

- 4: Необходимо включить заголовочный файл TComparable.
- 19: Объявление функции `t_comparable_interface_init ()`.
Это объявление должно быть сделано перед макросом `G_DEFINE_TYPE_WITH_CODE`.
- 21-22: Макрос `G_DEFINE_TYPE_WITH_CODE`.
Последний параметр — это макрос `G_IMPLEMENT_INTERFACE`.
Второй параметр `G_IMPLEMENT_INTERFACE` — это `t_comparable_interface_init`.
Эти два макроса раскрываются в:
  - Объявление `t_int_class_init ()`.
  - Объявление `t_int_init ()`.
  - Определение статической переменной `t_int_parent_class`, которая указывает на родительский класс.
  - Определение `t_int_get_type ()`.
Эта функция включает `g_type_register_static_simple ()` и `g_type_add_interface_static ()`.
Функция `g_type_register_static_simple ()` — это удобная версия `g_type_register_static ()`.
Она регистрирует тип TInt в системе типов.
Функция `g_type_add_interface_static ()` добавляет тип интерфейса к типу экземпляра.
Хороший пример есть в [GObject Reference Manual, Interfaces](https://docs.gtk.org/gobject/concepts.html#non-instantiatable-classed-types-interfaces).
Если вы хотите узнать, как писать код без макросов, см. [`tint_without_macro.c`](../src/tcomparble/tint_without_macro.c).
- 24-48: `t_int_comparable_cmp` — это функция для сравнения экземпляра TInt с экземпляром TNumber.
- 26-29: Проверяет тип `other`.
Если тип аргумента не TNumber, генерируется сигнал "arg-error" с помощью `g_signal_emit_by_name`.
- 34: Преобразует `self` в double.
- 35-39: Получает значение `other`, и если это TInt, то значение приводится к double.
- 40-47: сравнивает `s` и `o` и возвращает 1, 0, -1 и -2.
- 50-53: `t_comparable_interface_init`.
Эта функция вызывается в процессе инициализации TInt.
Функция `t_int_comparable_cmp` присваивается `iface->cmp`.

`tdouble.c` почти такой же, как `tint.c`.
Эти два объекта можно сравнивать, потому что int приводится к double перед сравнением.

`tstr.c` следующий.

~~~C
  1 #include "../tstr/tstr.h"
  2 #include "tcomparable.h"
  3 
  4 enum {
  5   PROP_0,
  6   PROP_STRING,
  7   N_PROPERTIES
  8 };
  9 
 10 static GParamSpec *str_properties[N_PROPERTIES] = {NULL, };
 11 
 12 typedef struct {
 13   char *string;
 14 } TStrPrivate;
 15 
 16 static void t_comparable_interface_init (TComparableInterface *iface);
 17 
 18 G_DEFINE_TYPE_WITH_CODE (TStr, t_str, G_TYPE_OBJECT,
 19                          G_ADD_PRIVATE (TStr)
 20                          G_IMPLEMENT_INTERFACE (T_TYPE_COMPARABLE, t_comparable_interface_init))
 21 
 22 static void
 23 t_str_set_property (GObject *object, guint property_id, const GValue *value, GParamSpec *pspec) {
 24   TStr *self = T_STR (object);
 25 
 26 /* The returned value of the function g_value_get_string can be NULL. */
 27 /* The function t_str_set_string calls a class method, */
 28 /* which is expected to rewrite in the descendant object. */
 29   if (property_id == PROP_STRING)
 30     t_str_set_string (self, g_value_get_string (value));
 31   else
 32     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 33 }
 34 
 35 static void
 36 t_str_get_property (GObject *object, guint property_id, GValue *value, GParamSpec *pspec) {
 37   TStr *self = T_STR (object);
 38   TStrPrivate *priv = t_str_get_instance_private (self);
 39 
 40 /* The second argument of the function g_value_set_string can be NULL. */
 41   if (property_id == PROP_STRING)
 42     g_value_set_string (value, priv->string);
 43   else
 44     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 45 }
 46 
 47 /* This function just set the string. */
 48 /* So, no notify signal is emitted. */
 49 static void
 50 t_str_real_set_string (TStr *self, const char *s) {
 51   TStrPrivate *priv = t_str_get_instance_private (self);
 52 
 53   if (priv->string)
 54     g_free (priv->string);
 55   priv->string = g_strdup (s);
 56 }
 57 
 58 static void
 59 t_str_finalize (GObject *object) {
 60   TStr *self = T_STR (object);
 61   TStrPrivate *priv = t_str_get_instance_private (self);
 62 
 63   if (priv->string)
 64     g_free (priv->string);
 65   G_OBJECT_CLASS (t_str_parent_class)->finalize (object);
 66 }
 67 
 68 static int
 69 t_str_comparable_cmp (TComparable *self, TComparable *other) {
 70   if (! T_IS_STR (other)) {
 71     g_signal_emit_by_name (self, "arg-error");
 72     return -2;
 73   }
 74 
 75   char *s, *o;
 76   int result;
 77 
 78   s = t_str_get_string (T_STR (self));
 79   o = t_str_get_string (T_STR (other));
 80 
 81   if (strcmp (s, o) > 0)
 82     result = 1;
 83   else if (strcmp (s, o) == 0)
 84     result = 0;
 85   else if (strcmp (s, o) < 0)
 86     result = -1;
 87   else /* This can't happen. */
 88     result = -2;
 89   g_free (s);
 90   g_free (o);
 91   return result;
 92 }
 93 
 94 static void
 95 t_comparable_interface_init (TComparableInterface *iface) {
 96   iface->cmp = t_str_comparable_cmp;
 97 }
 98 
 99 static void
100 t_str_init (TStr *self) {
101   TStrPrivate *priv = t_str_get_instance_private (self);
102 
103   priv->string = NULL;
104 }
105 
106 static void
107 t_str_class_init (TStrClass *class) {
108   GObjectClass *gobject_class = G_OBJECT_CLASS (class);
109 
110   gobject_class->finalize = t_str_finalize;
111   gobject_class->set_property = t_str_set_property;
112   gobject_class->get_property = t_str_get_property;
113   str_properties[PROP_STRING] = g_param_spec_string ("string", "str", "string", "", G_PARAM_READWRITE);
114   g_object_class_install_properties (gobject_class, N_PROPERTIES, str_properties);
115 
116   class->set_string = t_str_real_set_string;
117 }
118 
119 /* setter and getter */
120 void
121 t_str_set_string (TStr *self, const char *s) {
122   g_return_if_fail (T_IS_STR (self));
123   TStrClass *class = T_STR_GET_CLASS (self);
124 
125 /* The setter calls the class method 'set_string', */
126 /* which is expected to be overridden by the descendant TNumStr. */
127 /* Therefore, the behavior of the setter is different between TStr and TNumStr. */
128   class->set_string (self, s);
129 }
130 
131 char *
132 t_str_get_string (TStr *self) {
133   g_return_val_if_fail (T_IS_STR (self), NULL);
134   TStrPrivate *priv = t_str_get_instance_private (self);
135 
136   return g_strdup (priv->string);
137 }
138 
139 TStr *
140 t_str_concat (TStr *self, TStr *other) {
141   g_return_val_if_fail (T_IS_STR (self), NULL);
142   g_return_val_if_fail (T_IS_STR (other), NULL);
143 
144   char *s1, *s2, *s3;
145   TStr *str;
146 
147   s1 = t_str_get_string (self);
148   s2 = t_str_get_string (other);
149   if (s1 && s2)
150     s3 = g_strconcat (s1, s2, NULL);
151   else if (s1)
152     s3 = g_strdup (s1);
153   else if (s2)
154     s3 = g_strdup (s2);
155   else
156     s3 = NULL;
157   str = t_str_new_with_string (s3);
158   if (s1) g_free (s1);
159   if (s2) g_free (s2);
160   if (s3) g_free (s3);
161   return str;
162 }
163 
164 /* create a new TStr instance */
165 TStr *
166 t_str_new_with_string (const char *s) {
167   return T_STR (g_object_new (T_TYPE_STR, "string", s, NULL));
168 }
169 
170 TStr *
171 t_str_new (void) {
172   return T_STR (g_object_new (T_TYPE_STR, NULL));
173 }
~~~

- 16: Объявляет функцию `t_comparable_interface_init`.
Она должна быть объявлена перед макросом `G_DEFINE_TYPE_WITH_CODE`.
- 18-20: Макрос `G_DEFINE_TYPE_WITH_CODE`.
Поскольку TStr является производным типом, необходима его приватная область (TStrPrivate).
Макрос `G_ADD_PRIVATE` создает приватную область.
Обратите внимание, что после макроса `G_ADD_PRIVATE` нет запятой.
- 68-92: `t_str_comparable_cmp`.
- 70-73: Проверяет тип `other`.
Если это не TStr, генерируется сигнал "arg-error".
- 78-79: Получает строки `s` и `o` из объектов TStr `self` и `other`.
- 81-88: Сравнивает `s` и `o` с помощью стандартной функции C `strcmp`.
- 89-90: Освобождает `s` и `o`.
- 91: Возвращает результат.
- 94-97: Функция `t_comparable_interface_init`.
Она переопределяет `iface->comp` на `t_str_comparable_cmp`.

TStr можно сравнивать с TStr, но не с TInt и не с TDouble.
Обычно сравнение доступно между двумя экземплярами одного типа.

TNumStr сам по себе не реализует TComparable.
Но он является потомком TStr, поэтому он сравниваем.
Сравнение основано на алфавитном порядке.
Таким образом, "a" больше чем "b", а "three" больше чем "two".

## Тестовая программа.

`main.c` — это тестовая программа.

~~~C
 1 #include <glib-object.h>
 2 #include "tcomparable.h"
 3 #include "../tnumber/tnumber.h"
 4 #include "../tnumber/tint.h"
 5 #include "../tnumber/tdouble.h"
 6 #include "../tstr/tstr.h"
 7 
 8 static void
 9 t_print (const char *cmp, TComparable *c1, TComparable *c2) {
10   char *s1, *s2;
11   TStr *ts1, *ts2, *ts3;
12 
13   ts1 = t_str_new_with_string("\"");
14   if (T_IS_NUMBER (c1))
15     s1 = t_number_to_s (T_NUMBER (c1));
16   else if (T_IS_STR (c1)) {
17     ts2 = t_str_concat (ts1, T_STR (c1));
18     ts3 = t_str_concat (ts2, ts1);
19     s1 = t_str_get_string (T_STR (ts3));
20     g_object_unref (ts2);
21     g_object_unref (ts3);
22   } else {
23     g_print ("c1 isn't TInt, TDouble nor TStr.\n");
24     return;
25   }
26   if (T_IS_NUMBER (c2))
27     s2 = t_number_to_s (T_NUMBER (c2));
28   else if (T_IS_STR (c2)) {
29     ts2 = t_str_concat (ts1, T_STR (c2));
30     ts3 = t_str_concat (ts2, ts1);
31     s2 = t_str_get_string (T_STR (ts3));
32     g_object_unref (ts2);
33     g_object_unref (ts3);
34   } else {
35     g_print ("c2 isn't TInt, TDouble nor TStr.\n");
36     return;
37   }
38   g_print ("%s %s %s.\n", s1, cmp, s2);
39   g_object_unref (ts1);
40   g_free (s1);
41   g_free (s2);
42 }    
43 
44 static void
45 compare (TComparable *c1, TComparable *c2) {
46   if (t_comparable_eq (c1, c2))
47     t_print ("equals", c1, c2);
48   else if (t_comparable_gt (c1, c2))
49     t_print ("is greater than", c1, c2);
50   else if (t_comparable_lt (c1, c2))
51     t_print ("is less than", c1, c2);
52   else if (t_comparable_ge (c1, c2))
53     t_print ("is greater than or equal to", c1, c2);
54   else if (t_comparable_le (c1, c2))
55     t_print ("is less than or equal to", c1, c2);
56   else
57     t_print ("can't compare to", c1, c2);
58 }
59 
60 int
61 main (int argc, char **argv) {
62   const char *one = "one";
63   const char *two = "two";
64   const char *three = "three";
65   TInt *i;
66   TDouble *d;
67   TStr *str1, *str2, *str3;
68 
69   i = t_int_new_with_value (124);
70   d = t_double_new_with_value (123.45);
71   str1 = t_str_new_with_string (one);
72   str2 = t_str_new_with_string (two);
73   str3 = t_str_new_with_string (three);
74 
75   compare (T_COMPARABLE (i), T_COMPARABLE (d));
76   compare (T_COMPARABLE (str1), T_COMPARABLE (str2));
77   compare (T_COMPARABLE (str2), T_COMPARABLE (str3));
78   compare (T_COMPARABLE (i), T_COMPARABLE (str1));
79 
80   g_object_unref (i);
81   g_object_unref (d);
82   g_object_unref (str1);
83   g_object_unref (str2);
84   g_object_unref (str3);
85 
86   return 0;
87 }
~~~

- 8-42: Функция `t_print` имеет три параметра и формирует выходную строку, затем отображает ее на дисплее.
При формировании вывода строки окружаются двойными кавычками.
- 44-58: Функция `compare` сравнивает два объекта TComparable и вызывает `t_print` для отображения результата.
- 60-87: Функция `main`.
- 69-73: Создает экземпляры TInt, TDouble и три TStr.
Им присваиваются значения.
- 75: Сравнивает TInt и TDouble.
- 76-77: Сравнивает два TStr.
- 78: Сравнивает TInt с TStr.
Это вызывает "arg-error".
- 80-84 Освобождает объекты.

## Компиляция и выполнение

Измените текущий каталог на [src/tcomparable](../src/comparable).

~~~
$ cd src/tcomparable
$ meson setup _build
$ ninja -C _build
~~~

Затем выполните его.

~~~
$ _build/tcomparable
124 is greater than 123.450000.
"one" is less than "two".
"two" is greater than "three".

TComparable: argument error.

TComparable: argument error.

TComparable: argument error.

TComparable: argument error.

TComparable: argument error.
124 can't compare to "one".
~~~

## Создание интерфейса без макросов

Мы использовали макросы, такие как `G_DECLARE_INTERFACE`, `G_DEFINE_INTERFACE` для создания интерфейса.
И `G_DEFINE_TYPE_WITH_CODE` для реализации интерфейса.
Мы также можем создать его без макросов.
В каталоге `tcomparable` есть три файла.

- `tcomparable_without_macro.h`
- `tcomparable_without_macro.c`
- `tint_without_macro.c`

Они не используют макросы.
Вместо этого они регистрируют интерфейс или реализацию интерфейса в системе типов напрямую.
Если вы хотите узнать это, см. исходные файлы в [src/tcomparable](../src/tcomparable).

## Система GObject и объектно-ориентированные языки

Если вы знаете какие-либо объектно-ориентированные языки, вы, вероятно, подумали, что GObject и эти языки похожи.
Изучение таких языков очень полезно для понимания системы GObject.

Вверх: [Readme.md](../Readme.md),  Назад: [Раздел 8](sec8.md)
