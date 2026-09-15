#python #pandas #визуализация #аналитика #backend #собеседования

# 🐍 Python для аналитика — конспект для собеседований (RU)

> Конспект по Python, Pandas и визуализации данных с прицелом на задачи Data/BI/Product Analyst: от базового синтаксиса до сводных таблиц, графиков и продуктовых метрик.

## Содержание

- [[#Условия и циклы if for while]]
- [[#Функции]]
- [[#Списки List]]
- [[#Словари Dict]]
- [[#Множества Set и кортежи Tuple]]
- [[#Lambda-функции]]
- [[#List Dict Set comprehension]]
- [[#Pandas — основы]]
- [[#Чтение данных read_csv и др]]
- [[#Индексация и фильтрация в Pandas]]
- [[#merge join concat]]
- [[#groupby]]
- [[#pivot_table]]
- [[#Преобразования данных apply map astype]]
- [[#Работа с датами и пропусками]]
- [[#NumPy и базовая статистика]]
- [[#Визуализация matplotlib и seaborn]]
- [[#Продуктовые метрики и A B тесты]]
- [[#Частые вопросы и задачи на собеседовании]]

---

## Условия и циклы (if / for / while)

> [!definition] Определение `if/elif/else` — управление ветвлением; `for` — цикл по итерируемому объекту; `while` — цикл, пока условие истинно.

> [!example] Пример
> 
> ```python
> # if / elif / else
> score = 82
> if score >= 90:
>     grade = 'A'
> elif score >= 75:
>     grade = 'B'
> else:
>     grade = 'C'
> 
> # for
> for i in range(5):
>     print(i)          # 0 1 2 3 4
> 
> for name in ['Аня', 'Борис', 'Вика']:
>     print(name)
> 
> # while
> n = 5
> while n > 0:
>     print(n)
>     n -= 1
> 
> # break / continue / else у цикла
> for x in range(10):
>     if x == 3:
>         continue      # пропустить итерацию
>     if x == 7:
>         break         # прервать цикл
>     print(x)
> ```

> [!warning] Частая ошибка Изменение списка внутри `for`, по которому итерируемся (например, `list.remove()` в цикле `for item in list`) — часть элементов "проскакивает", т.к. индексы сдвигаются. Для безопасного удаления — итерируйтесь по копии (`list[:]`) или используйте list comprehension.

---

## Функции

> [!definition] Определение Функция — именованный блок кода, принимающий аргументы и (опционально) возвращающий значение через `return`.

> [!example] Пример
> 
> ```python
> def greet(name, greeting='Привет'):
>     """Docstring: приветствие пользователя"""
>     return f"{greeting}, {name}!"
> 
> print(greet('Аня'))                  # Привет, Аня!
> print(greet('Борис', 'Здравствуй'))  # Здравствуй, Борис!
> 
> # *args и **kwargs
> def summarize(*args, **kwargs):
>     print(args)     # кортеж позиционных аргументов
>     print(kwargs)   # словарь именованных аргументов
> 
> summarize(1, 2, 3, name='Аня', age=25)
> 
> # аннотации типов (не влияют на выполнение, но полезны для читаемости)
> def add(a: int, b: int) -> int:
>     return a + b
> ```

> [!important] Ключевой вывод Изменяемые объекты (списки, словари) как значения по умолчанию — классическая ловушка: значение по умолчанию создаётся **один раз** при определении функции и переиспользуется между вызовами.
> 
> ```python
> def bad(items=[]):     # ❌ опасно
>     items.append(1)
>     return items
> 
> def good(items=None):  # ✅ правильно
>     if items is None:
>         items = []
>     items.append(1)
>     return items
> ```

---

## Списки (List)

> [!definition] Определение `list` — упорядоченная изменяемая коллекция элементов, допускает дубликаты и разные типы данных.

> [!example] Пример
> 
> ```python
> nums = [3, 1, 4, 1, 5, 9]
> nums.append(2)          # добавить в конец
> nums.insert(0, 100)      # вставить по индексу
> nums.remove(1)           # удалить первое вхождение значения 1
> nums.sort()               # сортировка на месте
> nums_sorted = sorted(nums, reverse=True)  # новая отсортированная копия
> nums[1:4]                 # срез
> len(nums)                 # длина
> sum(nums), max(nums), min(nums)
> 
> # частая задача: найти дубликаты
> from collections import Counter
> counts = Counter(nums)
> duplicates = [item for item, cnt in counts.items() if cnt > 1]
> ```

---

## Словари (Dict)

> [!definition] Определение `dict` — неупорядоченная (с Python 3.7 — сохраняет порядок вставки) изменяемая коллекция пар "ключ-значение". Ключи уникальны и должны быть хешируемыми.

> [!example] Пример
> 
> ```python
> user = {'name': 'Аня', 'age': 25, 'city': 'Москва'}
> user['email'] = 'anya@mail.com'   # добавить ключ
> user.get('phone', 'нет данных')    # безопасное чтение с дефолтом
> user.pop('city')                    # удалить ключ
> 
> for key, value in user.items():
>     print(key, value)
> 
> # частая задача: подсчёт частот слов
> from collections import defaultdict
> word_count = defaultdict(int)
> for word in 'a b a c b a'.split():
>     word_count[word] += 1
> # {'a': 3, 'b': 2, 'c': 1}
> ```

---

## Множества (Set) и кортежи (Tuple)

> [!definition] Определение `set` — неупорядоченная коллекция **уникальных** элементов, поддерживает операции теории множеств. `tuple` — упорядоченная **неизменяемая** коллекция.

> [!example] Пример
> 
> ```python
> a = {1, 2, 3}
> b = {2, 3, 4}
> a | b   # объединение -> {1, 2, 3, 4}
> a & b   # пересечение -> {2, 3}
> a - b   # разность -> {1}
> a ^ b   # симметричная разность -> {1, 4}
> 
> # удаление дубликатов из списка
> unique_items = list(set([1, 2, 2, 3, 3, 3]))
> 
> point = (10, 20)   # tuple — нельзя изменить point[0] = 5
> x, y = point         # распаковка
> ```

---

## Lambda-функции

> [!definition] Определение `lambda` — анонимная функция из одного выражения. Используется там, где полноценная функция избыточна (сортировка, `map`, `filter`, `apply` в Pandas).

> [!example] Пример
> 
> ```python
> square = lambda x: x ** 2
> square(5)  # 25
> 
> # сортировка по кастомному ключу
> people = [('Аня', 25), ('Борис', 20), ('Вика', 30)]
> sorted(people, key=lambda p: p[1])   # сортировка по возрасту
> 
> # map / filter
> nums = [1, 2, 3, 4, 5]
> squared = list(map(lambda x: x ** 2, nums))
> evens = list(filter(lambda x: x % 2 == 0, nums))
> ```

---

## List / Dict / Set comprehension

> [!definition] Определение Comprehension — компактный синтаксис для создания коллекции на основе итерации и (опционально) условия. Обычно быстрее и читабельнее эквивалентного цикла `for` с `.append()`.

> [!formula] Синтаксис
> 
> ```python
> [выражение for элемент in итерируемое if условие]
> {ключ: значение for элемент in итерируемое}
> {выражение for элемент in итерируемое}
> ```

> [!example] Пример
> 
> ```python
> # list comprehension
> squares = [x**2 for x in range(10)]
> evens = [x for x in range(20) if x % 2 == 0]
> 
> # вложенный comprehension (матрица -> плоский список)
> matrix = [[1, 2], [3, 4], [5, 6]]
> flat = [x for row in matrix for x in row]
> 
> # dict comprehension
> squares_dict = {x: x**2 for x in range(5)}
> 
> # set comprehension
> unique_lengths = {len(word) for word in ['ab', 'abc', 'de']}
> ```

> [!warning] Частая ошибка Слишком сложные, многоуровневые comprehension'ы ухудшают читаемость — если внутри 2+ вложенных `for`/`if`, лучше вернуться к обычному циклу.

---

## Pandas — основы

> [!definition] Определение **Pandas** — библиотека для работы с табличными данными. Базовые структуры: `Series` (одномерный массив с индексом) и `DataFrame` (двумерная таблица, как в Excel/SQL).

> [!example] Пример
> 
> ```python
> import pandas as pd
> 
> df = pd.DataFrame({
>     'name': ['Аня', 'Борис', 'Вика'],
>     'age': [25, 30, 22],
>     'city': ['Москва', 'СПб', 'Казань']
> })
> 
> df.head()        # первые 5 строк
> df.info()         # типы данных, кол-во непустых значений
> df.describe()     # статистика по числовым столбцам
> df.shape           # (строки, столбцы)
> df.columns         # список столбцов
> df.dtypes          # типы данных по столбцам
> ```

---

## Чтение данных (read_csv и др.)

> [!example] Пример
> 
> ```python
> df = pd.read_csv('data.csv',
>                   sep=',',
>                   encoding='utf-8',
>                   parse_dates=['order_date'],
>                   dtype={'customer_id': 'int32'})
> 
> pd.read_excel('data.xlsx', sheet_name='Sheet1')
> pd.read_json('data.json')
> pd.read_sql('SELECT * FROM orders', con=connection)
> 
> df.to_csv('output.csv', index=False)
> df.to_excel('output.xlsx', index=False)
> ```

> [!warning] Частая ошибка Забывают `index=False` при сохранении в CSV/Excel — в результате появляется лишний столбец с индексом `Unnamed: 0` при повторном чтении файла.

---

## Индексация и фильтрация в Pandas

> [!formula] loc vs iloc
> 
> - `.loc[]` — доступ по **меткам** (label-based): именам индекса и столбцов
> - `.iloc[]` — доступ по **позиции** (integer-based): числовым индексам, как в обычном списке

> [!example] Пример
> 
> ```python
> df.loc[0, 'name']            # значение по метке
> df.loc[df['age'] > 25]        # фильтрация по условию
> df.iloc[0:2, 1:3]              # срез по позициям
> 
> # фильтрация с несколькими условиями (обязательны скобки!)
> df[(df['age'] > 20) & (df['city'] == 'Москва')]
> df[(df['age'] < 20) | (df['city'] == 'СПб')]
> 
> df.query('age > 20 and city == "Москва"')   # альтернативный синтаксис
> df[df['city'].isin(['Москва', 'СПб'])]
> df[~df['city'].isin(['Москва'])]              # отрицание (NOT)
> ```

> [!warning] Частая ошибка Использование `and`/`or` вместо `&`/`|` при фильтрации DataFrame — вызовет ошибку `ValueError: truth value of a Series is ambiguous`, потому что операторы `and`/`or` не векторизуются pandas.

---

## merge / join / concat

> [!definition] Определение `merge()` — аналог SQL `JOIN`, объединяет DataFrame'ы по ключевым столбцам. `concat()` — просто "склеивает" DataFrame'ы по строкам или столбцам без ключа соединения.

> [!formula] Типы merge (как SQL JOIN) `how='inner' | 'left' | 'right' | 'outer'`

> [!example] Пример
> 
> ```python
> orders = pd.DataFrame({'order_id': [1, 2, 3], 'customer_id': [1, 2, 4]})
> customers = pd.DataFrame({'customer_id': [1, 2, 3], 'name': ['Аня', 'Борис', 'Вика']})
> 
> pd.merge(orders, customers, on='customer_id', how='inner')
> pd.merge(orders, customers, on='customer_id', how='left')
> pd.merge(orders, customers,
>           left_on='customer_id', right_on='customer_id',
>           how='outer', indicator=True)   # indicator=True покажет источник строки
> 
> # concat: объединение по строкам (одинаковые столбцы)
> pd.concat([df_2023, df_2024], axis=0, ignore_index=True)
> # concat: объединение по столбцам
> pd.concat([df1, df2], axis=1)
> ```

> [!important] Ключевой вывод После `merge` с `how='left'` полезно сразу проверить, не увеличилось ли число строк (дублирование из-за неуникальности ключа справа) — `assert len(result) == len(orders)`, если ожидается связь "один к одному".

---

## groupby

> [!definition] Определение `groupby()` — аналог SQL `GROUP BY`: группирует строки по значению столбца(ов) и позволяет применить агрегатные функции к каждой группе.

> [!example] Пример
> 
> ```python
> df.groupby('city')['age'].mean()
> df.groupby('city').agg(
>     avg_age=('age', 'mean'),
>     total=('age', 'count'),
>     max_age=('age', 'max')
> )
> 
> # группировка по нескольким столбцам
> df.groupby(['city', 'gender'])['salary'].sum()
> 
> # своя функция через apply
> df.groupby('city')['salary'].apply(lambda x: x.max() - x.min())
> 
> # transform: вернуть агрегат в исходной форме (для вычисления доли от группы)
> df['city_avg_salary'] = df.groupby('city')['salary'].transform('mean')
> ```

---

## pivot_table

> [!definition] Определение `pivot_table()` — сводная таблица (как в Excel): группирует данные по строкам/столбцам и агрегирует значения на пересечении.

> [!example] Пример
> 
> ```python
> pd.pivot_table(
>     df,
>     values='sales',
>     index='region',
>     columns='month',
>     aggfunc='sum',
>     fill_value=0,
>     margins=True   # добавить итоговую строку/столбец "Все"
> )
> 
> # pivot vs pivot_table: pivot() не агрегирует и требует уникальности пар (index, columns)
> df.pivot(index='date', columns='product', values='price')
> ```

> [!warning] Частая ошибка Использование `pivot()` вместо `pivot_table()`, когда есть дубликаты в комбинации index/columns — `pivot()` выбросит `ValueError`, а `pivot_table()` автоматически агрегирует их (по умолчанию через `mean`).

---

## Преобразования данных (apply, map, astype)

> [!example] Пример
> 
> ```python
> df['age_group'] = df['age'].apply(lambda x: 'взрослый' if x >= 18 else 'ребёнок')
> df['city_upper'] = df['city'].map(str.upper)     # map — для Series
> df['salary'] = df['salary'].astype(float)          # смена типа
> 
> df.rename(columns={'name': 'full_name'}, inplace=True)
> df.sort_values('age', ascending=False)
> df.drop(columns=['city'])
> df.drop_duplicates(subset=['customer_id'])
> df['category'] = df['category'].replace({'old': 'new'})
> ```

> [!important] Ключевой вывод `apply()` для DataFrame построчно (`axis=1`) работает **значительно медленнее** векторизованных операций pandas/NumPy. Правило: если операцию можно выразить через встроенные векторные методы (`df['a'] + df['b']`, `np.where`) — используйте их вместо `apply`.

---

## Работа с датами и пропусками

> [!example] Пример
> 
> ```python
> df['order_date'] = pd.to_datetime(df['order_date'])
> df['year'] = df['order_date'].dt.year
> df['month'] = df['order_date'].dt.month
> df['weekday'] = df['order_date'].dt.day_name()
> 
> # пропуски (NaN)
> df.isna().sum()                        # кол-во пропусков по столбцам
> df.dropna(subset=['age'])                # удалить строки с NaN в столбце
> df['age'].fillna(df['age'].median())     # заполнить медианой
> df.fillna({'city': 'Неизвестно', 'age': 0})
> ```

---

## NumPy и базовая статистика

> [!definition] Определение **NumPy** — библиотека для быстрых векторных вычислений над массивами (`ndarray`), основа Pandas и большинства библиотек анализа данных.

> [!example] Пример
> 
> ```python
> import numpy as np
> 
> arr = np.array([1, 2, 3, 4, 5])
> arr.mean(), arr.std(), np.median(arr)
> np.where(arr > 3, 'high', 'low')    # векторное условие
> 
> # корреляция между столбцами
> df[['age', 'salary']].corr()
> 
> # базовые статистики
> df['salary'].describe()   # count, mean, std, min, 25%, 50%, 75%, max
> ```

> [!important] Ключевой вывод Для BI/аналитика важно понимать разницу между **средним (mean)** и **медианой (median)**: среднее чувствительно к выбросам, медиана — устойчива. Для skewed-распределений (например, зарплаты, чек) медиана часто более репрезентативна.

---

## Визуализация: matplotlib и seaborn

> [!definition] Определение **matplotlib** — базовая библиотека визуализации в Python (низкоуровневая, гибкая). **seaborn** — надстройка над matplotlib с удобным высокоуровневым синтаксисом и красивыми стилями по умолчанию, хорошо интегрирована с Pandas DataFrame.

> [!example] Пример: базовый matplotlib
> 
> ```python
> import matplotlib.pyplot as plt
> 
> plt.figure(figsize=(8, 5))
> plt.plot(df['month'], df['sales'])
> plt.title('Продажи по месяцам')
> plt.xlabel('Месяц')
> plt.ylabel('Продажи')
> plt.show()
> ```

> [!example] Пример: seaborn
> 
> ```python
> import seaborn as sns
> 
> # столбчатая диаграмма (сравнение категорий)
> sns.barplot(data=df, x='city', y='salary', estimator='mean')
> 
> # гистограмма (распределение числовой переменной)
> sns.histplot(data=df, x='age', bins=20, kde=True)
> 
> # scatter plot (связь двух числовых переменных)
> sns.scatterplot(data=df, x='age', y='salary', hue='city')
> 
> # heatmap (матрица корреляций)
> sns.heatmap(df[['age', 'salary', 'bonus']].corr(), annot=True, cmap='coolwarm')
> 
> # line plot (динамика во времени)
> sns.lineplot(data=df, x='month', y='sales', hue='region')
> 
> # boxplot (распределение + выбросы по категориям)
> sns.boxplot(data=df, x='city', y='salary')
> 
> plt.show()
> ```

> [!formula] Когда какой график использовать
> 
> |Задача|График|
> |---|---|
> |Сравнение категорий|`barplot`|
> |Распределение одной числовой переменной|`histplot`, `boxplot`|
> |Связь двух числовых переменных|`scatterplot`|
> |Динамика во времени|`lineplot`|
> |Корреляции между многими переменными|`heatmap`|
> |Распределение по категориям + выбросы|`boxplot`, `violinplot`|

> [!warning] Частая ошибка Строить `lineplot` для несортированных по оси X данных — линия будет "скакать" туда-сюда. Перед построением временного ряда обязательно `df.sort_values('date')`.

---

## Продуктовые метрики и A/B-тесты

> [!definition] Определение Специфичный для BI/Product Analyst блок: ключевые метрики продукта и статистическая проверка гипотез.

Основные метрики:

- **DAU/WAU/MAU** — Daily/Weekly/Monthly Active Users
- **Retention Rate** — доля пользователей, вернувшихся через N дней после первого визита
- **Churn Rate** — доля пользователей, переставших пользоваться продуктом
- **Conversion Rate** — доля пользователей, совершивших целевое действие
- **LTV (Lifetime Value)** — суммарная ценность клиента за всё время
- **ARPU/ARPPU** — средний доход на пользователя / на платящего пользователя
- **Funnel (воронка)** — последовательность шагов до целевого действия, с конверсией на каждом шаге

> [!example] Пример: когортный анализ retention в Pandas
> 
> ```python
> df['cohort'] = df.groupby('user_id')['event_date'].transform('min').dt.to_period('M')
> df['period'] = (df['event_date'].dt.to_period('M') - df['cohort']).apply(lambda x: x.n)
> 
> cohort_pivot = df.pivot_table(
>     index='cohort', columns='period', values='user_id', aggfunc='nunique'
> )
> retention = cohort_pivot.divide(cohort_pivot[0], axis=0)  # доля от размера когорты
> ```

> [!example] Пример: A/B-тест (t-test) через scipy
> 
> ```python
> from scipy import stats
> 
> group_a = df[df['group'] == 'control']['conversion']
> group_b = df[df['group'] == 'test']['conversion']
> 
> t_stat, p_value = stats.ttest_ind(group_a, group_b)
> if p_value < 0.05:
>     print('Статистически значимое различие')
> else:
>     print('Значимого различия не обнаружено')
> 
> # для долей (конверсий) — z-test пропорций
> from statsmodels.stats.proportion import proportions_ztest
> count = [120, 150]   # число успехов в каждой группе
> nobs = [1000, 1000]  # размер каждой группы
> z_stat, p_value = proportions_ztest(count, nobs)
> ```

> [!important] Ключевой вывод Перед тем как делать вывод из A/B-теста, всегда проверяйте: достаточен ли размер выборки (**power analysis**), нет ли **novelty/primacy effect**, не смотрели ли вы на результат "подглядыванием" много раз (peeking problem увеличивает false positive rate).

---

## Частые вопросы и задачи на собеседовании

> [!important] Шпаргалка вопросов
> 
> 1. Чем отличается список от кортежа? Когда использовать `set`?
> 2. Что такое list comprehension и чем он быстрее обычного цикла?
> 3. Разница между `.loc` и `.iloc`?
> 4. Как объединить два DataFrame — `merge` vs `concat` vs `join`?
> 5. Чем `pivot_table` отличается от `groupby`?
> 6. Как обработать пропущенные значения — когда `dropna`, а когда `fillna`?
> 7. Почему `apply(axis=1)` — плохая практика для больших данных?
> 8. Как посчитать retention/конверсию по когортам в Pandas?
> 9. Как выбрать тип графика под задачу (barplot vs histplot vs scatter)?
> 10. Классическая задача: «даны два списка, найти элементы, встречающиеся в обоих» → `set(a) & set(b)`.
> 11. Классическая задача: «посчитать частоту слов в тексте» → `Counter`.
> 12. Классическая задача: «перевернуть строку / проверить палиндром» → `s[::-1]`, `s == s[::-1]`.

---

## Связанные заметки

- [[SQL — конспект для собеседований]]
- [[Индексы]]
- [[A B тесты и аналитика]]
- [[NumPy и статистика]]
- [[Продуктовые метрики]]
- [[BI-инструменты Tableau Power BI]]

---

---

# 🐍 Python for Analysts — Interview Study Notes (EN)

#python #pandas #visualization #analytics #backend #interviews

> Study notes on Python, Pandas, and data visualization for Data/BI/Product Analyst interviews: from basic syntax to pivot tables, charts, and product metrics.

## Table of Contents

- [[#Conditionals and Loops if for while]]
- [[#Functions (EN)]]
- [[#Lists]]
- [[#Dictionaries]]
- [[#Sets and Tuples]]
- [[#Lambda Functions]]
- [[#List Dict Set Comprehensions]]
- [[#Pandas Basics]]
- [[#Reading Data read_csv and more]]
- [[#Indexing and Filtering in Pandas]]
- [[#merge join concat (EN)]]
- [[#groupby (EN)]]
- [[#pivot_table (EN)]]
- [[#Data Transformations apply map astype]]
- [[#Dates and Missing Values]]
- [[#NumPy and Basic Statistics]]
- [[#Visualization matplotlib and seaborn]]
- [[#Product Metrics and A B Testing]]
- [[#Common Interview Questions and Tasks]]

---

## Conditionals and Loops (if / for / while)

> [!definition] Definition `if/elif/else` controls branching; `for` iterates over an iterable; `while` loops as long as a condition holds.

> [!example] Example
> 
> ```python
> # if / elif / else
> score = 82
> if score >= 90:
>     grade = 'A'
> elif score >= 75:
>     grade = 'B'
> else:
>     grade = 'C'
> 
> # for
> for i in range(5):
>     print(i)          # 0 1 2 3 4
> 
> for name in ['Anna', 'Bob', 'Vika']:
>     print(name)
> 
> # while
> n = 5
> while n > 0:
>     print(n)
>     n -= 1
> 
> # break / continue / loop else
> for x in range(10):
>     if x == 3:
>         continue      # skip this iteration
>     if x == 7:
>         break         # exit the loop
>     print(x)
> ```

> [!warning] Common Mistake Modifying a list while iterating over it (e.g., calling `list.remove()` inside `for item in list`) — some elements get skipped because indices shift. Iterate over a copy (`list[:]`) or use a list comprehension for safe removal.

---

## Functions (EN)

> [!definition] Definition A function is a named block of code that accepts arguments and (optionally) returns a value via `return`.

> [!example] Example
> 
> ```python
> def greet(name, greeting='Hello'):
>     """Docstring: greets the user"""
>     return f"{greeting}, {name}!"
> 
> print(greet('Anna'))            # Hello, Anna!
> print(greet('Bob', 'Hi'))         # Hi, Bob!
> 
> # *args and **kwargs
> def summarize(*args, **kwargs):
>     print(args)     # tuple of positional args
>     print(kwargs)   # dict of keyword args
> 
> summarize(1, 2, 3, name='Anna', age=25)
> 
> # type hints (don't affect execution, but improve readability)
> def add(a: int, b: int) -> int:
>     return a + b
> ```

> [!important] Key Takeaway Mutable default arguments (lists, dicts) are a classic trap: the default value is created **once** at function definition time and reused across calls.
> 
> ```python
> def bad(items=[]):     # ❌ dangerous
>     items.append(1)
>     return items
> 
> def good(items=None):  # ✅ correct
>     if items is None:
>         items = []
>     items.append(1)
>     return items
> ```

---

## Lists

> [!definition] Definition A `list` is an ordered, mutable collection of elements; it allows duplicates and mixed data types.

> [!example] Example
> 
> ```python
> nums = [3, 1, 4, 1, 5, 9]
> nums.append(2)          # add to the end
> nums.insert(0, 100)      # insert at an index
> nums.remove(1)           # remove first occurrence of value 1
> nums.sort()               # sort in place
> nums_sorted = sorted(nums, reverse=True)  # new sorted copy
> nums[1:4]                 # slice
> len(nums)                 # length
> sum(nums), max(nums), min(nums)
> 
> # common task: find duplicates
> from collections import Counter
> counts = Counter(nums)
> duplicates = [item for item, cnt in counts.items() if cnt > 1]
> ```

---

## Dictionaries

> [!definition] Definition A `dict` is an (insertion-ordered since Python 3.7) mutable collection of key-value pairs. Keys must be unique and hashable.

> [!example] Example
> 
> ```python
> user = {'name': 'Anna', 'age': 25, 'city': 'Moscow'}
> user['email'] = 'anna@mail.com'    # add a key
> user.get('phone', 'not provided')    # safe read with default
> user.pop('city')                      # remove a key
> 
> for key, value in user.items():
>     print(key, value)
> 
> # common task: word frequency count
> from collections import defaultdict
> word_count = defaultdict(int)
> for word in 'a b a c b a'.split():
>     word_count[word] += 1
> # {'a': 3, 'b': 2, 'c': 1}
> ```

---

## Sets and Tuples

> [!definition] Definition A `set` is an unordered collection of **unique** elements supporting set-theory operations. A `tuple` is an ordered, **immutable** collection.

> [!example] Example
> 
> ```python
> a = {1, 2, 3}
> b = {2, 3, 4}
> a | b   # union -> {1, 2, 3, 4}
> a & b   # intersection -> {2, 3}
> a - b   # difference -> {1}
> a ^ b   # symmetric difference -> {1, 4}
> 
> # removing duplicates from a list
> unique_items = list(set([1, 2, 2, 3, 3, 3]))
> 
> point = (10, 20)   # tuple — you can't do point[0] = 5
> x, y = point         # unpacking
> ```

---

## Lambda Functions

> [!definition] Definition `lambda` creates an anonymous, single-expression function. Useful where a full `def` would be overkill (sorting, `map`, `filter`, `apply` in Pandas).

> [!example] Example
> 
> ```python
> square = lambda x: x ** 2
> square(5)  # 25
> 
> # sort by a custom key
> people = [('Anna', 25), ('Bob', 20), ('Vika', 30)]
> sorted(people, key=lambda p: p[1])   # sort by age
> 
> # map / filter
> nums = [1, 2, 3, 4, 5]
> squared = list(map(lambda x: x ** 2, nums))
> evens = list(filter(lambda x: x % 2 == 0, nums))
> ```

---

## List / Dict / Set Comprehensions

> [!definition] Definition A comprehension is compact syntax for building a collection from iteration plus an optional condition. Usually faster and more readable than an equivalent `for` loop with `.append()`.

> [!formula] Syntax
> 
> ```python
> [expression for item in iterable if condition]
> {key: value for item in iterable}
> {expression for item in iterable}
> ```

> [!example] Example
> 
> ```python
> # list comprehension
> squares = [x**2 for x in range(10)]
> evens = [x for x in range(20) if x % 2 == 0]
> 
> # nested comprehension (matrix -> flat list)
> matrix = [[1, 2], [3, 4], [5, 6]]
> flat = [x for row in matrix for x in row]
> 
> # dict comprehension
> squares_dict = {x: x**2 for x in range(5)}
> 
> # set comprehension
> unique_lengths = {len(word) for word in ['ab', 'abc', 'de']}
> ```

> [!warning] Common Mistake Overly complex, multi-level comprehensions hurt readability — if you need 2+ nested `for`/`if` clauses, switch back to a regular loop.

---

## Pandas Basics

> [!definition] Definition **Pandas** is a library for working with tabular data. Core structures: `Series` (1-D indexed array) and `DataFrame` (2-D table, like Excel/SQL).

> [!example] Example
> 
> ```python
> import pandas as pd
> 
> df = pd.DataFrame({
>     'name': ['Anna', 'Bob', 'Vika'],
>     'age': [25, 30, 22],
>     'city': ['Moscow', 'SPb', 'Kazan']
> })
> 
> df.head()        # first 5 rows
> df.info()         # data types, non-null counts
> df.describe()     # summary stats for numeric columns
> df.shape           # (rows, columns)
> df.columns         # list of column names
> df.dtypes          # per-column data types
> ```

---

## Reading Data (read_csv and more)

> [!example] Example
> 
> ```python
> df = pd.read_csv('data.csv',
>                   sep=',',
>                   encoding='utf-8',
>                   parse_dates=['order_date'],
>                   dtype={'customer_id': 'int32'})
> 
> pd.read_excel('data.xlsx', sheet_name='Sheet1')
> pd.read_json('data.json')
> pd.read_sql('SELECT * FROM orders', con=connection)
> 
> df.to_csv('output.csv', index=False)
> df.to_excel('output.xlsx', index=False)
> ```

> [!warning] Common Mistake Forgetting `index=False` when saving to CSV/Excel — this adds a stray `Unnamed: 0` index column when the file is read back in.

---

## Indexing and Filtering in Pandas

> [!formula] loc vs iloc
> 
> - `.loc[]` — **label-based** access: index and column names
> - `.iloc[]` — **position-based** access: integer positions, like a plain list

> [!example] Example
> 
> ```python
> df.loc[0, 'name']            # value by label
> df.loc[df['age'] > 25]        # boolean filtering
> df.iloc[0:2, 1:3]              # positional slice
> 
> # filtering with multiple conditions (parentheses required!)
> df[(df['age'] > 20) & (df['city'] == 'Moscow')]
> df[(df['age'] < 20) | (df['city'] == 'SPb')]
> 
> df.query('age > 20 and city == "Moscow"')   # alternative syntax
> df[df['city'].isin(['Moscow', 'SPb'])]
> df[~df['city'].isin(['Moscow'])]              # negation (NOT)
> ```

> [!warning] Common Mistake Using `and`/`or` instead of `&`/`|` when filtering a DataFrame raises `ValueError: truth value of a Series is ambiguous`, since Python's `and`/`or` don't vectorize over pandas Series.

---

## merge / join / concat (EN)

> [!definition] Definition `merge()` is the pandas equivalent of a SQL `JOIN` — it combines DataFrames on key columns. `concat()` simply stacks DataFrames along rows or columns without a join key.

> [!formula] merge types (like SQL JOIN) `how='inner' | 'left' | 'right' | 'outer'`

> [!example] Example
> 
> ```python
> orders = pd.DataFrame({'order_id': [1, 2, 3], 'customer_id': [1, 2, 4]})
> customers = pd.DataFrame({'customer_id': [1, 2, 3], 'name': ['Anna', 'Bob', 'Vika']})
> 
> pd.merge(orders, customers, on='customer_id', how='inner')
> pd.merge(orders, customers, on='customer_id', how='left')
> pd.merge(orders, customers,
>           left_on='customer_id', right_on='customer_id',
>           how='outer', indicator=True)   # indicator=True shows the row's source
> 
> # concat: stacking rows (same columns)
> pd.concat([df_2023, df_2024], axis=0, ignore_index=True)
> # concat: stacking columns
> pd.concat([df1, df2], axis=1)
> ```

> [!important] Key Takeaway After a `left` merge, it's good practice to check whether the row count grew (duplication from a non-unique right-side key) — `assert len(result) == len(orders)` when a one-to-one relationship is expected.

---

## groupby (EN)

> [!definition] Definition `groupby()` is the pandas equivalent of SQL `GROUP BY`: it groups rows by column value(s) and lets you apply aggregate functions to each group.

> [!example] Example
> 
> ```python
> df.groupby('city')['age'].mean()
> df.groupby('city').agg(
>     avg_age=('age', 'mean'),
>     total=('age', 'count'),
>     max_age=('age', 'max')
> )
> 
> # grouping by multiple columns
> df.groupby(['city', 'gender'])['salary'].sum()
> 
> # custom function via apply
> df.groupby('city')['salary'].apply(lambda x: x.max() - x.min())
> 
> # transform: broadcast the aggregate back to original shape (e.g. share of group)
> df['city_avg_salary'] = df.groupby('city')['salary'].transform('mean')
> ```

---

## pivot_table (EN)

> [!definition] Definition `pivot_table()` builds a pivot table (like in Excel): groups data by rows/columns and aggregates values at their intersection.

> [!example] Example
> 
> ```python
> pd.pivot_table(
>     df,
>     values='sales',
>     index='region',
>     columns='month',
>     aggfunc='sum',
>     fill_value=0,
>     margins=True   # add an "All" total row/column
> )
> 
> # pivot vs pivot_table: pivot() does not aggregate and requires unique (index, columns) pairs
> df.pivot(index='date', columns='product', values='price')
> ```

> [!warning] Common Mistake Using `pivot()` instead of `pivot_table()` when there are duplicate index/columns combinations — `pivot()` raises a `ValueError`, while `pivot_table()` aggregates them automatically (via `mean` by default).

---

## Data Transformations (apply, map, astype)

> [!example] Example
> 
> ```python
> df['age_group'] = df['age'].apply(lambda x: 'adult' if x >= 18 else 'minor')
> df['city_upper'] = df['city'].map(str.upper)     # map — for a Series
> df['salary'] = df['salary'].astype(float)          # type conversion
> 
> df.rename(columns={'name': 'full_name'}, inplace=True)
> df.sort_values('age', ascending=False)
> df.drop(columns=['city'])
> df.drop_duplicates(subset=['customer_id'])
> df['category'] = df['category'].replace({'old': 'new'})
> ```

> [!important] Key Takeaway Row-wise `apply(axis=1)` on a DataFrame is **significantly slower** than vectorized pandas/NumPy operations. Rule of thumb: if an operation can be expressed with built-in vectorized methods (`df['a'] + df['b']`, `np.where`), use those instead of `apply`.

---

## Dates and Missing Values

> [!example] Example
> 
> ```python
> df['order_date'] = pd.to_datetime(df['order_date'])
> df['year'] = df['order_date'].dt.year
> df['month'] = df['order_date'].dt.month
> df['weekday'] = df['order_date'].dt.day_name()
> 
> # missing values (NaN)
> df.isna().sum()                        # count of missing values per column
> df.dropna(subset=['age'])                # drop rows with NaN in a column
> df['age'].fillna(df['age'].median())     # fill with the median
> df.fillna({'city': 'Unknown', 'age': 0})
> ```

---

## NumPy and Basic Statistics

> [!definition] Definition **NumPy** is a library for fast vectorized computation on arrays (`ndarray`); it underlies Pandas and most data analysis libraries.

> [!example] Example
> 
> ```python
> import numpy as np
> 
> arr = np.array([1, 2, 3, 4, 5])
> arr.mean(), arr.std(), np.median(arr)
> np.where(arr > 3, 'high', 'low')    # vectorized condition
> 
> # correlation between columns
> df[['age', 'salary']].corr()
> 
> # basic statistics
> df['salary'].describe()   # count, mean, std, min, 25%, 50%, 75%, max
> ```

> [!important] Key Takeaway For a BI/analyst role, it's important to understand the difference between **mean** and **median**: the mean is sensitive to outliers, the median is robust. For skewed distributions (e.g., salaries, order value) the median is often more representative.

---

## Visualization: matplotlib and seaborn

> [!definition] Definition **matplotlib** is Python's foundational visualization library (low-level, flexible). **seaborn** is a higher-level wrapper around matplotlib with a convenient syntax, good default styling, and tight integration with Pandas DataFrames.

> [!example] Example: basic matplotlib
> 
> ```python
> import matplotlib.pyplot as plt
> 
> plt.figure(figsize=(8, 5))
> plt.plot(df['month'], df['sales'])
> plt.title('Sales by Month')
> plt.xlabel('Month')
> plt.ylabel('Sales')
> plt.show()
> ```

> [!example] Example: seaborn
> 
> ```python
> import seaborn as sns
> 
> # bar plot (comparing categories)
> sns.barplot(data=df, x='city', y='salary', estimator='mean')
> 
> # histogram (distribution of a numeric variable)
> sns.histplot(data=df, x='age', bins=20, kde=True)
> 
> # scatter plot (relationship between two numeric variables)
> sns.scatterplot(data=df, x='age', y='salary', hue='city')
> 
> # heatmap (correlation matrix)
> sns.heatmap(df[['age', 'salary', 'bonus']].corr(), annot=True, cmap='coolwarm')
> 
> # line plot (trend over time)
> sns.lineplot(data=df, x='month', y='sales', hue='region')
> 
> # boxplot (distribution + outliers by category)
> sns.boxplot(data=df, x='city', y='salary')
> 
> plt.show()
> ```

> [!formula] Which Chart for Which Task
> 
> |Task|Chart|
> |---|---|
> |Comparing categories|`barplot`|
> |Distribution of one numeric variable|`histplot`, `boxplot`|
> |Relationship between two numeric variables|`scatterplot`|
> |Trend over time|`lineplot`|
> |Correlations across many variables|`heatmap`|
> |Distribution by category + outliers|`boxplot`, `violinplot`|

> [!warning] Common Mistake Plotting a `lineplot` on data that isn't sorted by the X axis — the line will zigzag back and forth. Always `df.sort_values('date')` before plotting a time series.

---

## Product Metrics and A/B Testing

> [!definition] Definition A block specific to BI/Product Analyst roles: core product metrics and statistical hypothesis testing.

Core metrics:

- **DAU/WAU/MAU** — Daily/Weekly/Monthly Active Users
- **Retention Rate** — the share of users who return N days after their first visit
- **Churn Rate** — the share of users who stop using the product
- **Conversion Rate** — the share of users completing a target action
- **LTV (Lifetime Value)** — total value a customer generates over their lifetime
- **ARPU/ARPPU** — average revenue per user / per paying user
- **Funnel** — a sequence of steps toward a target action, with conversion at each step

> [!example] Example: cohort retention analysis in Pandas
> 
> ```python
> df['cohort'] = df.groupby('user_id')['event_date'].transform('min').dt.to_period('M')
> df['period'] = (df['event_date'].dt.to_period('M') - df['cohort']).apply(lambda x: x.n)
> 
> cohort_pivot = df.pivot_table(
>     index='cohort', columns='period', values='user_id', aggfunc='nunique'
> )
> retention = cohort_pivot.divide(cohort_pivot[0], axis=0)  # share of cohort size
> ```

> [!example] Example: A/B test (t-test) with scipy
> 
> ```python
> from scipy import stats
> 
> group_a = df[df['group'] == 'control']['conversion']
> group_b = df[df['group'] == 'test']['conversion']
> 
> t_stat, p_value = stats.ttest_ind(group_a, group_b)
> if p_value < 0.05:
>     print('Statistically significant difference')
> else:
>     print('No significant difference found')
> 
> # for proportions (conversion rates) — z-test for proportions
> from statsmodels.stats.proportion import proportions_ztest
> count = [120, 150]   # number of successes per group
> nobs = [1000, 1000]  # sample size per group
> z_stat, p_value = proportions_ztest(count, nobs)
> ```

> [!important] Key Takeaway Before drawing conclusions from an A/B test, always check: is the sample size sufficient (**power analysis**), is there a **novelty/primacy effect**, and did you avoid repeatedly "peeking" at results before the test ended (peeking inflates the false positive rate).

---

## Common Interview Questions and Tasks

> [!important] Cheat Sheet
> 
> 1. What's the difference between a list and a tuple? When would you use a `set`?
> 2. What is a list comprehension, and why is it faster than a regular loop?
> 3. What's the difference between `.loc` and `.iloc`?
> 4. How do you combine two DataFrames — `merge` vs `concat` vs `join`?
> 5. How does `pivot_table` differ from `groupby`?
> 6. How do you handle missing values — when to use `dropna` vs `fillna`?
> 7. Why is `apply(axis=1)` bad practice for large datasets?
> 8. How do you compute retention/conversion by cohort in Pandas?
> 9. How do you choose the right chart type (barplot vs histplot vs scatter)?
> 10. Classic task: "given two lists, find elements common to both" → `set(a) & set(b)`.
> 11. Classic task: "count word frequency in text" → `Counter`.
> 12. Classic task: "reverse a string / check for a palindrome" → `s[::-1]`, `s == s[::-1]`.

---

## Related Notes

- [[SQL Interview Study Notes]]
- [[Indexes]]
- [[A/B Testing and Analytics]]
- [[NumPy and Statistics]]
- [[Product Metrics]]
- [[BI Tools Tableau Power BI]]