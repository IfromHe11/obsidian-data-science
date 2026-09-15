#bi-инструменты #tableau #power-bi #datalens #аналитика #визуализация

# 📊 Tableau + Power BI + DataLens — конспект для собеседований (RU)

> Конспект по трём основным BI-инструментам: концепции, синтаксис (DAX, LOD), архитектура данных, сравнение и best practices дизайна дашбордов. Составлен с прицелом на собеседования BI/Product/Data Analyst.

## Содержание

- [[#Что такое BI-инструмент и зачем он нужен]]
- [[#Tableau — основы]]
- [[#Dimensions и Measures в Tableau]]
- [[#Calculated Fields в Tableau]]
- [[#LOD-выражения Level of Detail]]
- [[#Parameters и Filters в Tableau]]
- [[#Actions в Tableau]]
- [[#Live vs Extract и Joining vs Blending]]
- [[#Power BI — основы]]
- [[#Power Query]]
- [[#Модель данных и схема звезда]]
- [[#DAX — Data Analysis Expressions]]
- [[#Measures vs Calculated Columns]]
- [[#Row-Level Security RLS]]
- [[#Power BI Service и публикация]]
- [[#Yandex DataLens — основы]]
- [[#Датасеты коннекторы и формулы в DataLens]]
- [[#Права доступа и публикация в DataLens]]
- [[#Сравнение Tableau vs Power BI vs DataLens]]
- [[#Типы визуализаций для дашбордов]]
- [[#Best practices дизайна дашбордов]]
- [[#Частые вопросы на собеседовании РУ 4]]

---

## Что такое BI-инструмент и зачем он нужен

> [!definition] Определение **BI (Business Intelligence) инструмент** — платформа для сбора, обработки, визуализации данных и построения интерактивных дашбордов, позволяющая бизнес-пользователям самостоятельно исследовать данные без написания кода.

> [!important] Ключевой вывод BI-инструмент решает задачу «последней мили» аналитики: данные уже собраны в БД/хранилище (это делает SQL/ETL), а BI-инструмент превращает их в наглядные, интерактивные, регулярно обновляемые дашборды для принятия решений — self-service аналитика для нетехнических пользователей.

---

## Tableau — основы

> [!definition] Определение **Tableau** — один из лидеров рынка BI, известен мощными возможностями визуализации "drag-and-drop", гибким языком вычисляемых полей и LOD-выражениями. Компоненты: **Worksheet** (лист с одной визуализацией), **Dashboard** (набор листов на одном экране), **Story** (последовательность дашбордов для повествования).

---

## Dimensions и Measures в Tableau

> [!definition] Определение
> 
> - **Dimensions (измерения)** — категориальные/качественные поля (дата, регион, категория товара), определяют уровень детализации (гранулярность) визуализации
> - **Measures (меры)** — количественные поля, которые агрегируются (сумма, среднее, count) — продажи, прибыль, количество

> [!example] Пример Перетаскивание `Category` (Dimension) на Columns и `Sales` (Measure, агрегация SUM) на Rows автоматически строит bar chart суммы продаж по категориям.

> [!important] Ключевой вывод Дискретные (discrete, синий цвет в Tableau) поля создают заголовки/оси-категории, непрерывные (continuous, зелёный цвет) — создают ось с градиентом значений. Одно и то же поле (например, дата) можно использовать и как дискретное, и как непрерывное — от этого меняется тип визуализации.

---

## Calculated Fields в Tableau

> [!definition] Определение **Calculated Fields** — вычисляемые поля на основе существующих данных: агрегации, логика, строковые операции, даты.

> [!example] Пример
> 
> ```
> // Простое вычисляемое поле
> Profit Ratio = SUM([Profit]) / SUM([Sales])
> 
> // Условная логика
> Customer Segment =
> IF [Sales] > 1000 THEN "VIP"
> ELSEIF [Sales] > 500 THEN "Regular"
> ELSE "Low"
> END
> 
> // Табличные вычисления (Table Calculations)
> Running Total = RUNNING_SUM(SUM([Sales]))
> YoY Growth = (ZN(SUM([Sales])) - LOOKUP(ZN(SUM([Sales])), -1)) / ABS(LOOKUP(ZN(SUM([Sales])), -1))
> ```

> [!warning] Частая ошибка Путают агрегированные вычисления (`SUM([Profit])/SUM([Sales])`) с построчными (`[Profit]/[Sales]`, агрегированным потом) — при группировке по измерениям результат может сильно различаться. Всегда проверяйте, на каком уровне детализации выполняется вычисление.

---

## LOD-выражения (Level of Detail)

> [!definition] Определение **LOD-выражения** позволяют явно задать уровень детализации вычисления, независимо от уровня детализации визуализации — фиксированный, включающий дополнительные измерения, или исключающий часть измерений.

> [!formula] Синтаксис
> 
> - **FIXED** — вычисляет на заданном уровне детализации, игнорируя фильтры визуализации (кроме контекстных)
> - **INCLUDE** — добавляет измерение к текущему уровню детализации представления
> - **EXCLUDE** — убирает измерение из текущего уровня детализации представления

> [!example] Пример
> 
> ```
> // Средний чек по клиенту, показанный на уровне региона
> {FIXED [Customer ID]: SUM([Sales])}
> 
> // Продажи по товару, включая измерение даты, даже если на визуализации его нет
> {INCLUDE [Order Date]: SUM([Sales])}
> 
> // Общие продажи по региону, игнорируя фильтр по категории на визуализации
> {EXCLUDE [Category]: SUM([Sales])}
> ```

> [!important] Ключевой вывод Классическая задача с LOD на собеседовании: «посчитать долю клиентов, совершивших повторную покупку (retention/repeat customers)» — решается через `{FIXED [Customer ID]: COUNTD([Order ID])} > 1`, а затем агрегируется на уровне визуализации.

---

## Parameters и Filters в Tableau

> [!definition] Определение
> 
> - **Filters** — ограничивают данные, попадающие в визуализацию (по измерению, мере, дате, условию, топ-N)
> - **Parameters** — глобальные переменные, задаваемые пользователем, которые можно использовать в вычисляемых полях, заголовках, для динамического переключения меры/измерения на графике

> [!example] Пример
> 
> ```
> // Параметр "Select Measure" со списком значений: "Sales", "Profit"
> // Вычисляемое поле, переключающее меру по параметру:
> Selected Measure =
> CASE [Select Measure]
>     WHEN "Sales" THEN SUM([Sales])
>     WHEN "Profit" THEN SUM([Profit])
> END
> ```

> [!important] Ключевой вывод Порядок применения фильтров в Tableau важен: **Extract Filters → Data Source Filters → Context Filters → Dimension Filters → Measure Filters → Table Calculation Filters**. Контекстные фильтры (Context Filters) применяются раньше остальных и могут кардинально изменить результат LOD-вычислений типа FIXED.

---

## Actions в Tableau

> [!definition] Определение **Actions (действия)** — интерактивность между листами дашборда: клик/наведение на одном графике фильтрует, выделяет или переходит на другой лист/URL/дашборд.

Типы: **Filter Action**, **Highlight Action**, **URL Action**, **Parameter Action**, **Set Action**.

> [!example] Пример Клик по столбцу региона в bar chart фильтрует таблицу с детализацией заказов этого региона на другом листе того же дашборда (Filter Action).

---

## Live vs Extract и Joining vs Blending

> [!definition] Определение
> 
> - **Live-соединение** — Tableau отправляет запросы напрямую в БД в реальном времени (актуальные данные, но зависит от скорости источника)
> - **Extract (.hyper)** — Tableau извлекает и сохраняет снимок данных локально (быстрее, работает офлайн, требует ручного/scheduled обновления)

> [!definition] Определение: Joining vs Blending
> 
> - **Joining** — объединение таблиц **на уровне источника данных** (как SQL JOIN), до агрегации — работает построчно
> - **Blending (Data Blending)** — объединение данных из **разных источников** на уровне агрегированных результатов по общим измерениям — работает медленнее и с ограничениями (вторичный источник агрегируется первым)

> [!important] Ключевой вывод Blending используется, когда данные физически в разных источниках (например, Excel + SQL Server) и Join невозможен технически; при возможности лучше делать Join на уровне источника — он точнее и производительнее.

---

## Power BI — основы

> [!definition] Определение **Power BI** (Microsoft) — BI-инструмент с глубокой интеграцией в экосистему Microsoft (Excel, Azure, SQL Server), известен мощным движком моделирования данных и языком **DAX**. Компоненты: **Power BI Desktop** (разработка), **Power BI Service** (облачная публикация), **Power Query** (ETL), **Power BI Report Builder** (пейджинированные отчёты).

---

## Power Query

> [!definition] Определение **Power Query** — инструмент ETL внутри Power BI (и Excel) для загрузки, очистки и трансформации данных перед загрузкой в модель, использует язык **M**.

> [!example] Пример
> 
> ```
> // M-код (генерируется визуально через UI, но можно писать вручную)
> let
>     Source = Sql.Database("server", "database"),
>     FilteredRows = Table.SelectRows(Source, each [Sales] > 0),
>     RenamedColumns = Table.RenameColumns(FilteredRows, {{"old_name", "new_name"}})
> in
>     RenamedColumns
> ```

> [!important] Ключевой вывод Трансформации в Power Query выполняются **до** загрузки данных в модель (это ETL-слой), в отличие от DAX-мер, которые вычисляются "на лету" при взаимодействии с визуализацией. Тяжёлые преобразования лучше делать в Power Query (или ещё раньше — на уровне БД), а не в DAX.

---

## Модель данных и схема "звезда"

> [!definition] Определение **Схема "звезда" (star schema)** — рекомендуемая структура данных для Power BI: одна центральная **таблица фактов** (транзакции, события — числовые метрики) связана с несколькими **таблицами измерений** (справочники: даты, продукты, клиенты — описательные атрибуты).

> [!important] Ключевой вывод Схема "звезда" вместо плоской широкой таблицы даёт: меньший объём данных (нет дублирования атрибутов измерений в каждой строке фактов), более быстрые и предсказуемые DAX-вычисления, более простую поддержку связей "один ко многим". Это ключевая best practice при построении модели в Power BI (и вообще в BI/DWH).

---

## DAX — Data Analysis Expressions

> [!definition] Определение **DAX** — функциональный язык формул Power BI для создания вычисляемых столбцов, мер и таблиц. Похож синтаксически на Excel-формулы, но работает с таблицами и контекстом фильтрации/строк.

> [!formula] Ключевые функции
> 
> ```dax
> -- Базовая мера
> Total Sales = SUM(Sales[Amount])
> 
> -- CALCULATE — меняет контекст фильтрации
> Sales Last Year = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
> 
> -- Временная аналитика (Time Intelligence)
> YTD Sales = TOTALYTD([Total Sales], 'Date'[Date])
> Sales MoM % = DIVIDE([Total Sales] - [Sales Last Month], [Sales Last Month])
> 
> -- Итерационные функции (row context)
> Weighted Avg = SUMX(Sales, Sales[Price] * Sales[Quantity]) / SUM(Sales[Quantity])
> 
> -- Ранжирование
> Sales Rank = RANKX(ALL('Product'[ProductName]), [Total Sales])
> ```

> [!important] Ключевой вывод **CALCULATE** — самая важная функция DAX: она изменяет контекст фильтра, в котором вычисляется выражение. Понимание разницы между **контекстом строки (row context)** и **контекстом фильтра (filter context)** — ключевая концепция DAX и частый вопрос на собеседовании.

---

## Measures vs Calculated Columns

> [!definition] Определение
> 
> - **Calculated Column (вычисляемый столбец)** — вычисляется **построчно** при обновлении данных, физически хранится в модели (занимает память), доступен для использования в срезах/фильтрах/осях
> - **Measure (мера)** — вычисляется **динамически** во время взаимодействия с визуализацией, в контексте текущих фильтров/группировок, не занимает память на хранение результата

> [!important] Ключевой вывод По умолчанию предпочтение отдаётся **мерам** — они гибче (пересчитываются под любой контекст фильтрации на дашборде) и эффективнее по памяти. Calculated Column нужен только когда результат требуется как **измерение** — для среза, оси или сортировки (например, категоризация "Высокий/Средний/Низкий доход" для использования как фильтр).

---

## Row-Level Security (RLS)

> [!definition] Определение **RLS** — ограничение видимости строк данных в зависимости от того, кто просматривает отчёт (например, региональный менеджер видит только данные своего региона), настраивается через роли и DAX-фильтры на уровне таблицы.

> [!example] Пример
> 
> ```dax
> // Роль "RegionalManager": правило фильтрации на таблице Sales
> [Region] = USERPRINCIPALNAME()
> ```

---

## Power BI Service и публикация

> [!definition] Определение **Power BI Service** — облачная платформа для публикации, совместного использования дашбордов, настройки расписаний обновления данных (scheduled refresh), подписок и алертов.

> [!important] Ключевой вывод Разница между **Report** (интерактивный многостраничный отчёт с визуализациями, единица работы в Power BI Desktop) и **Dashboard** (в Power BI Service — коллекция "закреплённых" плиток (tiles) из разных отчётов на одном экране, поддерживает Q&A на естественном языке, но менее интерактивен, чем сам отчёт).

---

## Yandex DataLens — основы

> [!definition] Определение **DataLens** — BI-инструмент от Яндекса (облачный сервис Yandex Cloud), популярен на русскоязычном рынке благодаря нативной интеграции с ClickHouse, YQL, Яндекс Метрикой/Директом и бесплатному тарифу для небольших объёмов.

---

## Датасеты, коннекторы и формулы в DataLens

> [!definition] Определение Работа строится по цепочке: **Подключение (Connection)** → **Датасет (Dataset)** → **Чарт (Chart)** → **Дашборд (Dashboard)**. В датасете задаются типы полей, связи, вычисляемые поля на собственном языке формул (похожем на Tableau/Excel).

> [!example] Пример
> 
> ```
> // Вычисляемое поле в DataLens (синтаксис похож на LOD Tableau)
> [Revenue per Customer] = SUM([Revenue]) / COUNT_DISTINCT([Customer ID])
> 
> // Аналог LOD FIXED — вычисление на фиксированном уровне детализации
> FIXED([Customer ID]: SUM([Revenue]))
> ```

> [!important] Ключевой вывод Поддерживаемые коннекторы включают ClickHouse, PostgreSQL, MySQL, Google Sheets, файлы CSV/Excel, а также нативные коннекторы к Яндекс Метрике и Яндекс Директу — что делает DataLens удобным выбором для продуктовой/маркетинговой аналитики в экосистеме Яндекса.

---

## Права доступа и публикация в DataLens

> [!definition] Определение DataLens поддерживает встраивание дашбордов на внешние сайты (embedding), публичные ссылки и разграничение прав доступа на уровне рабочих пространств и объектов (датасетов, чартов, дашбордов).

> [!important] Ключевой вывод Ключевое конкурентное отличие DataLens — **бесплатный тариф** с достаточно щедрыми лимитами и **встроенная интеграция с российскими сервисами** (Метрика, Директ, Яндекс Облако), что делает его частым выбором в компаниях, ориентированных на российский рынок или уже использующих экосистему Яндекса.

---

## Сравнение Tableau vs Power BI vs DataLens

> [!formula] Сравнительная таблица
> 
> |Критерий|Tableau|Power BI|DataLens|
> |---|---|---|---|
> |Визуализация|Сильнейшая, максимально гибкая|Хорошая, стандартизированная|Базовая-средняя, набор растёт|
> |Язык вычислений|Calculated Fields, LOD|DAX|Собственный язык формул|
> |Моделирование данных|Слабее, чем Power BI|Мощное (star schema, DAX)|Проще, ориентировано на датасеты|
> |Кривая обучения|Средняя-высокая|Средняя (сложнее с DAX)|Низкая-средняя|
> |Стоимость|Высокая|Средняя (дешевле при MS-стеке)|Есть бесплатный тариф|
> |Интеграция|Универсальная, много коннекторов|Глубокая с MS-экосистемой|С Яндекс-экосистемой, ClickHouse|
> |Популярность|Мировой рынок, крупный enterprise|Мировой рынок, MS-компании|СНГ/русскоязычный рынок|

> [!important] Ключевой вывод Выбор инструмента чаще определяется не "какой лучше", а **существующим стеком компании**: Microsoft-стек (Excel, Azure, SQL Server) → Power BI; максимальная гибкость визуализации и enterprise-бюджет → Tableau; облачная инфраструктура на Яндексе, ClickHouse, ограниченный бюджет → DataLens.

---

## Типы визуализаций для дашбордов

|Визуализация|Когда использовать|
|---|---|
|**KPI-карточка (Big Number)**|Один ключевой показатель "на видном месте" с трендом/сравнением с прошлым периодом|
|**Line chart**|Динамика метрики во времени|
|**Bar/Column chart**|Сравнение категорий|
|**Funnel (воронка)**|Последовательные этапы конверсии (например, воронка продаж)|
|**Waterfall (водопад)**|Разложение изменения метрики на составляющие (например, что повлияло на рост/падение выручки)|
|**Treemap**|Иерархические доли (например, доля выручки по категориям и подкategориям)|
|**Heatmap**|Матрица значений по двум измерениям (например, активность по дням недели и часам)|
|**Gauge (спидометр)**|Прогресс к цели/плану (используется умеренно, часто критикуется как неэффективный)|
|**Sankey diagram**|Потоки между состояниями/этапами (например, переходы пользователей между страницами)|
|**Scatter plot**|Связь двух метрик, поиск сегментов/выбросов|
|**Geo map**|Данные с географической привязкой|
|**Combo chart (комбо)**|Две метрики разного масштаба на одном графике (например, продажи — столбцы, маржа% — линия)|

---

## Best practices дизайна дашбордов

> [!important] Ключевые принципы
> 
> - **F-паттерн / Z-паттерн** — самое важное размещайте вверху слева (взгляд пользователя движется по этой траектории)
> - **Правило 5 секунд** — ключевой инсайт должен считываться за 5 секунд без объяснений
> - Ограничивайте число визуализаций на дашборде (обычно 5-9) — перегруженный дашборд снижает восприятие
> - Используйте цвет осмысленно: один акцентный цвет для ключевой метрики, нейтральные тона для остального, избегайте "радуги" без семантики
> - Всегда подписывайте оси, единицы измерения, период данных и дату последнего обновления
> - Интерактивность (фильтры, drill-down) вместо перегрузки статики — дайте пользователю самому углубиться в нужный срез
> - Мобильная адаптация — если дашборд будут смотреть с телефона, проектируйте отдельный вертикальный layout

> [!warning] Частая ошибка Использование 3D-графиков, избыточных spinning-элементов, круговых диаграмм (pie chart) с большим числом сегментов (>5-6) — человеческий глаз плохо сравнивает углы/площади секторов; в большинстве случаев bar chart читается точнее и быстрее, чем pie chart.

---

## Частые вопросы на собеседовании (РУ)

> [!important] Шпаргалка вопросов
> 
> 1. Чем LOD FIXED отличается от обычной агрегации на уровне визуализации в Tableau?
> 2. В чём разница между Measure и Calculated Column в Power BI, когда что использовать?
> 3. Что делает функция CALCULATE в DAX, и почему она центральная для языка?
> 4. Что такое схема "звезда" и зачем она нужна вместо плоской таблицы?
> 5. Чем Joining отличается от Blending в Tableau?
> 6. Что такое Live-соединение и Extract, какие у них плюсы/минусы?
> 7. Как реализуется разграничение доступа к данным (RLS) в Power BI?
> 8. Какие 3-5 принципов хорошего дизайна дашборда вы бы назвали?
> 9. Почему выбор BI-инструмента часто определяется текущим стеком компании, а не только функциональностью?

---

## Связанные заметки

- [[Python для аналитика — конспект для собеседований]]
- [[SQL — конспект для собеседований]]
- [[Статистика и A B тесты — конспект для собеседований]]
- [[Продуктовые метрики]]
- [[Дизайн дашбордов]]

---

---

# 📊 Tableau + Power BI + DataLens — Interview Study Notes (EN)

#bi-tools #tableau #power-bi #datalens #analytics #visualization

> Study notes on the three major BI tools: concepts, formula languages (DAX, LOD), data architecture, comparison, and dashboard design best practices. Focused on BI/Product/Data Analyst interviews.

## Table of Contents

- [[#What is a BI Tool and Why It Matters]]
- [[#Tableau Basics]]
- [[#Dimensions and Measures in Tableau]]
- [[#Calculated Fields in Tableau]]
- [[#LOD Expressions Level of Detail]]
- [[#Parameters and Filters in Tableau]]
- [[#Actions in Tableau]]
- [[#Live vs Extract and Joining vs Blending]]
- [[#Power BI Basics]]
- [[#Power Query (EN)]]
- [[#Data Model and Star Schema]]
- [[#DAX — Data Analysis Expressions]]
- [[#Measures vs Calculated Columns]]
- [[#Row-Level Security RLS (EN)]]
- [[#Power BI Service and Publishing]]
- [[#Yandex DataLens Basics]]
- [[#Datasets Connectors and Formulas in DataLens]]
- [[#Access Rights and Publishing in DataLens]]
- [[#Comparison Tableau vs Power BI vs DataLens]]
- [[#Chart Types for Dashboards]]
- [[#Dashboard Design Best Practices]]
- [[#Common Interview Questions EN 4]]

---

## What is a BI Tool and Why It Matters

> [!definition] Definition A **BI (Business Intelligence) tool** is a platform for collecting, processing, and visualizing data and building interactive dashboards, letting business users explore data themselves without writing code.

> [!important] Key Takeaway A BI tool solves the "last mile" problem of analytics: data is already collected in a database/warehouse (that's SQL/ETL's job), and a BI tool turns it into clear, interactive, regularly refreshed dashboards for decision-making — self-service analytics for non-technical users.

---

## Tableau Basics

> [!definition] Definition **Tableau** is a market-leading BI tool known for powerful drag-and-drop visualization, a flexible calculated-field language, and LOD expressions. Components: **Worksheet** (a sheet with one visualization), **Dashboard** (a set of sheets on one screen), **Story** (a sequence of dashboards for a narrative).

---

## Dimensions and Measures in Tableau

> [!definition] Definition
> 
> - **Dimensions** — categorical/qualitative fields (date, region, product category) that determine the level of detail (granularity) of a visualization
> - **Measures** — quantitative fields that get aggregated (sum, average, count) — sales, profit, quantity

> [!example] Example Dragging `Category` (Dimension) onto Columns and `Sales` (Measure, SUM aggregation) onto Rows automatically builds a bar chart of total sales by category.

> [!important] Key Takeaway Discrete fields (blue in Tableau) create category headers/axes; continuous fields (green) create a gradient value axis. The same field (e.g., a date) can be used as either discrete or continuous — this changes the type of visualization produced.

---

## Calculated Fields in Tableau

> [!definition] Definition **Calculated Fields** are fields computed from existing data: aggregations, logic, string operations, dates.

> [!example] Example
> 
> ```
> // Simple calculated field
> Profit Ratio = SUM([Profit]) / SUM([Sales])
> 
> // Conditional logic
> Customer Segment =
> IF [Sales] > 1000 THEN "VIP"
> ELSEIF [Sales] > 500 THEN "Regular"
> ELSE "Low"
> END
> 
> // Table Calculations
> Running Total = RUNNING_SUM(SUM([Sales]))
> YoY Growth = (ZN(SUM([Sales])) - LOOKUP(ZN(SUM([Sales])), -1)) / ABS(LOOKUP(ZN(SUM([Sales])), -1))
> ```

> [!warning] Common Mistake Confusing aggregated calculations (`SUM([Profit])/SUM([Sales])`) with row-level ones (`[Profit]/[Sales]`, aggregated afterward) — grouping by dimensions can produce very different results. Always check at which level of detail the calculation runs.

---

## LOD Expressions (Level of Detail)

> [!definition] Definition **LOD expressions** let you explicitly set the level of detail for a calculation, independent of the visualization's level of detail — fixed, including additional dimensions, or excluding some dimensions.

> [!formula] Syntax
> 
> - **FIXED** — computes at a specified level of detail, ignoring visualization filters (except context filters)
> - **INCLUDE** — adds a dimension to the current view's level of detail
> - **EXCLUDE** — removes a dimension from the current view's level of detail

> [!example] Example
> 
> ```
> // Average order value per customer, shown at region level
> {FIXED [Customer ID]: SUM([Sales])}
> 
> // Sales by product, including the date dimension even if it isn't in the view
> {INCLUDE [Order Date]: SUM([Sales])}
> 
> // Total sales by region, ignoring the category filter in the view
> {EXCLUDE [Category]: SUM([Sales])}
> ```

> [!important] Key Takeaway A classic LOD interview task — "calculate the share of customers who made a repeat purchase (retention/repeat customers)" — is solved with `{FIXED [Customer ID]: COUNTD([Order ID])} > 1`, then aggregated at the visualization level.

---

## Parameters and Filters in Tableau

> [!definition] Definition
> 
> - **Filters** — restrict the data included in a visualization (by dimension, measure, date, condition, top-N)
> - **Parameters** — global, user-set variables usable in calculated fields, titles, or to dynamically switch which measure/dimension a chart displays

> [!example] Example
> 
> ```
> // Parameter "Select Measure" with values: "Sales", "Profit"
> // Calculated field switching the measure based on the parameter:
> Selected Measure =
> CASE [Select Measure]
>     WHEN "Sales" THEN SUM([Sales])
>     WHEN "Profit" THEN SUM([Profit])
> END
> ```

> [!important] Key Takeaway Filter application order in Tableau matters: **Extract Filters → Data Source Filters → Context Filters → Dimension Filters → Measure Filters → Table Calculation Filters**. Context filters apply earlier than the rest and can dramatically change the results of FIXED-type LOD calculations.

---

## Actions in Tableau

> [!definition] Definition **Actions** enable interactivity between dashboard sheets: clicking/hovering on one chart filters, highlights, or navigates to another sheet/URL/dashboard.

Types: **Filter Action**, **Highlight Action**, **URL Action**, **Parameter Action**, **Set Action**.

> [!example] Example Clicking a region's bar in a bar chart filters an order-detail table on another sheet of the same dashboard to that region (Filter Action).

---

## Live vs Extract and Joining vs Blending

> [!definition] Definition
> 
> - **Live connection** — Tableau sends queries directly to the database in real time (up-to-date data, but performance depends on the source)
> - **Extract (.hyper)** — Tableau pulls and stores a data snapshot locally (faster, works offline, requires manual/scheduled refresh)

> [!definition] Definition: Joining vs Blending
> 
> - **Joining** — combines tables **at the data source level** (like a SQL JOIN), before aggregation — operates row by row
> - **Blending (Data Blending)** — combines data from **different sources** at the level of aggregated results over shared dimensions — slower and more limited (the secondary source is aggregated first)

> [!important] Key Takeaway Blending is used when data physically lives in different sources (e.g., Excel + SQL Server) and a Join isn't technically possible; when a Join is possible, it's preferred — it's more accurate and performs better.

---

## Power BI Basics

> [!definition] Definition **Power BI** (Microsoft) is a BI tool with deep integration into the Microsoft ecosystem (Excel, Azure, SQL Server), known for a powerful data-modeling engine and the **DAX** language. Components: **Power BI Desktop** (development), **Power BI Service** (cloud publishing), **Power Query** (ETL), **Power BI Report Builder** (paginated reports).

---

## Power Query (EN)

> [!definition] Definition **Power Query** is Power BI's (and Excel's) built-in ETL tool for loading, cleaning, and transforming data before it's loaded into the model; it uses the **M** language.

> [!example] Example
> 
> ```
> // M code (generated visually via UI, but can also be written by hand)
> let
>     Source = Sql.Database("server", "database"),
>     FilteredRows = Table.SelectRows(Source, each [Sales] > 0),
>     RenamedColumns = Table.RenameColumns(FilteredRows, {{"old_name", "new_name"}})
> in
>     RenamedColumns
> ```

> [!important] Key Takeaway Power Query transformations happen **before** data is loaded into the model (it's the ETL layer), unlike DAX measures, which compute "on the fly" as visuals are interacted with. Heavy transformations are better done in Power Query (or earlier still — at the database level) rather than in DAX.

---

## Data Model and Star Schema

> [!definition] Definition A **star schema** is the recommended data structure for Power BI: a single central **fact table** (transactions, events — numeric metrics) links to several **dimension tables** (lookups: dates, products, customers — descriptive attributes).

> [!important] Key Takeaway A star schema, versus a flat wide table, gives you: less data volume (no duplicated dimension attributes in every fact row), faster and more predictable DAX calculations, and easier-to-maintain one-to-many relationships. This is a key best practice when building a Power BI model (and BI/DWH modeling generally).

---

## DAX — Data Analysis Expressions

> [!definition] Definition **DAX** is Power BI's functional formula language for creating calculated columns, measures, and tables. Syntactically similar to Excel formulas, but operates on tables with row/filter context.

> [!formula] Key Functions
> 
> ```dax
> -- Basic measure
> Total Sales = SUM(Sales[Amount])
> 
> -- CALCULATE — modifies filter context
> Sales Last Year = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
> 
> -- Time Intelligence
> YTD Sales = TOTALYTD([Total Sales], 'Date'[Date])
> Sales MoM % = DIVIDE([Total Sales] - [Sales Last Month], [Sales Last Month])
> 
> -- Iterator functions (row context)
> Weighted Avg = SUMX(Sales, Sales[Price] * Sales[Quantity]) / SUM(Sales[Quantity])
> 
> -- Ranking
> Sales Rank = RANKX(ALL('Product'[ProductName]), [Total Sales])
> ```

> [!important] Key Takeaway **CALCULATE** is DAX's most important function — it changes the filter context in which an expression is evaluated. Understanding the difference between **row context** and **filter context** is a core DAX concept and a common interview topic.

---

## Measures vs Calculated Columns

> [!definition] Definition
> 
> - **Calculated Column** — computed **row by row** when data refreshes, physically stored in the model (uses memory), available for use in slicers/filters/axes
> - **Measure** — computed **dynamically** during interaction with a visual, in the context of current filters/groupings, doesn't consume memory to store its result

> [!important] Key Takeaway Measures are preferred by default — they're more flexible (recalculate for any filter context on the dashboard) and more memory-efficient. A calculated column is needed only when the result is required as a **dimension** — for a slicer, axis, or sort order (e.g., categorizing "High/Medium/Low income" for use as a filter).

---

## Row-Level Security (RLS) (EN)

> [!definition] Definition **RLS** restricts the visibility of data rows based on who's viewing the report (e.g., a regional manager sees only their region's data), configured via roles and table-level DAX filters.

> [!example] Example
> 
> ```dax
> // Role "RegionalManager": filter rule on the Sales table
> [Region] = USERPRINCIPALNAME()
> ```

---

## Power BI Service and Publishing

> [!definition] Definition **Power BI Service** is the cloud platform for publishing and sharing dashboards, configuring scheduled data refresh, subscriptions, and alerts.

> [!important] Key Takeaway The difference between a **Report** (an interactive, multi-page collection of visuals, the unit of work in Power BI Desktop) and a **Dashboard** (in Power BI Service — a collection of "pinned" tiles from different reports on one screen, supports natural-language Q&A, but is less interactive than the report itself).

---

## Yandex DataLens Basics

> [!definition] Definition **DataLens** is Yandex's BI tool (a Yandex Cloud service), popular in the Russian-speaking market for its native integration with ClickHouse, YQL, Yandex Metrica/Direct, and a free tier for small data volumes.

---

## Datasets, Connectors, and Formulas in DataLens

> [!definition] Definition The workflow is a chain: **Connection** → **Dataset** → **Chart** → **Dashboard**. Within a dataset you define field types, relationships, and calculated fields using its own formula language (similar to Tableau/Excel).

> [!example] Example
> 
> ```
> // Calculated field in DataLens (syntax similar to Tableau's LOD)
> [Revenue per Customer] = SUM([Revenue]) / COUNT_DISTINCT([Customer ID])
> 
> // Analogous to LOD FIXED — computing at a fixed level of detail
> FIXED([Customer ID]: SUM([Revenue]))
> ```

> [!important] Key Takeaway Supported connectors include ClickHouse, PostgreSQL, MySQL, Google Sheets, CSV/Excel files, plus native connectors to Yandex Metrica and Yandex Direct — making DataLens a convenient choice for product/marketing analytics within the Yandex ecosystem.

---

## Access Rights and Publishing in DataLens

> [!definition] Definition DataLens supports embedding dashboards on external sites, public links, and access control at the level of workspaces and objects (datasets, charts, dashboards).

> [!important] Key Takeaway DataLens's key competitive edge is a **free tier** with fairly generous limits and **built-in integration with Russian services** (Metrica, Direct, Yandex Cloud), making it a common choice for companies focused on the Russian market or already using the Yandex ecosystem.

---

## Comparison: Tableau vs Power BI vs DataLens

> [!formula] Comparison Table
> 
> |Criterion|Tableau|Power BI|DataLens|
> |---|---|---|---|
> |Visualization|Strongest, most flexible|Good, standardized|Basic-to-moderate, growing|
> |Formula language|Calculated Fields, LOD|DAX|Its own formula language|
> |Data modeling|Weaker than Power BI|Powerful (star schema, DAX)|Simpler, dataset-oriented|
> |Learning curve|Moderate-to-high|Moderate (DAX adds complexity)|Low-to-moderate|
> |Cost|High|Moderate (cheaper within an MS stack)|Has a free tier|
> |Integration|Universal, many connectors|Deep with the MS ecosystem|With the Yandex ecosystem, ClickHouse|
> |Popularity|Global market, large enterprise|Global market, MS-stack companies|CIS/Russian-speaking market|

> [!important] Key Takeaway Tool choice is usually driven less by "which is best" and more by **the company's existing stack**: a Microsoft stack (Excel, Azure, SQL Server) → Power BI; maximum visualization flexibility and enterprise budget → Tableau; cloud infrastructure on Yandex, ClickHouse, limited budget → DataLens.

---

## Chart Types for Dashboards

|Visualization|When to Use|
|---|---|
|**KPI Card (Big Number)**|One key metric front and center, with a trend/comparison to a prior period|
|**Line chart**|Metric trend over time|
|**Bar/Column chart**|Comparing categories|
|**Funnel**|Sequential conversion steps (e.g., a sales funnel)|
|**Waterfall**|Breaking down a metric's change into contributing factors (e.g., what drove revenue up/down)|
|**Treemap**|Hierarchical shares (e.g., revenue share by category and subcategory)|
|**Heatmap**|A matrix of values across two dimensions (e.g., activity by weekday and hour)|
|**Gauge**|Progress toward a goal/target (used sparingly, often criticized as ineffective)|
|**Sankey diagram**|Flows between states/stages (e.g., user transitions between pages)|
|**Scatter plot**|Relationship between two metrics, finding segments/outliers|
|**Geo map**|Geographically-tagged data|
|**Combo chart**|Two metrics of different scale on one chart (e.g., sales as bars, margin% as a line)|

---

## Dashboard Design Best Practices

> [!important] Key Principles
> 
> - **F-pattern / Z-pattern** — put the most important information top-left (where the user's eye naturally travels first)
> - **5-second rule** — the key insight should be graspable within 5 seconds, without explanation
> - Limit the number of visuals on a dashboard (typically 5-9) — a cluttered dashboard hurts comprehension
> - Use color purposefully: one accent color for the key metric, neutral tones for the rest, avoid a meaningless "rainbow" palette
> - Always label axes, units, the data period, and the last-refresh date
> - Favor interactivity (filters, drill-down) over static overload — let users dig into the slice they need
> - Mobile adaptation — if the dashboard will be viewed on a phone, design a separate vertical layout

> [!warning] Common Mistake Using 3D charts, excessive spinning elements, or pie charts with too many segments (>5-6) — the human eye compares angles/areas poorly; in most cases, a bar chart reads more accurately and quickly than a pie chart.

---

## Common Interview Questions (EN)

> [!important] Cheat Sheet
> 
> 1. How does LOD FIXED differ from regular aggregation at the visualization level in Tableau?
> 2. What's the difference between a Measure and a Calculated Column in Power BI, and when do you use each?
> 3. What does the CALCULATE function do in DAX, and why is it central to the language?
> 4. What is a star schema, and why use it instead of a flat table?
> 5. How does Joining differ from Blending in Tableau?
> 6. What are Live connections and Extracts, and what are their pros/cons?
> 7. How is row-level data access control (RLS) implemented in Power BI?
> 8. What 3-5 principles of good dashboard design would you name?
> 9. Why is BI tool choice often driven by a company's existing stack rather than functionality alone?

---

## Related Notes

- [[Python for Analysts — Interview Study Notes]]
- [[SQL Interview Study Notes]]
- [[Statistics & A/B Testing — Interview Study Notes]]
- [[Product Metrics]]
- [[Dashboard Design]]