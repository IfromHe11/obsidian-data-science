#статистика #ab-тесты #теория-вероятностей #аналитика #data-science

# 📊 Статистика и A/B-тесты — конспект для собеседований (RU)

> Глубокий конспект по статистике для аналитиков: от описательной статистики и критериев до дизайна A/B-теста, продвинутых методов (CUPED, бутстрап) и альтернатив A/B (DiD, causal impact). Составлен с прицелом на собеседования Data/Product/BI Analyst и Data Scientist.

## Содержание

- [[#Описательная статистика]]
- [[#Меры центральной тенденции]]
- [[#Меры разброса]]
- [[#Квантили перцентили]]
- [[#Распределения — краткий обзор]]
- [[#Статистические критерии]]
- [[#t-test]]
- [[#z-test]]
- [[#Mann-Whitney U-тест]]
- [[#Хи-квадрат]]
- [[#Как выбрать критерий]]
- [[#Дизайн A B теста]]
- [[#Формулировка гипотез]]
- [[#Размер выборки и MDE]]
- [[#Длительность теста]]
- [[#Разбиение на группы]]
- [[#Анализ результатов]]
- [[#p-value]]
- [[#Доверительные интервалы]]
- [[#Ошибки I и II рода и мощность]]
- [[#Продвинутые методы]]
- [[#CUPED]]
- [[#Бутстрап]]
- [[#Бакетизация]]
- [[#Множественные сравнения]]
- [[#Байесовский подход к A B тестам]]
- [[#Альтернативы A B тестированию]]
- [[#Difference-in-Differences DiD]]
- [[#Causal Impact]]
- [[#Региональные гео тесты]]
- [[#Свичбек-тесты]]
- [[#Частые вопросы на собеседовании РУ 2]]

---

## Описательная статистика

> [!definition] Определение **Описательная статистика** — методы обобщения и представления данных числами и графиками: где "центр" данных, насколько они разбросаны, какая у них форма распределения.

---

## Меры центральной тенденции

> [!formula] Среднее (mean) $$\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i$$

> [!formula] Медиана (median) Значение, которое делит упорядоченный набор данных пополам. Для чётного n — среднее двух центральных значений.

> [!definition] Мода (mode) Наиболее часто встречающееся значение в наборе данных. Может быть несколько мод (мультимодальность) или ни одной (все значения уникальны).

> [!example] Пример
> 
> ```python
> import numpy as np
> from scipy import stats
> 
> data = [1, 2, 2, 3, 4, 100]  # выброс = 100
> np.mean(data)     # 18.67 — сильно искажено выбросом
> np.median(data)    # 2.5   — устойчива к выбросу
> stats.mode(data)    # мода = 2
> ```

> [!important] Ключевой вывод На собеседовании часто спрашивают: «когда медиана лучше среднего?» — ответ: при **skewed**-распределениях и наличии **выбросов** (доходы, время загрузки страницы, чек покупки). Среднее чувствительно к экстремальным значениям, медиана — робастна.

---

## Меры разброса

> [!formula] Дисперсия (variance) $$\sigma^2 = \frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})^2 \quad \text{(генеральная)}$$ $$s^2 = \frac{1}{n-1}\sum_{i=1}^{n}(x_i - \bar{x})^2 \quad \text{(выборочная, несмещённая)}$$

> [!formula] Стандартное отклонение (std) $$\sigma = \sqrt{\sigma^2}$$ Измеряется в тех же единицах, что и сами данные (в отличие от дисперсии) — поэтому интерпретируется интуитивнее.

> [!definition] Размах и межквартильный размах (IQR)
> 
> - **Размах** = max − min
> - **IQR** = Q3 − Q1 (межквартильный размах, устойчив к выбросам, используется в boxplot для определения "усов" и выбросов: точка — выброс, если она вне $[Q1 - 1.5 \cdot IQR,\ Q3 + 1.5 \cdot IQR]$)

> [!warning] Частая ошибка Деление на `n` вместо `n-1` при расчёте **выборочной** дисперсии (поправка Бесселя) — деление на `n` даёт смещённую (заниженную) оценку генеральной дисперсии. В `pandas`/`numpy`: `np.var(data, ddof=1)` — выборочная, `np.var(data, ddof=0)` (по умолчанию) — генеральная.

---

## Квантили / перцентили

> [!definition] Определение **Квантиль порядка p** — значение, ниже которого лежит p-доля данных. Перцентиль — то же самое в шкале 0–100.

> [!example] Пример
> 
> ```python
> np.percentile(data, 25)   # Q1 — первый квартиль
> np.percentile(data, 50)   # медиана
> np.percentile(data, 90)   # 90-й перцентиль — часто используют для SLA (время ответа)
> 
> df['salary'].describe()   # count, mean, std, min, 25%, 50%, 75%, max
> ```

> [!important] Ключевой вывод В продуктовой аналитике для метрик времени (загрузка страницы, ответ сервера) чаще смотрят на **перцентили (p90, p95, p99)**, а не на среднее — среднее "прячет" плохой опыт небольшой доли пользователей.

---

## Распределения — краткий обзор

|Распределение|Тип|Когда используется|
|---|---|---|
|**Бернулли**|Дискретное|Один бинарный исход (клик/не клик)|
|**Биномиальное**|Дискретное|Число успехов в n испытаниях|
|**Пуассона**|Дискретное|Число редких событий за интервал времени|
|**Нормальное**|Непрерывное|Среднее большой выборки (ЦПТ), рост, ошибки измерений|
|**Экспоненциальное**|Непрерывное|Время между событиями (напр. между заказами)|
|**Равномерное**|Непрерывное/дискретное|Все исходы равновероятны|

> [!important] Ключевой вывод Центральная предельная теорема (ЦПТ): распределение выборочного **среднего** стремится к нормальному при увеличении размера выборки — независимо от формы исходного распределения. Именно поэтому t-test/z-test применимы даже к бинарным метрикам (конверсиям) при достаточно большой выборке.

---

## Статистические критерии

> [!definition] Определение **Статистический критерий (тест)** — процедура проверки гипотезы на основе данных, дающая p-value — вероятность получить такие или более экстремальные данные при условии, что верна нулевая гипотеза.

---

## t-test

> [!definition] Определение **t-критерий Стьюдента** сравнивает средние двух групп (или среднее с известным значением). Используется, когда дисперсия генеральной совокупности **неизвестна** и оценивается по выборке; хорошо работает и при относительно небольших выборках, если данные приблизительно нормальны (или выборка достаточно велика для ЦПТ).

Виды:

- **Одновыборочный** — сравнение среднего выборки с заданным значением
- **Независимых выборок (unpaired/independent)** — сравнение средних двух независимых групп (control vs test)
- **Парный (paired)** — сравнение двух измерений на одних и тех же объектах (до/после)

> [!example] Пример
> 
> ```python
> from scipy import stats
> 
> control = df[df['group'] == 'control']['session_time']
> test = df[df['group'] == 'test']['session_time']
> 
> t_stat, p_value = stats.ttest_ind(control, test, equal_var=False)  # Welch's t-test
> # equal_var=False — не предполагаем равенство дисперсий групп (безопаснее по умолчанию)
> ```

> [!warning] Частая ошибка Использование стандартного t-test (`equal_var=True`) при явно разных дисперсиях в группах — приводит к неверной оценке p-value. По умолчанию безопаснее использовать **тест Уэлча** (`equal_var=False`), который не требует равенства дисперсий.

---

## z-test

> [!definition] Определение **z-критерий** сравнивает средние (или доли), когда дисперсия генеральной совокупности **известна**, либо выборка достаточно большая (обычно n > 30), чтобы выборочное стандартное отклонение было надёжной оценкой. В A/B-тестах чаще всего применяется **z-test для пропорций (долей)** — сравнение конверсий.

> [!formula] Формула z-статистики для двух пропорций $$z = \frac{\hat{p}_1 - \hat{p}_2}{\sqrt{\hat{p}(1-\hat{p})\left(\frac{1}{n_1} + \frac{1}{n_2}\right)}}$$ где $\hat{p}$ — объединённая (pooled) пропорция.

> [!example] Пример
> 
> ```python
> from statsmodels.stats.proportion import proportions_ztest
> 
> count = [230, 280]     # число конверсий в каждой группе
> nobs = [5000, 5000]     # размер каждой группы
> z_stat, p_value = proportions_ztest(count, nobs)
> ```

> [!important] Ключевой вывод На практике при больших выборках t-test и z-test дают почти одинаковый результат (t-распределение стремится к нормальному при росте степеней свободы) — но для сравнения **конверсий (долей)** идиоматичнее и точнее использовать именно z-test для пропорций.

---

## Mann-Whitney U-тест

> [!definition] Определение **Критерий Манна-Уитни** — непараметрический аналог t-test для независимых выборок. Не требует нормальности распределения, сравнивает **ранги** значений, а не сами значения. Проверяет, что одна выборка "систематически больше" другой (проверка сдвига распределений).

> [!example] Пример
> 
> ```python
> from scipy import stats
> 
> u_stat, p_value = stats.mannwhitneyu(control, test, alternative='two-sided')
> ```

> [!important] Ключевой вывод Используйте Mann-Whitney вместо t-test, когда:
> 
> - данные сильно скошены (skewed) или содержат выбросы (например, время на сайте, чек)
> - выборка маленькая, и нельзя полагаться на ЦПТ
> - метрика порядковая (ranking), а не интервальная

> [!warning] Частая ошибка Считают, что Mann-Whitney сравнивает **медианы** — строго говоря, это не всегда так (тест чувствителен к разнице распределений в целом, а не только к сдвигу медианы, особенно при разной форме распределений в группах).

---

## Хи-квадрат

> [!definition] Определение **Критерий хи-квадрат ($\chi^2$)** проверяет связь между двумя **категориальными** переменными (тест независимости) или соответствие наблюдаемого распределения ожидаемому (тест согласия).

> [!formula] Формула $$\chi^2 = \sum \frac{(O_i - E_i)^2}{E_i}$$ где $O_i$ — наблюдаемая частота, $E_i$ — ожидаемая частота.

> [!example] Пример
> 
> ```python
> import pandas as pd
> from scipy.stats import chi2_contingency
> 
> # таблица сопряжённости: группа x купил/не купил
> contingency = pd.crosstab(df['group'], df['purchased'])
> chi2, p_value, dof, expected = chi2_contingency(contingency)
> ```

> [!important] Ключевой вывод Хи-квадрат для теста 2 групп x 2 исхода (бинарная конверсия) даёт математически эквивалентный результат z-test для пропорций — это одна и та же проверка, разные формулировки. Хи-квадрат удобнее, когда исходов/категорий **больше двух** (например, сравнение распределения по 5 тарифным планам между группами).

---

## Как выбрать критерий

> [!formula] Шпаргалка выбора теста
> 
> |Ситуация|Критерий|
> |---|---|
> |Сравнение средних 2 групп, известна/большая выборка|z-test|
> |Сравнение средних 2 групп, малая выборка, неизвестна дисперсия|t-test (Welch)|
> |Сравнение средних до/после на тех же объектах|Парный t-test|
> |Данные скошены / есть выбросы / порядковая шкала|Mann-Whitney U|
> |Сравнение пропорций/конверсий|z-test для пропорций или хи-квадрат|
> |Связь двух категориальных переменных, >2 категорий|Хи-квадрат|
> |Сравнение >2 групп по среднему|ANOVA (или Kruskal-Wallis для непараметрического случая)|

---

## Дизайн A/B-теста

> [!definition] Определение **A/B-тест** — контролируемый эксперимент, где пользователи случайно распределяются между вариантами (control/test), чтобы измерить причинный эффект изменения на метрику.

---

## Формулировка гипотез

> [!definition] Определение
> 
> - **$H_0$ (нулевая гипотеза)** — изменения нет, различие между группами объясняется случайностью
> - **$H_1$ (альтернативная гипотеза)** — эффект существует

> [!example] Пример $H_0$: конверсия в control = конверсии в test. $H_1$: конверсия в test выше конверсии в control (односторонняя) или просто отличается (двусторонняя).

> [!important] Ключевой вывод Хорошая гипотеза для A/B-теста формулируется по схеме: **«Если мы сделаем X, то метрика Y изменится, потому что Z»** — с чётким механизмом (Z), а не просто "давайте попробуем и посмотрим".

---

## Размер выборки и MDE

> [!definition] Определение **MDE (Minimum Detectable Effect)** — минимальный эффект, который тест способен статистически значимо обнаружить при заданных α, мощности и размере выборки. Чем меньше MDE нужно поймать, тем **больше** требуется выборка.

> [!formula] Приближённая формула размера выборки (для сравнения долей) $$n \approx \frac{2 \cdot (z_{\alpha/2} + z_{\beta})^2 \cdot p(1-p)}{\Delta^2}$$ где $p$ — базовая конверсия, $\Delta$ — минимальный эффект (MDE), $z_{\alpha/2}$ и $z_{\beta}$ — критические значения для заданных α и мощности.

> [!example] Пример
> 
> ```python
> from statsmodels.stats.power import NormalIndPower
> from statsmodels.stats.proportion import proportion_effectsize
> 
> effect_size = proportion_effectsize(0.10, 0.12)  # базовая 10% -> целевая 12%
> analysis = NormalIndPower()
> n = analysis.solve_power(effect_size, power=0.8, alpha=0.05, ratio=1)
> ```

> [!important] Ключевой вывод Связь четырёх параметров: **размер выборки, MDE, α, мощность** — зафиксировав 3, можно вычислить 4-й. На практике чаще всего фиксируют α=0.05, мощность=0.8, и по доступному трафику/срокам рассчитывают, какой MDE реально поймать — если он больше, чем ожидаемый реальный эффект, тест не имеет смысла запускать.

---

## Длительность теста

> [!important] Ключевой вывод Длительность = (необходимый размер выборки) / (трафик в день на группу). Но важно учитывать:
> 
> - **Недельную сезонность** — тест должен захватывать хотя бы 1-2 полных недели (поведение в будни/выходные различается)
> - **Novelty/primacy effect** — первая реакция пользователей на изменение может быть нетипичной; иногда нужен "период стабилизации"
> - Нельзя произвольно продлевать тест "пока не станет значимо" — это форма peeking (см. ниже)

---

## Разбиение на группы

> [!definition] Определение Ключевой принцип — **случайное** распределение единиц (пользователей/сессий) между группами, чтобы группы были сопоставимы по всем характеристикам, кроме тестируемого изменения.

> [!example] Пример
> 
> ```python
> import hashlib
> 
> def assign_group(user_id, salt='experiment_1'):
>     h = hashlib.md5(f'{user_id}_{salt}'.encode()).hexdigest()
>     bucket = int(h, 16) % 100
>     return 'test' if bucket < 50 else 'control'
> ```

> [!warning] Частая ошибка **SRM (Sample Ratio Mismatch)** — фактическое распределение пользователей по группам заметно отличается от заданного (например, 48/52 вместо 50/50 на большой выборке). Это сигнал технической проблемы (баг в рандомизации, разная скорость загрузки вариантов) и повод не доверять результатам теста, пока причина не найдена. Проверяется хи-квадратом на соответствие ожидаемому распределению.

---

## Анализ результатов

## p-value

> [!definition] Определение **p-value** — вероятность получить наблюдаемые (или более экстремальные) данные **при условии**, что $H_0$ верна. $$p\text{-value} = P(\text{данные} \mid H_0)$$

> [!warning] Частая ошибка p-value — это **не** вероятность того, что $H_0$ верна, и **не** вероятность того, что результат случаен. Также маленький p-value не означает большой практический эффект — при огромных выборках даже незначимая с практической точки зрения разница может дать p < 0.05 (нужно смотреть на **величину эффекта**, не только на значимость).

---

## Доверительные интервалы

> [!definition] Определение **Доверительный интервал (ДИ)** — диапазон значений, который с заданной вероятностью (обычно 95%) содержит истинное значение параметра генеральной совокупности (при многократном повторении эксперимента 95% построенных интервалов покроют истинное значение).

> [!formula] Формула (для среднего, большая выборка) $$\bar{x} \pm z_{\alpha/2} \cdot \frac{s}{\sqrt{n}}$$

> [!example] Пример
> 
> ```python
> import scipy.stats as st
> 
> mean = test.mean()
> sem = st.sem(test)   # standard error of the mean
> ci = st.t.interval(0.95, len(test)-1, loc=mean, scale=sem)
> ```

> [!important] Ключевой вывод ДИ даёт больше информации, чем один p-value: показывает не только "есть значимость или нет", но и **диапазон вероятного размера эффекта**. Если ДИ разницы между группами не пересекает 0 — эффект статистически значим на соответствующем уровне.

---

## Ошибки I и II рода и мощность

|Термин|Значение|Формула/связь|
|---|---|---|
|**Ошибка I рода (α)**|Отвергли верную $H_0$ (ложное срабатывание)|Обычно фиксируется на 0.05|
|**Ошибка II рода (β)**|Не отвергли неверную $H_0$ (пропустили эффект)|Зависит от размера выборки, MDE, α|
|**Мощность (Power = 1-β)**|Вероятность верно обнаружить эффект, если он есть|Обычно целятся в 0.8 (80%)|

> [!example] Пример: матрица решений
> 
> ||$H_0$ верна|$H_0$ неверна|
> |---|---|---|
> |Не отвергаем $H_0$|Верно (True Negative)|Ошибка II рода (β)|
> |Отвергаем $H_0$|Ошибка I рода (α)|Верно (True Positive)|

> [!important] Ключевой вывод α и β находятся в компромиссе при фиксированной выборке: снижение α (более строгий порог значимости) при том же размере выборки **увеличивает** β (снижает мощность). Единственный способ снизить обе одновременно — увеличить размер выборки.

---

## Продвинутые методы

## CUPED

> [!definition] Определение **CUPED (Controlled-experiment Using Pre-Experiment Data)** — метод снижения дисперсии метрики за счёт использования данных **до** эксперимента (ковариаты), что позволяет обнаруживать более мелкие эффекты при том же размере выборки (или требует меньшей выборки для той же мощности).

> [!formula] Формула $$Y_{cuped} = Y - \theta \cdot (X - \bar{X})$$ где $Y$ — метрика во время эксперимента, $X$ — та же метрика (или коррелированная) до эксперимента, $\theta = \frac{Cov(X,Y)}{Var(X)}$.

> [!important] Ключевой вывод CUPED особенно эффективен, когда пре-экспериментальная метрика сильно коррелирует с экспериментальной (например, история покупок пользователя за прошлый месяц коррелирует с покупками во время теста) — может сократить необходимый размер выборки на 30-50%.

---

## Бутстрап

> [!definition] Определение **Бутстрап (bootstrap)** — метод оценки распределения статистики (среднего, медианы, любой сложной метрики) через многократное сэмплирование **с возвратом** из имеющихся данных, без предположений о форме распределения.

> [!example] Пример
> 
> ```python
> import numpy as np
> 
> def bootstrap_ci(data, n_boot=10000, ci=95):
>     boot_means = [np.mean(np.random.choice(data, size=len(data), replace=True))
>                   for _ in range(n_boot)]
>     lower = np.percentile(boot_means, (100-ci)/2)
>     upper = np.percentile(boot_means, 100 - (100-ci)/2)
>     return lower, upper
> 
> ci_low, ci_high = bootstrap_ci(test_group_conversions)
> ```

> [!important] Ключевой вывод Бутстрап особенно полезен для метрик, у которых нет простой аналитической формулы для дисперсии/ДИ (например, медиана, отношение метрик, сложные агрегаты типа "выручка на активного пользователя") — там, где классические t/z-тесты неприменимы напрямую.

---

## Бакетизация

> [!definition] Определение **Бакетизация (bucketing)** — группировка пользователей/событий в "бакеты" (например, по хешу user_id) перед агрегацией метрики, что снижает вычислительную сложность анализа больших данных и может уменьшать влияние выбросов при усреднении внутри бакетов.

> [!important] Ключевой вывод Бакетизация также используется для получения **приблизительно нормально распределённой** метрики из скошенной исходной метрики (среднее по бакету стремится к нормальности по ЦПТ даже при малом числе бакетов, если исходных наблюдений в каждом бакете много) — что позволяет применять параметрические тесты (t-test) там, где сырые данные сильно не нормальны.

---

## Множественные сравнения

> [!definition] Определение **Проблема множественных сравнений (multiple comparisons problem)** — при проверке нескольких гипотез одновременно (несколько метрик, несколько вариантов теста, несколько сегментов) вероятность получить хотя бы одно ложноположительное срабатывание растёт с числом сравнений.

> [!formula] Формула вероятности хотя бы одной ложной находки $$P(\text{хотя бы 1 ложное срабатывание}) = 1 - (1-\alpha)^m$$ где $m$ — число независимых сравнений. При $\alpha=0.05$ и $m=20$: $P \approx 0.64$ — почти гарантированно найдётся "значимый" результат чисто случайно!

Методы коррекции:

- **Поправка Бонферрони** — новый порог значимости $\alpha' = \alpha / m$ (консервативная, но простая)
- **Benjamini-Hochberg (FDR)** — контролирует ожидаемую долю ложных находок среди отвергнутых гипотез, менее консервативна, чаще применяется на практике при большом числе сравнений

> [!warning] Частая ошибка "Peeking" (многократная проверка p-value по ходу теста, пока не станет значимо) — это тоже форма множественных сравнений во времени: чем чаще смотрите на p-value до окончания эксперимента, тем выше реальная вероятность ложного срабатывания, даже если формально вы "не меняли" α=0.05.

---

## Байесовский подход к A/B-тестам

> [!definition] Определение Байесовский A/B-тест обновляет **априорное** распределение вероятности («какой вариант лучше») на основе данных теста, получая **апостериорное** распределение, из которого можно напрямую получить вероятность «вариант B лучше варианта A на X%».

> [!important] Ключевой вывод Преимущества байесовского подхода: интуитивная интерпретация («вероятность 87%, что B лучше A» — в отличие от p-value, который так интерпретировать нельзя), возможность непрерывно "подглядывать" за результатами без инфляции ошибки (при правильной байесовской последовательной методологии), естественная работа с приоритетными знаниями. Недостатки: выбор prior субъективен и может влиять на результат, вычислительно сложнее, менее стандартизирован в индустрии.

---

## Альтернативы A/B-тестированию

## Difference-in-Differences (DiD)

> [!definition] Определение **DiD** — метод причинного вывода, сравнивающий изменение метрики **во времени** между группой, подвергшейся воздействию, и контрольной группой — используется, когда случайная рандомизация невозможна (например, изменение выкатили на весь город/регион).

> [!formula] Формула $$\text{DiD} = (Y_{treatment,after} - Y_{treatment,before}) - (Y_{control,after} - Y_{control,before})$$

> [!important] Ключевой вывод Ключевое допущение DiD — **parallel trends**: без вмешательства обе группы должны были бы двигаться параллельно во времени. Это допущение нужно проверять на пре-периоде и не всегда выполняется.

## Causal Impact

> [!definition] Определение **Causal Impact** (метод Google, пакет `CausalImpact`) — байесовская структурная модель временных рядов, которая строит "синтетический контроль" (прогноз того, что было бы без вмешательства) на основе коррелированных с целевой метрикой рядов, не затронутых вмешательством, и сравнивает его с фактом.

> [!important] Ключевой вывод Полезен, когда есть только **один** объект воздействия (например, запуск рекламной кампании в одном городе) и нет классической контрольной группы — модель "создаёт" контрфактический прогноз на основе исторических данных и коррелированных рядов.

## Региональные (гео) тесты

> [!definition] Определение **Гео-тесты** — рандомизация на уровне географических регионов (город, страна) вместо отдельных пользователей — применяются, когда эффект от воздействия распространяется на всех пользователей региона (сетевые эффекты, оффлайн-маркетинг, ценообразование) и индивидуальная рандомизация невозможна или вызывает "утечку" эффекта между группами (interference/spillover).

## Свичбек-тесты

> [!definition] Определение **Свичбек-тест (switchback test)** — вариант рандомизации по **времени**, а не по пользователям: весь система/регион попеременно переключается между вариантом A и B в случайные интервалы времени (часы/дни). Часто используется в маркетплейсах и двусторонних рынках (такси, доставка), где эффект от изменения политики ценообразования влияет на весь рынок одновременно и пользовательская рандомизация невозможна из-за сетевых эффектов между водителями/пассажирами.

> [!important] Ключевой вывод Общая причина использования DiD/Causal Impact/гео-тестов/свичбек-тестов вместо классического A/B — **network effects (интерференция)**: когда пользователи из группы test и control взаимодействуют друг с другом (общий пул водителей, общий склад товаров, социальные сети), классическая рандомизация по пользователям нарушает независимость наблюдений и искажает оценку эффекта.

---

## Частые вопросы на собеседовании (РУ)

> [!important] Шпаргалка вопросов
> 
> 1. В чём разница между t-test и z-test? Когда использовать Mann-Whitney?
> 2. Как рассчитать необходимый размер выборки для A/B-теста?
> 3. Что такое MDE и как он связан с размером выборки?
> 4. Объясните ошибки I и II рода на конкретном примере.
> 5. Что такое SRM и как его обнаружить?
> 6. Как работает CUPED и зачем он нужен?
> 7. Зачем использовать бутстрап вместо аналитической формулы ДИ?
> 8. Почему peeking увеличивает вероятность ложного срабатывания?
> 9. Когда классический A/B-тест неприменим, и какие есть альтернативы?
> 10. В чём разница между частотным и байесовским подходом к A/B-тестам?

---

## Связанные заметки

- [[Теория вероятностей — конспект для собеседований]]
- [[SQL — конспект для собеседований]]
- [[Python для аналитика — конспект для собеседований]]
- [[Продуктовые метрики]]
- [[Причинно-следственный вывод Causal Inference]]

---

---

# 📊 Statistics & A/B Testing — Interview Study Notes (EN)

#statistics #ab-testing #probability-theory #analytics #data-science

> Deep-dive study notes on statistics for analysts: from descriptive statistics and hypothesis tests to A/B test design, advanced methods (CUPED, bootstrap), and A/B alternatives (DiD, causal impact). Focused on Data/Product/BI Analyst and Data Scientist interviews.

## Table of Contents

- [[#Descriptive Statistics]]
- [[#Measures of Central Tendency]]
- [[#Measures of Spread]]
- [[#Quantiles Percentiles]]
- [[#Distributions — Quick Overview]]
- [[#Statistical Tests]]
- [[#t-test (EN)]]
- [[#z-test (EN)]]
- [[#Mann-Whitney U Test]]
- [[#Chi-Square Test]]
- [[#How to Choose a Test]]
- [[#A B Test Design]]
- [[#Formulating Hypotheses]]
- [[#Sample Size and MDE]]
- [[#Test Duration]]
- [[#Group Assignment]]
- [[#Analyzing Results]]
- [[#p-value (EN)]]
- [[#Confidence Intervals]]
- [[#Type I and II Errors and Power]]
- [[#Advanced Methods]]
- [[#CUPED (EN)]]
- [[#Bootstrap]]
- [[#Bucketing]]
- [[#Multiple Comparisons]]
- [[#Bayesian Approach to A B Testing]]
- [[#Alternatives to A B Testing]]
- [[#Difference-in-Differences DiD (EN)]]
- [[#Causal Impact (EN)]]
- [[#Geo Tests]]
- [[#Switchback Tests]]
- [[#Common Interview Questions EN 2]]

---

## Descriptive Statistics

> [!definition] Definition **Descriptive statistics** summarize and present data through numbers and charts: where the "center" of the data lies, how spread out it is, and what shape its distribution has.

---

## Measures of Central Tendency

> [!formula] Mean $$\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i$$

> [!formula] Median The value that splits the sorted dataset in half. For even n, it's the average of the two middle values.

> [!definition] Mode The most frequently occurring value in a dataset. There can be multiple modes (multimodal) or none (all values unique).

> [!example] Example
> 
> ```python
> import numpy as np
> from scipy import stats
> 
> data = [1, 2, 2, 3, 4, 100]  # outlier = 100
> np.mean(data)     # 18.67 — heavily distorted by the outlier
> np.median(data)    # 2.5   — robust to the outlier
> stats.mode(data)    # mode = 2
> ```

> [!important] Key Takeaway A common interview question: "when is the median better than the mean?" — answer: for **skewed** distributions and data with **outliers** (income, page load time, order value). The mean is sensitive to extreme values; the median is robust.

---

## Measures of Spread

> [!formula] Variance $$\sigma^2 = \frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})^2 \quad \text{(population)}$$ $$s^2 = \frac{1}{n-1}\sum_{i=1}^{n}(x_i - \bar{x})^2 \quad \text{(sample, unbiased)}$$

> [!formula] Standard Deviation $$\sigma = \sqrt{\sigma^2}$$ Measured in the same units as the data itself (unlike variance) — so it's more intuitive to interpret.

> [!definition] Range and Interquartile Range (IQR)
> 
> - **Range** = max − min
> - **IQR** = Q3 − Q1 (robust to outliers; used in boxplots to define "whiskers" and outliers: a point is an outlier if it's outside $[Q1 - 1.5 \cdot IQR,\ Q3 + 1.5 \cdot IQR]$)

> [!warning] Common Mistake Dividing by `n` instead of `n-1` when computing **sample** variance (Bessel's correction) — dividing by `n` produces a biased (underestimated) estimate of population variance. In `pandas`/`numpy`: `np.var(data, ddof=1)` is sample variance, `np.var(data, ddof=0)` (the default) is population variance.

---

## Quantiles / Percentiles

> [!definition] Definition The **p-th quantile** is the value below which p-share of the data falls. A percentile is the same thing on a 0–100 scale.

> [!example] Example
> 
> ```python
> np.percentile(data, 25)   # Q1 — first quartile
> np.percentile(data, 50)   # median
> np.percentile(data, 90)   # 90th percentile — commonly used for SLAs (response time)
> 
> df['salary'].describe()   # count, mean, std, min, 25%, 50%, 75%, max
> ```

> [!important] Key Takeaway In product analytics, for latency-type metrics (page load, server response) it's more common to look at **percentiles (p90, p95, p99)** than the mean — the mean "hides" poor experience for a small share of users.

---

## Distributions — Quick Overview

|Distribution|Type|When It's Used|
|---|---|---|
|**Bernoulli**|Discrete|A single binary outcome (click/no click)|
|**Binomial**|Discrete|Number of successes in n trials|
|**Poisson**|Discrete|Number of rare events over a time interval|
|**Normal**|Continuous|Mean of a large sample (CLT), heights, measurement errors|
|**Exponential**|Continuous|Time between events (e.g., between orders)|
|**Uniform**|Continuous/Discrete|All outcomes equally likely|

> [!important] Key Takeaway The Central Limit Theorem (CLT): the distribution of the sample **mean** approaches normal as sample size increases — regardless of the shape of the underlying distribution. This is why t-tests/z-tests apply even to binary metrics (conversions) at sufficient sample sizes.

---

## Statistical Tests

> [!definition] Definition A **statistical test** is a procedure for testing a hypothesis based on data, producing a p-value — the probability of observing the data (or something more extreme) given that the null hypothesis is true.

---

## t-test (EN)

> [!definition] Definition **Student's t-test** compares the means of two groups (or a mean against a known value). Used when the population variance is **unknown** and estimated from the sample; works well even for relatively small samples if the data is roughly normal (or the sample is large enough for the CLT).

Types:

- **One-sample** — compares a sample mean to a specified value
- **Independent samples (unpaired)** — compares means of two independent groups (control vs. test)
- **Paired** — compares two measurements on the same subjects (before/after)

> [!example] Example
> 
> ```python
> from scipy import stats
> 
> control = df[df['group'] == 'control']['session_time']
> test = df[df['group'] == 'test']['session_time']
> 
> t_stat, p_value = stats.ttest_ind(control, test, equal_var=False)  # Welch's t-test
> # equal_var=False — don't assume equal group variances (safer default)
> ```

> [!warning] Common Mistake Using the standard t-test (`equal_var=True`) when group variances clearly differ leads to an incorrect p-value. It's safer to default to **Welch's t-test** (`equal_var=False`), which doesn't require equal variances.

---

## z-test (EN)

> [!definition] Definition The **z-test** compares means (or proportions) when the population variance is **known**, or the sample is large enough (typically n > 30) that the sample standard deviation is a reliable estimate. In A/B testing, the most common use is the **z-test for proportions** — comparing conversion rates.

> [!formula] Two-proportion z-statistic $$z = \frac{\hat{p}_1 - \hat{p}_2}{\sqrt{\hat{p}(1-\hat{p})\left(\frac{1}{n_1} + \frac{1}{n_2}\right)}}$$ where $\hat{p}$ is the pooled proportion.

> [!example] Example
> 
> ```python
> from statsmodels.stats.proportion import proportions_ztest
> 
> count = [230, 280]     # number of conversions in each group
> nobs = [5000, 5000]     # sample size of each group
> z_stat, p_value = proportions_ztest(count, nobs)
> ```

> [!important] Key Takeaway In practice, for large samples the t-test and z-test give nearly identical results (the t-distribution converges to normal as degrees of freedom grow) — but for comparing **conversion rates (proportions)** the proportions z-test is the more idiomatic and precise choice.

---

## Mann-Whitney U Test

> [!definition] Definition The **Mann-Whitney U test** is a non-parametric alternative to the t-test for independent samples. It doesn't require normality and compares the **ranks** of values rather than the raw values. It tests whether one sample is "systematically larger" than the other (a shift in distributions).

> [!example] Example
> 
> ```python
> from scipy import stats
> 
> u_stat, p_value = stats.mannwhitneyu(control, test, alternative='two-sided')
> ```

> [!important] Key Takeaway Use Mann-Whitney instead of the t-test when:
> 
> - data is heavily skewed or contains outliers (e.g., time on site, order value)
> - the sample is small and you can't rely on the CLT
> - the metric is ordinal rather than interval

> [!warning] Common Mistake Assuming Mann-Whitney compares **medians** — strictly speaking, this isn't always true (the test is sensitive to differences in distributions overall, not just a median shift, especially when the shapes of the distributions differ between groups).

---

## Chi-Square Test

> [!definition] Definition The **chi-square test ($\chi^2$)** checks the association between two **categorical** variables (test of independence), or how well an observed distribution matches an expected one (goodness-of-fit test).

> [!formula] Formula $$\chi^2 = \sum \frac{(O_i - E_i)^2}{E_i}$$ where $O_i$ is the observed frequency and $E_i$ is the expected frequency.

> [!example] Example
> 
> ```python
> import pandas as pd
> from scipy.stats import chi2_contingency
> 
> # contingency table: group x purchased/not purchased
> contingency = pd.crosstab(df['group'], df['purchased'])
> chi2, p_value, dof, expected = chi2_contingency(contingency)
> ```

> [!important] Key Takeaway A chi-square test on a 2-group x 2-outcome table (binary conversion) is mathematically equivalent to the proportions z-test — same underlying check, different formulation. Chi-square is more convenient when there are **more than two** outcomes/categories (e.g., comparing the distribution across 5 pricing tiers between groups).

---

## How to Choose a Test

> [!formula] Test Selection Cheat Sheet
> 
> |Situation|Test|
> |---|---|
> |Comparing means of 2 groups, known/large sample|z-test|
> |Comparing means of 2 groups, small sample, unknown variance|t-test (Welch)|
> |Comparing before/after means on the same subjects|Paired t-test|
> |Skewed data / outliers / ordinal scale|Mann-Whitney U|
> |Comparing proportions/conversions|Proportions z-test or chi-square|
> |Association between two categorical variables, >2 categories|Chi-square|
> |Comparing means across >2 groups|ANOVA (or Kruskal-Wallis for non-parametric)|

---

## A/B Test Design

> [!definition] Definition An **A/B test** is a controlled experiment where users are randomly split between variants (control/test) to measure the causal effect of a change on a metric.

---

## Formulating Hypotheses

> [!definition] Definition
> 
> - **$H_0$ (null hypothesis)** — there's no effect; the difference between groups is due to chance
> - **$H_1$ (alternative hypothesis)** — an effect exists

> [!example] Example $H_0$: conversion in control = conversion in test. $H_1$: conversion in test is higher than control (one-sided) or simply different (two-sided).

> [!important] Key Takeaway A good A/B test hypothesis follows the pattern: **"If we do X, metric Y will change, because Z"** — with a clear mechanism (Z), rather than just "let's try it and see."

---

## Sample Size and MDE

> [!definition] Definition **MDE (Minimum Detectable Effect)** is the smallest effect the test can statistically detect for a given α, power, and sample size. The smaller the MDE you need to catch, the **larger** the required sample.

> [!formula] Approximate Sample Size Formula (for comparing proportions) $$n \approx \frac{2 \cdot (z_{\alpha/2} + z_{\beta})^2 \cdot p(1-p)}{\Delta^2}$$ where $p$ is the baseline conversion rate, $\Delta$ is the minimum effect (MDE), and $z_{\alpha/2}$, $z_{\beta}$ are critical values for the chosen α and power.

> [!example] Example
> 
> ```python
> from statsmodels.stats.power import NormalIndPower
> from statsmodels.stats.proportion import proportion_effectsize
> 
> effect_size = proportion_effectsize(0.10, 0.12)  # baseline 10% -> target 12%
> analysis = NormalIndPower()
> n = analysis.solve_power(effect_size, power=0.8, alpha=0.05, ratio=1)
> ```

> [!important] Key Takeaway Four parameters are interlinked: **sample size, MDE, α, power** — fix any 3 and you can solve for the 4th. In practice, α=0.05 and power=0.8 are usually fixed, and available traffic/timeline determine what MDE is realistically detectable — if that's larger than the expected real effect, the test isn't worth running.

---

## Test Duration

> [!important] Key Takeaway Duration = (required sample size) / (daily traffic per group). But also consider:
> 
> - **Weekly seasonality** — the test should span at least 1-2 full weeks (weekday/weekend behavior differs)
> - **Novelty/primacy effect** — users' first reaction to a change may be atypical; a "burn-in" period is sometimes needed
> - You can't arbitrarily extend a test "until it becomes significant" — that's a form of peeking (see below)

---

## Group Assignment

> [!definition] Definition The key principle is **random** assignment of units (users/sessions) to groups, so groups are comparable on all characteristics except the tested change.

> [!example] Example
> 
> ```python
> import hashlib
> 
> def assign_group(user_id, salt='experiment_1'):
>     h = hashlib.md5(f'{user_id}_{salt}'.encode()).hexdigest()
>     bucket = int(h, 16) % 100
>     return 'test' if bucket < 50 else 'control'
> ```

> [!warning] Common Mistake **SRM (Sample Ratio Mismatch)** — the actual split of users across groups noticeably deviates from the intended ratio (e.g., 48/52 instead of 50/50 on a large sample). This signals a technical issue (randomization bug, different load times between variants) and is a reason to distrust the test results until the cause is found. Check with a chi-square goodness-of-fit test against the expected split.

---

## Analyzing Results

## p-value (EN)

> [!definition] Definition The **p-value** is the probability of observing the data (or something more extreme) **given** that $H_0$ is true. $$p\text{-value} = P(\text{data} \mid H_0)$$

> [!warning] Common Mistake A p-value is **not** the probability that $H_0$ is true, and **not** the probability the result is due to chance. A small p-value also doesn't imply a large practical effect — at huge sample sizes, even a practically negligible difference can yield p < 0.05 (look at **effect size**, not just significance).

---

## Confidence Intervals

> [!definition] Definition A **confidence interval (CI)** is a range of values that, with a given probability (typically 95%), contains the true population parameter (if the experiment were repeated many times, 95% of constructed intervals would cover the true value).

> [!formula] Formula (for a mean, large sample) $$\bar{x} \pm z_{\alpha/2} \cdot \frac{s}{\sqrt{n}}$$

> [!example] Example
> 
> ```python
> import scipy.stats as st
> 
> mean = test.mean()
> sem = st.sem(test)   # standard error of the mean
> ci = st.t.interval(0.95, len(test)-1, loc=mean, scale=sem)
> ```

> [!important] Key Takeaway A CI carries more information than a single p-value: it shows not just "significant or not," but also the **plausible range of effect size**. If the CI of the difference between groups doesn't cross 0, the effect is statistically significant at that level.

---

## Type I and II Errors and Power

|Term|Meaning|Formula/Relation|
|---|---|---|
|**Type I Error (α)**|Rejected a true $H_0$ (false positive)|Typically fixed at 0.05|
|**Type II Error (β)**|Failed to reject a false $H_0$ (missed effect)|Depends on sample size, MDE, α|
|**Power (1-β)**|Probability of correctly detecting an effect, if one exists|Typically targeted at 0.8 (80%)|

> [!example] Example: decision matrix
> 
> ||$H_0$ true|$H_0$ false|
> |---|---|---|
> |Fail to reject $H_0$|Correct (True Negative)|Type II Error (β)|
> |Reject $H_0$|Type I Error (α)|Correct (True Positive)|

> [!important] Key Takeaway α and β trade off at a fixed sample size: lowering α (a stricter significance threshold) with the same sample size **increases** β (lowers power). The only way to lower both simultaneously is to increase the sample size.

---

## Advanced Methods

## CUPED (EN)

> [!definition] Definition **CUPED (Controlled-experiment Using Pre-Experiment Data)** reduces metric variance by using pre-experiment data (a covariate), enabling detection of smaller effects at the same sample size (or requiring a smaller sample for the same power).

> [!formula] Formula $$Y_{cuped} = Y - \theta \cdot (X - \bar{X})$$ where $Y$ is the metric during the experiment, $X$ is the same (or a correlated) metric before the experiment, and $\theta = \frac{Cov(X,Y)}{Var(X)}$.

> [!important] Key Takeaway CUPED is especially effective when the pre-experiment metric strongly correlates with the experimental one (e.g., a user's purchase history last month correlates with purchases during the test) — it can cut the required sample size by 30-50%.

---

## Bootstrap

> [!definition] Definition **Bootstrap** estimates the distribution of a statistic (mean, median, or any complex metric) by repeatedly resampling **with replacement** from the existing data, without assuming a particular distribution shape.

> [!example] Example
> 
> ```python
> import numpy as np
> 
> def bootstrap_ci(data, n_boot=10000, ci=95):
>     boot_means = [np.mean(np.random.choice(data, size=len(data), replace=True))
>                   for _ in range(n_boot)]
>     lower = np.percentile(boot_means, (100-ci)/2)
>     upper = np.percentile(boot_means, 100 - (100-ci)/2)
>     return lower, upper
> 
> ci_low, ci_high = bootstrap_ci(test_group_conversions)
> ```

> [!important] Key Takeaway Bootstrap is especially useful for metrics without a simple analytical variance/CI formula (e.g., the median, a ratio of metrics, complex aggregates like "revenue per active user") — cases where classical t/z-tests don't directly apply.

---

## Bucketing

> [!definition] Definition **Bucketing** groups users/events into "buckets" (e.g., by hashing user_id) before aggregating a metric, which reduces the computational cost of analyzing large datasets and can reduce the impact of outliers when averaging within buckets.

> [!important] Key Takeaway Bucketing is also used to derive an **approximately normally distributed** metric from a skewed underlying metric (the per-bucket mean tends toward normality via the CLT even with relatively few buckets, provided each bucket has enough observations) — enabling parametric tests (t-test) on data that's raw-form heavily non-normal.

---

## Multiple Comparisons

> [!definition] Definition The **multiple comparisons problem** — when testing several hypotheses simultaneously (multiple metrics, multiple test variants, multiple segments), the probability of at least one false positive grows with the number of comparisons.

> [!formula] Probability of at Least One False Finding $$P(\text{at least 1 false positive}) = 1 - (1-\alpha)^m$$ where $m$ is the number of independent comparisons. At $\alpha=0.05$ and $m=20$: $P \approx 0.64$ — a "significant" result is almost guaranteed by chance alone!

Correction methods:

- **Bonferroni correction** — a new significance threshold $\alpha' = \alpha / m$ (conservative but simple)
- **Benjamini-Hochberg (FDR)** — controls the expected proportion of false discoveries among rejected hypotheses; less conservative, more commonly used in practice with many comparisons

> [!warning] Common Mistake "Peeking" (repeatedly checking the p-value throughout a test until it becomes significant) is also a form of multiple comparisons over time: the more often you check the p-value before the experiment ends, the higher the real false positive rate — even if you formally "kept" α=0.05.

---

## Bayesian Approach to A/B Testing

> [!definition] Definition A Bayesian A/B test updates a **prior** probability distribution ("which variant is better") using test data, producing a **posterior** distribution from which you can directly extract "probability that variant B beats variant A by X%."

> [!important] Key Takeaway Advantages of the Bayesian approach: intuitive interpretation ("87% probability that B beats A" — unlike a p-value, which cannot be interpreted this way), the ability to continuously "peek" at results without inflating error rates (under correct Bayesian sequential methodology), and natural incorporation of prior knowledge. Downsides: the choice of prior is subjective and can influence results, it's more computationally intensive, and less standardized across the industry.

---

## Alternatives to A/B Testing

## Difference-in-Differences (DiD) (EN)

> [!definition] Definition **DiD** is a causal inference method comparing the change in a metric **over time** between a treated group and a control group — used when random assignment isn't possible (e.g., a change rolls out to an entire city/region).

> [!formula] Formula $$\text{DiD} = (Y_{treatment,after} - Y_{treatment,before}) - (Y_{control,after} - Y_{control,before})$$

> [!important] Key Takeaway DiD's key assumption is **parallel trends**: absent the intervention, both groups would have moved in parallel over time. This assumption should be checked on pre-period data and doesn't always hold.

## Causal Impact (EN)

> [!definition] Definition **Causal Impact** (Google's method, `CausalImpact` package) is a Bayesian structural time-series model that builds a "synthetic control" (a forecast of what would have happened without the intervention) using series correlated with the target metric but unaffected by the intervention, then compares it against the actual outcome.

> [!important] Key Takeaway Useful when there's only **one** treated unit (e.g., an ad campaign launched in a single city) and no classical control group exists — the model "creates" a counterfactual forecast based on historical and correlated data.

## Geo Tests

> [!definition] Definition **Geo tests** randomize at the level of geographic regions (city, country) rather than individual users — used when an intervention's effect spreads across all users in a region (network effects, offline marketing, pricing) and individual randomization is impossible or causes effect "leakage" between groups (interference/spillover).

## Switchback Tests

> [!definition] Definition A **switchback test** randomizes by **time** rather than by users: the entire system/region alternates between variant A and B at random time intervals (hours/days). Commonly used in marketplaces and two-sided markets (ride-hailing, delivery), where a pricing-policy change affects the entire market at once and user-level randomization is impossible due to network effects between drivers/riders.

> [!important] Key Takeaway The common reason to use DiD/Causal Impact/geo tests/switchback tests instead of a classical A/B test is **network effects (interference)**: when users in the test and control groups interact with each other (a shared pool of drivers, shared inventory, social networks), classic user-level randomization violates independence of observations and biases the effect estimate.

---

## Common Interview Questions (EN)

> [!important] Cheat Sheet
> 
> 1. What's the difference between a t-test and a z-test? When would you use Mann-Whitney?
> 2. How do you calculate the required sample size for an A/B test?
> 3. What is MDE, and how does it relate to sample size?
> 4. Explain Type I and Type II errors with a concrete example.
> 5. What is SRM, and how do you detect it?
> 6. How does CUPED work, and why is it useful?
> 7. Why use bootstrap instead of an analytical CI formula?
> 8. Why does peeking increase the false positive rate?
> 9. When is a classical A/B test not applicable, and what are the alternatives?
> 10. What's the difference between the frequentist and Bayesian approach to A/B testing?

---

## Related Notes

- [[Probability Theory — Interview Study Notes]]
- [[SQL Interview Study Notes]]
- [[Python for Analysts — Interview Study Notes]]
- [[Product Metrics]]
- [[Causal Inference]]