Вверх: [Readme.md](../Readme.md),  Пред: [Раздел 5](sec5.md), След: [Раздел 7](sec7.md)

# Наследуемые и абстрактные типы

## Наследуемый тип

Существует два вида типов: финальный тип и наследуемый тип.
Финальный тип не имеет дочерних объектов.
Наследуемый тип имеет дочерние объекты.

Основное различие между двумя объектами заключается в их классах.
Объекты финального типа не имеют собственной области класса.
Единственным членом класса является его родительский класс.

Наследуемый объект имеет собственную область в классе.
Класс открыт для своих потомков.

`G_DECLARE_DERIVABLE_TYPE` используется для объявления наследуемого типа.
Он записывается в заголовочном файле следующим образом:

~~~C
#define T_TYPE_NUMBER             (t_number_get_type ())
G_DECLARE_DERIVABLE_TYPE (TNumber, t_number, T, NUMBER, GObject)
~~~

## Абстрактный тип

Абстрактный тип не имеет экземпляров.
Этот тип объекта является наследуемым, и его дочерние объекты могут использовать функции и сигналы абстрактного объекта.

Примерами в этом разделе являются объекты TNumber, TInt и TDouble.
TInt и TDouble уже были созданы в предыдущем разделе.
Они представляют целые числа и числа с плавающей точкой соответственно.
Числа являются более абстрактными, чем целые числа и числа с плавающей точкой.

TNumber — это абстрактный объект, который представляет числа.
TNumber является родительским объектом для TInt и TDouble.
TNumber не создается как экземпляр, потому что его тип абстрактный.
Когда экземпляр имеет тип TInt или TDouble, он также является экземпляром TNumber.

TInt и TDouble имеют пять операций: сложение, вычитание, умножение, деление и операцию унарного минуса.
Эти операции могут быть определены для объекта TNumber.

В этом разделе мы определим объект TNumber и пять указанных выше функций.
Кроме того, будет добавлена функция `to_s`.
Она преобразует значение TNumber в строку.
Она похожа на функцию sprintf.
И мы перепишем TInt и TDouble для реализации этих функций.

## Класс TNumber

`tnumber.h` — это заголовочный файл для класса TNumber.

~~~C
 1 #pragma once
 2 
 3 #include <glib-object.h>
 4 
 5 #define T_TYPE_NUMBER             (t_number_get_type ())
 6 G_DECLARE_DERIVABLE_TYPE (TNumber, t_number, T, NUMBER, GObject)
 7 
 8 struct _TNumberClass {
 9   GObjectClass parent_class;
10   TNumber* (*add) (TNumber *self, TNumber *other);
11   TNumber* (*sub) (TNumber *self, TNumber *other);
12   TNumber* (*mul) (TNumber *self, TNumber *other);
13   TNumber* (*div) (TNumber *self, TNumber *other);
14   TNumber* (*uminus) (TNumber *self);
15   char * (*to_s) (TNumber *self);
16   /* signal */
17   void (*div_by_zero) (TNumber *self);
18 };
19 
20 /* arithmetic operator */
21 /* These operators create a new instance and return a pointer to it. */
22 TNumber *
23 t_number_add (TNumber *self, TNumber *other);
24 
25 TNumber *
26 t_number_sub (TNumber *self, TNumber *other);
27 
28 TNumber *
29 t_number_mul (TNumber *self, TNumber *other);
30 
31 TNumber *
32 t_number_div (TNumber *self, TNumber *other);
33 
34 TNumber *
35 t_number_uminus (TNumber *self);
36 
37 char *
38 t_number_to_s (TNumber *self);
~~~

- 6: Макрос `G_DECLARE_DERIVABLE_TYPE`.
Он похож на макрос `G_DECLARE_FINAL_TYPE`.
Разница в том, наследуемый тип или финальный.
`G_DECLARE_DERIVABLE_TYPE` раскрывается в:
  - Объявление функции `t_number_get_type ()`. Эта функция должна быть определена в файле `tnumber.c`. Определение обычно выполняется с помощью макросов `G_DEFINE_TYPE` или его семейства.
  - Определение экземпляра TNumber, единственным членом которого является только его родитель.
  - Объявление TNumberClass. Он должен быть определён позже в заголовочном файле.
  - Определение удобных макросов `T_NUMBER` (приведение к экземпляру), `T_NUMBER_CLASS` (приведение к классу), `T_IS_NUMBER` (проверка экземпляра), `T_IS_NUMBER_CLASS` (проверка класса) и `T_NUMBER_GET_CLASS`.
  - Поддержка `g_autoptr()`.
- 8-18: Определение структуры TNumberClass.
- 10-15: Это указатели на функции.
Они называются методами класса или виртуальными функциями.
Ожидается, что они будут переопределены в дочернем объекте.
Методы представляют собой пять арифметических операторов и функцию `to_s`.
Функция `to_s` похожа на функцию sprintf.
- 17: Указатель на обработчик сигнала "div-by-zero" по умолчанию.
Смещение этого указателя передаётся в `g_signal_new` в качестве аргумента.
- 22-38: Функции. Они являются публичными.

`tnumber.c` выглядит следующим образом.

~~~C
 1 #include "tnumber.h"
 2 
 3 static guint t_number_signal;
 4 
 5 G_DEFINE_ABSTRACT_TYPE (TNumber, t_number, G_TYPE_OBJECT)
 6 
 7 static void
 8 div_by_zero_default_cb (TNumber *self) {
 9   g_printerr ("\nError: division by zero.\n\n");
10 }
11 
12 static void
13 t_number_class_init (TNumberClass *class) {
14 
15   /* virtual functions */
16   class->add = NULL;
17   class->sub = NULL;
18   class->mul = NULL;
19   class->div = NULL;
20   class->uminus = NULL;
21   class->to_s = NULL;
22   /* default signal handler */
23   class->div_by_zero = div_by_zero_default_cb;
24   /* signal */
25   t_number_signal =
26   g_signal_new ("div-by-zero",
27                 G_TYPE_FROM_CLASS (class),
28                 G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
29                 G_STRUCT_OFFSET (TNumberClass, div_by_zero),
30                 NULL /* accumulator */,
31                 NULL /* accumulator data */,
32                 NULL /* C marshaller */,
33                 G_TYPE_NONE /* return_type */,
34                 0     /* n_params */
35                 );
36 }
37 
38 static void
39 t_number_init (TNumber *self) {
40 }
41 
42 TNumber *
43 t_number_add (TNumber *self, TNumber *other) {
44   g_return_val_if_fail (T_IS_NUMBER (self), NULL);
45   g_return_val_if_fail (T_IS_NUMBER (other), NULL);
46 
47   TNumberClass *class = T_NUMBER_GET_CLASS(self);
48 
49   return class->add ? class->add (self, other) : NULL;
50 }
51 
52 TNumber *
53 t_number_sub (TNumber *self, TNumber *other) {
54   g_return_val_if_fail (T_IS_NUMBER (self), NULL);
55   g_return_val_if_fail (T_IS_NUMBER (other), NULL);
56 
57   TNumberClass *class = T_NUMBER_GET_CLASS(self);
58 
59   return class->sub ? class->sub (self, other) : NULL;
60 }
61 
62 TNumber *
63 t_number_mul (TNumber *self, TNumber *other) {
64   g_return_val_if_fail (T_IS_NUMBER (self), NULL);
65   g_return_val_if_fail (T_IS_NUMBER (other), NULL);
66 
67   TNumberClass *class = T_NUMBER_GET_CLASS(self);
68 
69   return class->mul ? class->mul (self, other) : NULL;
70 }
71 
72 TNumber *
73 t_number_div (TNumber *self, TNumber *other) {
74   g_return_val_if_fail (T_IS_NUMBER (self), NULL);
75   g_return_val_if_fail (T_IS_NUMBER (other), NULL);
76 
77   TNumberClass *class = T_NUMBER_GET_CLASS(self);
78 
79   return class->div ? class->div (self, other) : NULL;
80 }
81 
82 TNumber *
83 t_number_uminus (TNumber *self) {
84   g_return_val_if_fail (T_IS_NUMBER (self), NULL);
85 
86   TNumberClass *class = T_NUMBER_GET_CLASS(self);
87 
88   return class->uminus ? class->uminus (self) : NULL;
89 }
90 
91 char *
92 t_number_to_s (TNumber *self) {
93   g_return_val_if_fail (T_IS_NUMBER (self), NULL);
94 
95   TNumberClass *class = T_NUMBER_GET_CLASS(self);
96 
97   return class->to_s ? class->to_s (self) : NULL;
98 }
~~~

- 5: Макрос `G_DEFINE_ABSTRACT_TYPE`.
Этот макрос используется для определения абстрактного типа объекта.
Абстрактный тип не создаётся как экземпляр.
Этот макрос раскрывается в:
  - Объявление функции `t_number_init ()`.
  - Объявление функции `t_number_class_init ()`.
  - Определение функции `t_number_get_type ()`.
  - Определение статической переменной `t_number_parent_class`, которая указывает на родительский класс.
- 3, 7-10, 26-35: Определяет сигнал деления на ноль.
Функция `div_by_zero_default_cb` является обработчиком сигнала "div-by-zero" по умолчанию.
Обработчик по умолчанию не имеет параметра пользовательских данных.
Используется функция `g_signal_new` вместо `g_signal_new_class_handler`.
Она указывает обработчик как смещение от начала класса до указателя на обработчик.
- 12-36: Функция инициализации класса `t_number_class_init`.
- 16-21: Эти методы класса являются виртуальными функциями.
Ожидается, что они будут переопределены в дочернем объекте TNumber.
Здесь присваивается NULL, так что ничего не произойдёт, когда эти методы будут вызваны.
- 23: Присваивает адрес функции `dev_by_zero_default_cb` переменной `class->div_by_zero`.
Это обработчик сигнала "div-by-zero" по умолчанию.
- 38-40: `t_number_init` — это функция инициализации экземпляра.
Но абстрактный объект не создаётся как экземпляр.
Поэтому в этой функции ничего не делается.
Но нельзя опускать определение этой функции.
- 42-98: Публичные функции.
Эти функции просто вызывают соответствующие методы класса, если указатель на метод класса не равен NULL.

## Объект TInt.

`tint.h` — это заголовочный файл класса TInt.
TInt является дочерним классом TNumber.

~~~C
 1 #pragma once
 2 
 3 #include <glib-object.h>
 4 
 5 #define T_TYPE_INT  (t_int_get_type ())
 6 G_DECLARE_FINAL_TYPE (TInt, t_int, T, INT, TNumber)
 7 
 8 /* create a new TInt instance */
 9 TInt *
10 t_int_new_with_value (int value);
11 
12 TInt *
13 t_int_new (void);
~~~

- 9-13: Объявляет публичные функции.
Арифметические функции и `to_s` объявлены в TNumber, поэтому TInt не объявляет эти функции.
Объявляются только функции создания экземпляров.

Файл C `tint.c` реализует виртуальные функции (методы класса).
И указатели методов в TNumberClass переписываются здесь.

~~~C
  1 #include "tnumber.h"
  2 #include "tint.h"
  3 #include "tdouble.h"
  4 
  5 #define PROP_INT 1
  6 static GParamSpec *int_property = NULL;
  7 
  8 struct _TInt {
  9   TNumber parent;
 10   int value;
 11 };
 12 
 13 G_DEFINE_TYPE (TInt, t_int, T_TYPE_NUMBER)
 14 
 15 static void
 16 t_int_set_property (GObject *object, guint property_id, const GValue *value, GParamSpec *pspec) {
 17   TInt *self = T_INT (object);
 18 
 19   if (property_id == PROP_INT)
 20     self->value = g_value_get_int (value);
 21   else
 22     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 23 }
 24 
 25 static void
 26 t_int_get_property (GObject *object, guint property_id, GValue *value, GParamSpec *pspec) {
 27   TInt *self = T_INT (object);
 28 
 29   if (property_id == PROP_INT)
 30     g_value_set_int (value, self->value);
 31   else
 32     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 33 }
 34 
 35 static void
 36 t_int_init (TInt *self) {
 37 }
 38 
 39 /* arithmetic operator */
 40 /* These operators create a new instance and return a pointer to it. */
 41 #define t_int_binary_op(op) \
 42   int i; \
 43   double d; \
 44   if (T_IS_INT (other)) { \
 45     g_object_get (T_INT (other), "value", &i, NULL); \
 46     return  T_NUMBER (t_int_new_with_value (T_INT(self)->value op i)); \
 47   } else { \
 48     g_object_get (T_DOUBLE (other), "value", &d, NULL); \
 49     return  T_NUMBER (t_int_new_with_value (T_INT(self)->value op (int) d)); \
 50   }
 51 
 52 static TNumber *
 53 t_int_add (TNumber *self, TNumber *other) {
 54   g_return_val_if_fail (T_IS_INT (self), NULL);
 55 
 56   t_int_binary_op (+)
 57 }
 58 
 59 static TNumber *
 60 t_int_sub (TNumber *self, TNumber *other) {
 61   g_return_val_if_fail (T_IS_INT (self), NULL);
 62 
 63   t_int_binary_op (-)
 64 }
 65 
 66 static TNumber *
 67 t_int_mul (TNumber *self, TNumber *other) {
 68   g_return_val_if_fail (T_IS_INT (self), NULL);
 69 
 70   t_int_binary_op (*)
 71 }
 72 
 73 static TNumber *
 74 t_int_div (TNumber *self, TNumber *other) {
 75   g_return_val_if_fail (T_IS_INT (self), NULL);
 76 
 77   int i;
 78   double d;
 79 
 80   if (T_IS_INT (other)) {
 81     g_object_get (T_INT (other), "value", &i, NULL);
 82     if (i == 0) {
 83       g_signal_emit_by_name (self, "div-by-zero");
 84       return NULL;
 85     } else
 86       return  T_NUMBER (t_int_new_with_value (T_INT(self)->value / i));
 87   } else {
 88     g_object_get (T_DOUBLE (other), "value", &d, NULL);
 89     if (d == 0) {
 90       g_signal_emit_by_name (self, "div-by-zero");
 91       return NULL;
 92     } else
 93       return  T_NUMBER (t_int_new_with_value (T_INT(self)->value / (int)  d));
 94   }
 95 }
 96 
 97 static TNumber *
 98 t_int_uminus (TNumber *self) {
 99   g_return_val_if_fail (T_IS_INT (self), NULL);
100 
101   return T_NUMBER (t_int_new_with_value (- T_INT(self)->value));
102 }
103 
104 static char *
105 t_int_to_s (TNumber *self) {
106   g_return_val_if_fail (T_IS_INT (self), NULL);
107 
108   int i;
109 
110   g_object_get (T_INT (self), "value", &i, NULL); 
111   return g_strdup_printf ("%d", i);
112 }
113 
114 static void
115 t_int_class_init (TIntClass *class) {
116   TNumberClass *tnumber_class = T_NUMBER_CLASS (class);
117   GObjectClass *gobject_class = G_OBJECT_CLASS (class);
118 
119   /* override virtual functions */
120   tnumber_class->add = t_int_add;
121   tnumber_class->sub = t_int_sub;
122   tnumber_class->mul = t_int_mul;
123   tnumber_class->div = t_int_div;
124   tnumber_class->uminus = t_int_uminus;
125   tnumber_class->to_s = t_int_to_s;
126 
127   gobject_class->set_property = t_int_set_property;
128   gobject_class->get_property = t_int_get_property;
129   int_property = g_param_spec_int ("value", "val", "Integer value", G_MININT, G_MAXINT, 0, G_PARAM_READWRITE);
130   g_object_class_install_property (gobject_class, PROP_INT, int_property);
131 }
132 
133 TInt *
134 t_int_new_with_value (int value) {
135   TInt *i;
136 
137   i = g_object_new (T_TYPE_INT, "value", value, NULL);
138   return i;
139 }
140 
141 TInt *
142 t_int_new (void) {
143   TInt *i;
144 
145   i = g_object_new (T_TYPE_INT, NULL);
146   return i;
147 }
~~~

- 5-6, 15-33, 127-130: Определение свойства "value".
Это то же самое, что и раньше.
- 8-11: Определение структуры TInt.
Это должно быть определено перед `G_DEFINE_TYPE`.
- 13: Макрос `G_DEFINE_TYPE`.
Этот макрос раскрывается в:
  - Объявление функции `t_int_init ()`.
  - Определение функции `t_int_get_type ()`.
  - Определение статической переменной `t_int_parent_class`, которая указывает на родительский класс.
- 35-37: `t_int_init`.
- 41-112: Эти функции подключаются к указателям методов класса в TIntClass.
Они являются реализацией виртуальных функций, определённых в `tnumber.c`.
- 41-50: Определяет макрос, используемый в `t_int_add`, `t_int_sub` и `t_int_mul`.
Этот макрос похож на функцию `t_int_div`.
Обратитесь к объяснению ниже для `t_int_div`.
- 52-71: Функции `t_int_add`, `t_int_sub` и `t_int_mul`.
Используется макрос `t_int_binary_op`.
- 73-95: Функция `t_int_div`.
Первый аргумент `self` — это объект, для которого вызывается функция.
Второй аргумент `other` — это другой объект TNumber.
Это может быть TInt или TDouble.
Если это TDouble, его значение приводится к int перед выполнением деления.
Если делитель (`other`) равен нулю, испускается сигнал "div-by-zero".
Сигнал определён в TNumber, поэтому TInt не знает идентификатор сигнала.
Поэтому испускание выполняется с помощью `g_signal_emit_by_name` вместо `g_signal_emit`.
Возвращаемое значение `t_int_div` имеет тип объекта TNumber
Однако, поскольку TNumber абстрактный, фактический тип объекта — TInt.
- 97-102: Функция для оператора унарного минуса.
- 104-112: Функция `to_s`. Эта функция преобразует int в строку.
Например, если значение объекта равно 123, то результатом будет строка "123".
Вызывающая сторона должна освободить строку, когда она станет ненужной.
- 114- 131: Функция инициализации класса `t_int_class_init`.
- 120-125: Методы класса переопределяются.
Например, если `t_number_add` вызывается для объекта TInt, то функция вызывает метод класса `*tnumber_class->add`.
Указатель указывает на функцию `t_int_add`.
Поэтому в итоге вызывается `t_int_add`.
- 133-147: Функции создания экземпляров такие же, как раньше.

## Объект TDouble.

Объект TDouble определён в файлах `tdouble.h` и `tdouble.c`.
Определение очень похоже на TInt.
Поэтому в этом подразделе просто показывается содержимое файлов.

tdouble.h

~~~C
 1 #pragma once
 2 
 3 #include <glib-object.h>
 4 
 5 #define T_TYPE_DOUBLE  (t_double_get_type ())
 6 G_DECLARE_FINAL_TYPE (TDouble, t_double, T, DOUBLE, TNumber)
 7 
 8 /* create a new TDouble instance */
 9 TDouble *
10 t_double_new_with_value (double value);
11 
12 TDouble *
13 t_double_new (void);
~~~

tdouble.c

~~~C
  1 #include "tnumber.h"
  2 #include "tdouble.h"
  3 #include "tint.h"
  4 
  5 #define PROP_DOUBLE 1
  6 static GParamSpec *double_property = NULL;
  7 
  8 struct _TDouble {
  9   TNumber parent;
 10   double value;
 11 };
 12 
 13 G_DEFINE_TYPE (TDouble, t_double, T_TYPE_NUMBER)
 14 
 15 static void
 16 t_double_set_property (GObject *object, guint property_id, const GValue *value, GParamSpec *pspec) {
 17   TDouble *self = T_DOUBLE (object);
 18   if (property_id == PROP_DOUBLE) {
 19     self->value = g_value_get_double (value);
 20   } else
 21     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 22 }
 23 
 24 static void
 25 t_double_get_property (GObject *object, guint property_id, GValue *value, GParamSpec *pspec) {
 26   TDouble *self = T_DOUBLE (object);
 27 
 28   if (property_id == PROP_DOUBLE)
 29     g_value_set_double (value, self->value);
 30   else
 31     G_OBJECT_WARN_INVALID_PROPERTY_ID (object, property_id, pspec);
 32 }
 33 
 34 static void
 35 t_double_init (TDouble *self) {
 36 }
 37 
 38 /* arithmetic operator */
 39 /* These operators create a new instance and return a pointer to it. */
 40 #define t_double_binary_op(op) \
 41   int i; \
 42   double d; \
 43   if (T_IS_INT (other)) { \
 44     g_object_get (T_INT (other), "value", &i, NULL); \
 45     return  T_NUMBER (t_double_new_with_value (T_DOUBLE(self)->value op (double) i)); \
 46   } else { \
 47     g_object_get (T_DOUBLE (other), "value", &d, NULL); \
 48     return  T_NUMBER (t_double_new_with_value (T_DOUBLE(self)->value op d)); \
 49   }
 50 
 51 static TNumber *
 52 t_double_add (TNumber *self, TNumber *other) {
 53   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 54 
 55   t_double_binary_op (+)
 56 }
 57 
 58 static TNumber *
 59 t_double_sub (TNumber *self, TNumber *other) {
 60   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 61 
 62   t_double_binary_op (-)
 63 }
 64 
 65 static TNumber *
 66 t_double_mul (TNumber *self, TNumber *other) {
 67   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 68 
 69   t_double_binary_op (*)
 70 }
 71 
 72 static TNumber *
 73 t_double_div (TNumber *self, TNumber *other) {
 74   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 75 
 76   int i;
 77   double d;
 78 
 79   if (T_IS_INT (other)) {
 80     g_object_get (T_INT (other), "value", &i, NULL);
 81     if (i == 0) {
 82       g_signal_emit_by_name (self, "div-by-zero");
 83       return NULL;
 84     } else
 85       return  T_NUMBER (t_double_new_with_value (T_DOUBLE(self)->value / (double) i));
 86   } else {
 87     g_object_get (T_DOUBLE (other), "value", &d, NULL);
 88     if (d == 0) {
 89       g_signal_emit_by_name (self, "div-by-zero");
 90       return NULL;
 91     } else
 92       return  T_NUMBER (t_double_new_with_value (T_DOUBLE(self)->value / d));
 93   }
 94 }
 95 
 96 static TNumber *
 97 t_double_uminus (TNumber *self) {
 98   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
 99 
100   return T_NUMBER (t_double_new_with_value (- T_DOUBLE(self)->value));
101 }
102 
103 static char *
104 t_double_to_s (TNumber *self) {
105   g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
106 
107   double d;
108 
109   g_object_get (T_DOUBLE (self), "value", &d, NULL);
110   return g_strdup_printf ("%lf", d);
111 }
112 
113 static void
114 t_double_class_init (TDoubleClass *class) {
115   TNumberClass *tnumber_class = T_NUMBER_CLASS (class);
116   GObjectClass *gobject_class = G_OBJECT_CLASS (class);
117 
118   /* override virtual functions */
119   tnumber_class->add = t_double_add;
120   tnumber_class->sub = t_double_sub;
121   tnumber_class->mul = t_double_mul;
122   tnumber_class->div = t_double_div;
123   tnumber_class->uminus = t_double_uminus;
124   tnumber_class->to_s = t_double_to_s;
125 
126   gobject_class->set_property = t_double_set_property;
127   gobject_class->get_property = t_double_get_property;
128   double_property = g_param_spec_double ("value", "val", "Double value", -G_MAXDOUBLE, G_MAXDOUBLE, 0, G_PARAM_READWRITE);
129   g_object_class_install_property (gobject_class, PROP_DOUBLE, double_property);
130 }
131 
132 TDouble *
133 t_double_new_with_value (double value) {
134   TDouble *d;
135 
136   d = g_object_new (T_TYPE_DOUBLE, "value", value, NULL);
137   return d;
138 }
139 
140 TDouble *
141 t_double_new (void) {
142   TDouble *d;
143 
144   d = g_object_new (T_TYPE_DOUBLE, NULL);
145   return d;
146 }
~~~

## main.c

`main.c` — это простая программа для тестирования объектов.

~~~C
 1 #include <glib-object.h>
 2 #include "tnumber.h"
 3 #include "tint.h"
 4 #include "tdouble.h"
 5 
 6 static void
 7 notify_cb (GObject *gobject, GParamSpec *pspec, gpointer user_data) {
 8   const char *name;
 9   int i;
10   double d;
11 
12   name = g_param_spec_get_name (pspec);
13   if (T_IS_INT (gobject) && strcmp (name, "value") == 0) {
14     g_object_get (T_INT (gobject), "value", &i, NULL);
15     g_print ("Property \"%s\" is set to %d.\n", name, i);
16   } else if (T_IS_DOUBLE (gobject) && strcmp (name, "value") == 0) {
17     g_object_get (T_DOUBLE (gobject), "value", &d, NULL);
18     g_print ("Property \"%s\" is set to %lf.\n", name, d);
19   }
20 }
21 
22 int
23 main (int argc, char **argv) {
24   TNumber *i, *d, *num;
25   char *si, *sd, *snum;
26 
27   i = T_NUMBER (t_int_new ());
28   d = T_NUMBER (t_double_new ());
29 
30   g_signal_connect (G_OBJECT (i), "notify::value", G_CALLBACK (notify_cb), NULL);
31   g_signal_connect (G_OBJECT (d), "notify::value", G_CALLBACK (notify_cb), NULL);
32 
33   g_object_set (T_INT (i), "value", 100, NULL);
34   g_object_set (T_DOUBLE (d), "value", 12.345, NULL);
35 
36   num = t_number_add (i, d);
37 
38   si = t_number_to_s (i);
39   sd = t_number_to_s (d);
40   snum = t_number_to_s (num);
41 
42   g_print ("%s + %s is %s.\n", si, sd, snum);
43 
44   g_object_unref (num);
45   g_free (snum);
46 
47   num = t_number_add (d, i);
48   snum = t_number_to_s (num);
49 
50   g_print ("%s + %s is %s.\n", sd, si, snum);
51 
52   g_object_unref (num);
53   g_free (sd);
54   g_free (snum);
55 
56   g_object_set (T_DOUBLE (d), "value", 0.0, NULL);
57   sd = t_number_to_s (d);
58   if ((num = t_number_div(i, d)) != NULL) {
59     snum = t_number_to_s (num);
60     g_print ("%s / %s is %s.\n", si, sd, snum);
61     g_object_unref (num);
62     g_free (snum);
63   }
64 
65   g_object_unref (i);
66   g_object_unref (d);
67   g_free (si);
68   g_free (sd);
69 
70   return 0;
71 }
~~~

- 6-20: Обработчик "notify".
Этот обработчик обновлён для поддержки как TInt, так и TDouble.
- 22-71: Функция `main`.
- 30-31: Подключает сигналы notify для `i` (TInt) и `d` (TDouble).
- 33-34: Устанавливает свойства "value" для `i` и `d`.
- 36: Добавляет `d` к `i`.
Результатом является объект TInt.
- 47: Добавляет `i` к `d`.
Результатом является объект TDouble.
Сложение двух объектов TNumber не коммутативно, потому что тип результата будет различным, если два объекта поменять местами.
- 56-63: Тестирует сигнал деления на ноль.

## Компиляция и выполнение

Исходные файлы расположены в каталоге [src/tnumber](../src/tnumber).
Файл `meson.buld`, который управляет процессом компиляции, выглядит следующим образом.

~~~meson
1 project ('tnumber', 'c')
2 
3 gobjdep = dependency('gobject-2.0')
4 
5 sourcefiles = files('main.c', 'tnumber.c', 'tint.c', 'tdouble.c')
6 
7 executable('tnumber', sourcefiles, dependencies: gobjdep, install: false)
8 
~~~

Компиляция и выполнение выполняются обычным способом.

~~~
$ cd src/tnumber
$ meson setup _build
$ ninja -C _build
$ _build/tnumber
~~~

Затем на экране отображается следующее.

~~~
Property "value" is set to 100.
Property "value" is set to 12.345000.
100 + 12.345000 is 112.
12.345000 + 100 is 112.345000.
Property "value" is set to 0.000000.

Error: division by zero.
~~~

Два ответа различаются из-за разных типов.

В этом разделе был показан простой пример наследуемого и абстрактного класса.
Вы можете определить свой наследуемый объект подобным образом.
Если ваш объект не является абстрактным, используйте `G_DEFINE_TYPE` вместо `G_DEFINE_ABSTRACT_TYPE`.
И вам нужна ещё одна вещь — как управлять приватными данными в вашем наследуемом объекте.
Есть руководство в [Справочнике API GObject](https://docs.gtk.org/gobject/tutorial.html#gobject-tutorial).
Смотрите руководство для изучения наследуемых объектов.

Также полезно посмотреть исходные файлы в GTK.

## Процесс инициализации класса

### Процесс инициализации TNumberClass

Поскольку TNumber является абстрактным объектом, вы не можете создать его экземпляр напрямую.
И вы также не можете создать класс TNumber.
Но когда вы создаёте экземпляр его потомка, класс TNumber создаётся и инициализируется.
Первый вызов `g_object_new (T_TYPE_INT, ...)` или `g_object_new (T_TYPE_DOUBLE, ...)` создаёт и инициализирует TNumberClass, если класс не существует.
После этого создаётся TIntClass или TDoubleClass, за которым следует создание экземпляра TInt или TDouble соответственно.

И процесс инициализации для класса TNumber выглядит следующим образом.

1. GObjectClass был инициализирован до того, как функция `main` начинает работу.
2. Память выделяется для TNumberClass.
3. Родительская (GObjectClass) часть класса копируется из GObjectClass.
4. Вызывается функция инициализации класса `t_number_class_init`.
Она инициализирует методы класса (указатели на методы класса) и обработчик сигнала по умолчанию.

Диаграмма ниже показывает процесс.

![TNumberClass initialization](../image/tnumberclass_init.png)

### Процесс инициализации TIntClass

1. TNumberClass был инициализирован до начала инициализации TIntClass.
2. Первый вызов `g_object_new (T_TYPE_INT, ...)` инициализирует TIntClass.
И процесс инициализации выглядит следующим образом.
3. Память выделяется для TIntClass.
TIntClass не имеет собственной области.
Поэтому его структура такая же, как у родительского класса (TNumberClass).
4. Родительская (TNumberClass) часть класса (это то же самое, что и весь TIntClass) копируется из TNumberClass.
5. Вызывается функция инициализации класса `t_int_class_init`.
Она переопределяет методы класса из TNumber, `set_property` и `get_property`.

Диаграмма ниже показывает процесс.

![TIntClass initialization](../image/tintclass_init.png)

Вверх: [Readme.md](../Readme.md),  Пред: [Раздел 5](sec5.md), След: [Раздел 7](sec7.md)
