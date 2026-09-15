#machine-learning #scikit-learn #data-science #аналитика #ml-алгоритмы

# 🤖 Базовые алгоритмы машинного обучения — конспект для собеседований (RU)

> Конспект по базовым алгоритмам ML: с примерами на scikit-learn, плюсами/минусами, критериями выбора алгоритма, функциями предобработки данных и метриками оценки моделей. Составлен с прицелом на собеседования Data Scientist / ML Analyst / Data Analyst.

## Содержание

- [[#Общие понятия переобучение недообучение bias-variance]]
- [[#Предобработка данных scikit-learn]]
- [[#Линейная регрессия]]
- [[#Логистическая регрессия]]
- [[#Регуляризация L1 L2 Ridge Lasso]]
- [[#Метод k ближайших соседей KNN]]
- [[#Наивный Байес]]
- [[#Метод опорных векторов SVM]]
- [[#Деревья решений]]
- [[#Случайный лес Random Forest]]
- [[#Градиентный бустинг]]
- [[#Ансамблевые методы — сводка]]
- [[#K-Means кластеризация]]
- [[#Иерархическая кластеризация]]
- [[#DBSCAN]]
- [[#PCA снижение размерности]]
- [[#Как выбрать алгоритм]]
- [[#Метрики оценки моделей]]
- [[#Метрики регрессии]]
- [[#Метрики классификации]]
- [[#Метрики кластеризации]]
- [[#Кросс-валидация и подбор гиперпараметров]]
- [[#Частые вопросы на собеседовании РУ 3]]

---

## Общие понятия: переобучение, недообучение, bias-variance

> [!definition] Определение
> 
> - **Недообучение (underfitting)** — модель слишком простая, плохо описывает даже обучающие данные (высокое смещение / bias)
> - **Переобучение (overfitting)** — модель слишком сложная, "запоминает" обучающие данные вместо того, чтобы обобщать закономерность (высокая дисперсия / variance), плохо работает на новых данных
> - **Bias-variance tradeoff** — компромисс: усложнение модели снижает bias, но увеличивает variance, и наоборот

> [!important] Ключевой вывод Главный признак переобучения — большой разрыв между качеством на обучающей и на тестовой/валидационной выборке (train accuracy высокая, test accuracy заметно ниже). Бороться с переобучением: регуляризация, упрощение модели, больше данных, кросс-валидация, ранняя остановка (early stopping), увеличение выборки, дропаут (для нейросетей).

---

## Предобработка данных (scikit-learn)

> [!definition] Определение Перед обучением модели данные почти всегда требуют предобработки: масштабирование, кодирование категориальных признаков, обработку пропусков, разбиение на train/test.

> [!example] Пример: полный набор инструментов предобработки
> 
> ```python
> from sklearn.model_selection import train_test_split
> from sklearn.preprocessing import StandardScaler, MinMaxScaler, OneHotEncoder, LabelEncoder
> from sklearn.impute import SimpleImputer
> from sklearn.pipeline import Pipeline
> from sklearn.compose import ColumnTransformer
> 
> # 1. Разбиение на train/test
> X_train, X_test, y_train, y_test = train_test_split(
>     X, y, test_size=0.2, random_state=42, stratify=y  # stratify — сохранить пропорции классов
> )
> 
> # 2. Масштабирование числовых признаков
> scaler = StandardScaler()           # (x - mean) / std -> среднее 0, std 1
> X_train_scaled = scaler.fit_transform(X_train)  # fit_transform ТОЛЬКО на train
> X_test_scaled = scaler.transform(X_test)          # на test только transform!
> 
> # MinMaxScaler — приводит к диапазону [0, 1], чувствителен к выбросам
> minmax = MinMaxScaler()
> 
> # 3. Кодирование категориальных признаков
> ohe = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
> X_cat_encoded = ohe.fit_transform(X[['city']])   # для номинальных категорий
> 
> le = LabelEncoder()
> y_encoded = le.fit_transform(y)                    # для целевой переменной / порядковых категорий
> 
> # 4. Обработка пропусков
> imputer = SimpleImputer(strategy='median')          # 'mean', 'median', 'most_frequent', 'constant'
> X_imputed = imputer.fit_transform(X)
> 
> # 5. Pipeline — объединение шагов в один объект (защита от утечки данных)
> pipeline = Pipeline([
>     ('imputer', SimpleImputer(strategy='median')),
>     ('scaler', StandardScaler()),
>     ('model', LogisticRegression())
> ])
> pipeline.fit(X_train, y_train)
> ```

> [!important] Ключевой вывод `fit_transform()` применяется **только** к обучающей выборке, к тестовой — только `transform()`. Если "подглядеть" статистики (среднее, min/max) теста при масштабировании train — это **утечка данных (data leakage)**, завышающая метрики на валидации нереалистично.

> [!warning] Частая ошибка Масштабирование/кодирование **до** разбиения на train/test (на всём датасете сразу) — классическая утечка данных: тестовая выборка "подсказывает" параметры трансформации модели.

---

## Линейная регрессия

> [!definition] Определение **Линейная регрессия** предсказывает непрерывную величину как линейную комбинацию признаков: $\hat{y} = \beta_0 + \beta_1 x_1 + ... + \beta_n x_n$. Обучается минимизацией суммы квадратов ошибок (MSE).

> [!example] Пример
> 
> ```python
> from sklearn.linear_model import LinearRegression
> 
> model = LinearRegression()
> model.fit(X_train, y_train)
> y_pred = model.predict(X_test)
> print(model.coef_, model.intercept_)   # коэффициенты и свободный член
> ```

> [!important] Плюсы и минусы **Плюсы:** простая, быстрая, интерпретируемая (коэффициенты показывают влияние признака), не требует много данных для базового качества. **Минусы:** предполагает линейную связь, чувствительна к выбросам и мультиколлинеарности, требует масштабирования при регуляризации. **Когда использовать:** базовый бенчмарк почти для любой регрессионной задачи, когда важна интерпретируемость, когда связь признаков с целевой переменной действительно близка к линейной.

---

## Логистическая регрессия

> [!definition] Определение **Логистическая регрессия** — несмотря на название, алгоритм **классификации**: предсказывает вероятность принадлежности к классу через сигмоиду от линейной комбинации признаков.

> [!formula] Формула $$P(y=1 \mid x) = \sigma(\beta_0 + \beta_1 x_1 + ... + \beta_n x_n) = \frac{1}{1 + e^{-z}}$$

> [!example] Пример
> 
> ```python
> from sklearn.linear_model import LogisticRegression
> 
> model = LogisticRegression(max_iter=1000, C=1.0)  # C — обратный коэфф. регуляризации
> model.fit(X_train, y_train)
> y_pred = model.predict(X_test)               # предсказанные классы
> y_proba = model.predict_proba(X_test)[:, 1]    # вероятности класса 1
> ```

> [!important] Плюсы и минусы **Плюсы:** быстрая, интерпретируемая, выдаёт вероятности (не только класс), хороший baseline для бинарной классификации. **Минусы:** предполагает линейную разделимость классов (в пространстве логитов), плохо работает при сложных нелинейных границах без feature engineering. **Когда использовать:** базовая модель для классификации, когда нужна интерпретируемость и калиброванные вероятности (например, в скоринге, медицине).

---

## Регуляризация (L1, L2, Ridge, Lasso)

> [!definition] Определение **Регуляризация** — добавление штрафа за величину коэффициентов модели в функцию потерь, чтобы предотвратить переобучение.

> [!formula] Формулы
> 
> - **L2 (Ridge):** $Loss = MSE + \alpha \sum \beta_i^2$ — "сжимает" коэффициенты к нулю, но не обнуляет
> - **L1 (Lasso):** $Loss = MSE + \alpha \sum |\beta_i|$ — может обнулять коэффициенты полностью (встроенный отбор признаков)
> - **ElasticNet:** комбинация L1 и L2

> [!example] Пример
> 
> ```python
> from sklearn.linear_model import Ridge, Lasso, ElasticNet
> 
> ridge = Ridge(alpha=1.0).fit(X_train, y_train)
> lasso = Lasso(alpha=0.1).fit(X_train, y_train)
> elastic = ElasticNet(alpha=0.1, l1_ratio=0.5).fit(X_train, y_train)
> ```

> [!important] Ключевой вывод Lasso полезен, когда подозреваете, что часть признаков нерелевантна (он обнулит их коэффициенты — встроенный feature selection). Ridge полезен при мультиколлинеарности признаков (не обнуляет, а равномерно уменьшает коррелированные коэффициенты).

---

## Метод k ближайших соседей (KNN)

> [!definition] Определение **KNN** — "ленивый" алгоритм (не строит явную модель на этапе обучения): классифицирует/предсказывает новую точку на основе k ближайших соседей в обучающей выборке (по большинству голосов для классификации, по среднему для регрессии).

> [!example] Пример
> 
> ```python
> from sklearn.neighbors import KNeighborsClassifier
> 
> model = KNeighborsClassifier(n_neighbors=5, metric='euclidean')
> model.fit(X_train_scaled, y_train)  # ОБЯЗАТЕЛЬНО масштабирование!
> y_pred = model.predict(X_test_scaled)
> ```

> [!important] Плюсы и минусы **Плюсы:** простой, интуитивный, не делает предположений о форме данных (непараметрический), хорошо работает на нелинейных границах. **Минусы:** медленный на инференсе при больших данных (нужно считать расстояния до всех точек), чувствителен к масштабу признаков и "проклятию размерности" (curse of dimensionality — в высокой размерности все точки становятся "далеко" друг от друга), чувствителен к выбросам и несбалансированным классам. **Когда использовать:** небольшие датасеты, baseline модель, когда важна простота и нет времени на сложную настройку.

---

## Наивный Байес

> [!definition] Определение **Наивный Байес** — вероятностный классификатор на основе теоремы Байеса с "наивным" предположением о независимости признаков друг от друга при условии класса.

> [!formula] Формула $$P(y \mid x_1,...,x_n) \propto P(y) \prod_{i=1}^{n} P(x_i \mid y)$$

> [!example] Пример
> 
> ```python
> from sklearn.naive_bayes import GaussianNB, MultinomialNB
> 
> # для непрерывных признаков (предполагается нормальное распределение)
> gnb = GaussianNB().fit(X_train, y_train)
> 
> # для текстовых данных / счётчиков слов (спам-фильтры, тональность текста)
> mnb = MultinomialNB().fit(X_train_counts, y_train)
> ```

> [!important] Плюсы и минусы **Плюсы:** очень быстрый (обучение и инференс), хорошо работает даже на малых выборках, отлично подходит для текстовой классификации (спам, тональность). **Минусы:** предположение о независимости признаков почти всегда нарушается на практике (но модель всё равно часто работает неплохо), оценки вероятностей обычно плохо откалиброваны. **Когда использовать:** быстрый baseline для текстовой классификации, спам-фильтры, когда нужна высокая скорость.

---

## Метод опорных векторов (SVM)

> [!definition] Определение **SVM** строит гиперплоскость, максимально разделяющую классы с наибольшим "зазором" (margin). С помощью **kernel trick** (RBF, полиномиальное ядро) может строить нелинейные границы, неявно проецируя данные в пространство большей размерности.

> [!example] Пример
> 
> ```python
> from sklearn.svm import SVC
> 
> model = SVC(kernel='rbf', C=1.0, gamma='scale', probability=True)
> model.fit(X_train_scaled, y_train)   # масштабирование обязательно
> ```

> [!important] Плюсы и минусы **Плюсы:** эффективен в пространствах высокой размерности, хорошо работает при чёткой границе между классами, устойчив к переобучению при правильной настройке C. **Минусы:** плохо масштабируется на большие датасеты (обучение $O(n^2)$–$O(n^3)$), требует тщательного подбора гиперпараметров (C, gamma, kernel), не выдаёт вероятности "из коробки" (нужен `probability=True`, что замедляет обучение). **Когда использовать:** средние по размеру датасеты с чёткой разделимостью классов, задачи с большим числом признаков (текстовая классификация, биоинформатика).

---

## Деревья решений

> [!definition] Определение **Дерево решений** — рекурсивно разбивает пространство признаков на области с помощью последовательности условий ("если признак X > порог"), в листьях — предсказание (класс/значение).

> [!example] Пример
> 
> ```python
> from sklearn.tree import DecisionTreeClassifier, DecisionTreeRegressor
> 
> model = DecisionTreeClassifier(max_depth=5, min_samples_leaf=10, random_state=42)
> model.fit(X_train, y_train)
> 
> # важность признаков
> importances = pd.Series(model.feature_importances_, index=X_train.columns)
> importances.sort_values(ascending=False)
> ```

> [!important] Плюсы и минусы **Плюсы:** очень интерпретируемо (можно визуализировать дерево), не требует масштабирования признаков, работает с числовыми и категориальными признаками, устойчиво к выбросам. **Минусы:** легко переобучается без ограничения глубины (`max_depth`, `min_samples_leaf`), нестабильно — небольшое изменение данных сильно меняет структуру дерева, слабее по качеству, чем ансамбли. **Когда использовать:** когда критична интерпретируемость (объяснить решение бизнесу), как базовый строительный блок для ансамблей (Random Forest, Gradient Boosting).

---

## Случайный лес (Random Forest)

> [!definition] Определение **Random Forest** — ансамбль множества деревьев решений, обученных на случайных подвыборках данных (**bagging**, bootstrap aggregating) и случайных подмножествах признаков, с усреднением предсказаний (голосование для классификации, среднее для регрессии).

> [!example] Пример
> 
> ```python
> from sklearn.ensemble import RandomForestClassifier
> 
> model = RandomForestClassifier(
>     n_estimators=200, max_depth=10, min_samples_leaf=5,
>     max_features='sqrt', random_state=42, n_jobs=-1
> )
> model.fit(X_train, y_train)
> ```

> [!important] Плюсы и минусы **Плюсы:** сильно снижает переобучение по сравнению с одним деревом (за счёт усреднения), хорошо работает "из коробки" с минимальной настройкой, устойчив к выбросам, не требует масштабирования, даёт feature importance. **Минусы:** менее интерпретируем, чем одно дерево, тяжелее и медленнее в инференсе (много деревьев), может уступать по качеству современному градиентному бустингу на табличных данных. **Когда использовать:** сильный универсальный baseline для табличных данных, когда важна устойчивость и минимальный тюнинг, задачи с шумными данными.

---

## Градиентный бустинг

> [!definition] Определение **Градиентный бустинг** — ансамбль деревьев, обучаемых **последовательно**: каждое следующее дерево исправляет ошибки предыдущих (обучается на остатках/градиенте функции потерь). В отличие от бэггинга — деревья не независимы, а зависят друг от друга.

Популярные реализации: **XGBoost**, **LightGBM**, **CatBoost** — оптимизированные, быстрые реализации градиентного бустинга, часто побеждающие в соревнованиях (Kaggle) на табличных данных.

> [!example] Пример
> 
> ```python
> from sklearn.ensemble import GradientBoostingClassifier
> import xgboost as xgb
> import lightgbm as lgb
> 
> # sklearn (базовая реализация)
> gbc = GradientBoostingClassifier(n_estimators=200, learning_rate=0.05, max_depth=3)
> gbc.fit(X_train, y_train)
> 
> # XGBoost (быстрее, регуляризация встроена)
> xgb_model = xgb.XGBClassifier(
>     n_estimators=300, learning_rate=0.05, max_depth=4,
>     subsample=0.8, colsample_bytree=0.8, eval_metric='logloss'
> )
> xgb_model.fit(X_train, y_train, eval_set=[(X_test, y_test)], verbose=False)
> 
> # LightGBM (быстрее на больших данных, лучше работает с категориальными признаками)
> lgb_model = lgb.LGBMClassifier(n_estimators=300, learning_rate=0.05, num_leaves=31)
> lgb_model.fit(X_train, y_train)
> ```

> [!important] Плюсы и минусы **Плюсы:** как правило, лучшее качество среди классических ML-алгоритмов на табличных данных, гибкая настройка, обработка пропусков "из коробки" (XGBoost/LightGBM/CatBoost), встроенная регуляризация. **Минусы:** легко переобучается без тщательного тюнинга (learning_rate, n_estimators, max_depth, early stopping), медленнее в обучении, чем Random Forest (последовательное обучение), сложнее интерпретировать, чувствителен к шуму в данных. **Когда использовать:** когда нужно максимальное качество на табличных данных и есть время на тюнинг гиперпараметров (соревнования, продакшн-модели скоринга, рекомендаций).

> [!warning] Частая ошибка Не использовать **early stopping** (`eval_set` + `early_stopping_rounds`) при обучении бустинга — модель легко переобучается при слишком большом `n_estimators` без остановки по валидационной метрике.

---

## Ансамблевые методы — сводка

|Метод|Принцип|Пример|
|---|---|---|
|**Bagging**|Параллельное обучение на случайных подвыборках, усреднение|Random Forest|
|**Boosting**|Последовательное обучение, исправление ошибок предыдущих моделей|XGBoost, LightGBM, CatBoost|
|**Stacking**|Обучение мета-модели на предсказаниях нескольких базовых моделей|`StackingClassifier` в sklearn|

> [!important] Ключевой вывод Bagging в первую очередь снижает **variance** (переобучение), boosting — снижает **bias** (недообучение), позволяя строить сильную модель из слабых (weak learners).

---

## K-Means (кластеризация)

> [!definition] Определение **K-Means** — алгоритм кластеризации без учителя: разбивает данные на k кластеров, минимизируя сумму квадратов расстояний точек до центроида своего кластера (итеративный алгоритм: назначение точек → пересчёт центроидов → повтор).

> [!example] Пример
> 
> ```python
> from sklearn.cluster import KMeans
> from sklearn.preprocessing import StandardScaler
> 
> X_scaled = StandardScaler().fit_transform(X)   # масштабирование обязательно
> 
> # метод локтя для выбора k
> inertias = []
> for k in range(1, 11):
>     km = KMeans(n_clusters=k, random_state=42, n_init=10).fit(X_scaled)
>     inertias.append(km.inertia_)
> 
> model = KMeans(n_clusters=4, random_state=42, n_init=10)
> labels = model.fit_predict(X_scaled)
> ```

> [!important] Плюсы и минусы **Плюсы:** прост, быстр, хорошо масштабируется на большие данные. **Минусы:** нужно заранее задать число кластеров k, предполагает сферические кластеры примерно одинакового размера, чувствителен к выбросам и масштабу признаков, чувствителен к начальной инициализации (решается через `n_init`). **Когда использовать:** сегментация клиентов, когда кластеры примерно выпуклой/сферической формы, для быстрого разведочного анализа.

---

## Иерархическая кластеризация

> [!definition] Определение Строит иерархию кластеров (дендрограмму) через последовательное объединение (агломеративная) или разделение (дивизивная) кластеров. Не требует заранее указывать число кластеров — можно "разрезать" дендрограмму на нужном уровне.

> [!example] Пример
> 
> ```python
> from sklearn.cluster import AgglomerativeClustering
> from scipy.cluster.hierarchy import dendrogram, linkage
> 
> model = AgglomerativeClustering(n_clusters=3, linkage='ward')
> labels = model.fit_predict(X_scaled)
> 
> Z = linkage(X_scaled, method='ward')
> dendrogram(Z)   # визуализация дендрограммы
> ```

> [!important] Ключевой вывод В отличие от K-Means, не требует фиксировать k заранее и не зависит от случайной инициализации, но хуже масштабируется на большие датасеты ($O(n^2)$ или $O(n^3)$ по памяти/времени).

---

## DBSCAN

> [!definition] Определение **DBSCAN** (Density-Based Spatial Clustering) находит кластеры произвольной формы на основе **плотности** точек: объединяет точки, у которых достаточно соседей в радиусе `eps`, а точки в разреженных областях помечает как выбросы (шум).

> [!example] Пример
> 
> ```python
> from sklearn.cluster import DBSCAN
> 
> model = DBSCAN(eps=0.5, min_samples=5)
> labels = model.fit_predict(X_scaled)   # -1 = выброс/шум
> ```

> [!important] Плюсы и минусы **Плюсы:** не требует задавать число кластеров, находит кластеры произвольной (не только сферической) формы, автоматически определяет выбросы. **Минусы:** чувствителен к выбору параметров `eps`/`min_samples`, плохо работает при разной плотности кластеров, хуже масштабируется на очень большие датасеты. **Когда использовать:** обнаружение аномалий/выбросов, кластеры сложной геометрической формы (например, гео-данные).

---

## PCA (снижение размерности)

> [!definition] Определение **PCA (Principal Component Analysis)** — метод снижения размерности: находит новые оси (главные компоненты) — линейные комбинации исходных признаков, вдоль которых данные имеют максимальную дисперсию, упорядоченные по убыванию объясняемой дисперсии.

> [!example] Пример
> 
> ```python
> from sklearn.decomposition import PCA
> 
> pca = PCA(n_components=0.95)   # сохранить компоненты, объясняющие 95% дисперсии
> X_reduced = pca.fit_transform(X_scaled)
> 
> print(pca.explained_variance_ratio_)   # доля дисперсии на каждую компоненту
> print(pca.n_components_)                # итоговое число компонент
> ```

> [!important] Плюсы и минусы **Плюсы:** уменьшает размерность (ускоряет обучение, борется с "проклятием размерности"), убирает мультиколлинеарность (компоненты ортогональны), полезен для визуализации многомерных данных (2D/3D). **Минусы:** новые компоненты теряют прямую интерпретируемость (это линейные комбинации исходных признаков, а не сами признаки), предполагает линейные зависимости, требует масштабирования данных перед применением. **Когда использовать:** визуализация многомерных данных, предобработка перед моделью при большом числе коррелированных признаков, сжатие данных.

---

## Как выбрать алгоритм

> [!formula] Шпаргалка выбора алгоритма
> 
> |Задача / ситуация|Рекомендуемый алгоритм|
> |---|---|
> |Нужна интерпретируемость|Линейная/логистическая регрессия, дерево решений|
> |Максимальное качество на табличных данных|Градиентный бустинг (XGBoost/LightGBM/CatBoost)|
> |Малый датасет, нужен быстрый baseline|Логистическая регрессия, Naive Bayes, KNN|
> |Много шума / выбросов|Random Forest, деревья решений|
> |Текстовая классификация|Naive Bayes, логистическая регрессия на TF-IDF|
> |Нелинейные сложные границы, средний размер данных|SVM с RBF-ядром, градиентный бустинг|
> |Сегментация без разметки|K-Means, иерархическая кластеризация|
> |Кластеры сложной формы / поиск аномалий|DBSCAN|
> |Много коррелированных признаков|PCA + любая модель, Ridge-регрессия|

---

## Метрики оценки моделей

## Метрики регрессии

> [!formula] Формулы
> 
> - **MAE** (Mean Absolute Error): $\frac{1}{n}\sum|y_i - \hat{y}_i|$ — интерпретируется в исходных единицах, устойчив к выбросам
> - **MSE** (Mean Squared Error): $\frac{1}{n}\sum(y_i - \hat{y}_i)^2$ — сильно штрафует большие ошибки
> - **RMSE**: $\sqrt{MSE}$ — в исходных единицах измерения, чувствителен к выбросам
> - **$R^2$** (коэффициент детерминации): доля объяснённой дисперсии, от $-\infty$ до 1 (1 = идеальная модель, 0 = не лучше среднего)
> - **MAPE** (Mean Absolute Percentage Error): средняя ошибка в процентах — удобна для сравнения на разных масштабах

> [!example] Пример
> 
> ```python
> from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
> 
> mae = mean_absolute_error(y_test, y_pred)
> rmse = mean_squared_error(y_test, y_pred, squared=False)
> r2 = r2_score(y_test, y_pred)
> ```

## Метрики классификации

> [!definition] Матрица ошибок (Confusion Matrix)
> 
> ||Предсказан 0|Предсказан 1|
> |---|---|---|
> |**Факт 0**|TN|FP|
> |**Факт 1**|FN|TP|

> [!formula] Формулы
> 
> - **Accuracy**: $\frac{TP+TN}{TP+TN+FP+FN}$ — доля верных предсказаний, вводит в заблуждение при дисбалансе классов
> - **Precision (точность)**: $\frac{TP}{TP+FP}$ — из тех, кого предсказали положительными, сколько реально положительны
> - **Recall (полнота)**: $\frac{TP}{TP+FN}$ — из реально положительных, сколько удалось найти
> - **F1-score**: $2 \cdot \frac{Precision \cdot Recall}{Precision + Recall}$ — гармоническое среднее precision и recall
> - **ROC-AUC**: площадь под ROC-кривой (TPR vs FPR при разных порогах) — устойчива к дисбалансу классов
> - **PR-AUC**: площадь под кривой precision-recall — более информативна при сильном дисбалансе, чем ROC-AUC
> - **Log Loss**: штрафует за уверенные, но неверные вероятностные предсказания

> [!example] Пример
> 
> ```python
> from sklearn.metrics import (
>     accuracy_score, precision_score, recall_score, f1_score,
>     roc_auc_score, confusion_matrix, classification_report, log_loss
> )
> 
> accuracy_score(y_test, y_pred)
> precision_score(y_test, y_pred)
> recall_score(y_test, y_pred)
> f1_score(y_test, y_pred)
> roc_auc_score(y_test, y_proba)     # нужны вероятности, не классы!
> confusion_matrix(y_test, y_pred)
> print(classification_report(y_test, y_pred))   # сразу все метрики по классам
> ```

> [!important] Ключевой вывод: когда что использовать
> 
> - **Дисбаланс классов** (мошенничество, редкие заболевания) → accuracy вводит в заблуждение, используйте **Precision/Recall/F1/PR-AUC**
> - **Цена ложноположительных и ложноотрицательных ошибок разная** → выбирайте метрику, отражающую бизнес-приоритет: для медицинской диагностики критичен **Recall** (не пропустить болезнь), для спам-фильтра — **Precision** (не заблокировать нужное письмо)
> - **Нужна метрика, не зависящая от порога классификации** → **ROC-AUC** или **PR-AUC**

> [!warning] Частая ошибка Использовать accuracy как основную метрику при сильном дисбалансе классов — модель, всегда предсказывающая "нет мошенничества" при 1% мошеннических транзакций, даст accuracy=99%, но абсолютно бесполезна.

## Метрики кластеризации

> [!formula] Формулы (без разметки — unsupervised)
> 
> - **Silhouette Score**: от -1 до 1, измеряет, насколько точка близка к своему кластеру относительно соседних кластеров (выше — лучше)
> - **Inertia** (внутрикластерная сумма квадратов): используется в методе локтя для выбора k в K-Means
> - **Davies-Bouldin Index**: чем ниже, тем лучше разделены кластеры

> [!example] Пример
> 
> ```python
> from sklearn.metrics import silhouette_score, davies_bouldin_score
> 
> silhouette_score(X_scaled, labels)
> davies_bouldin_score(X_scaled, labels)
> ```

---

## Кросс-валидация и подбор гиперпараметров

> [!definition] Определение **Кросс-валидация (cross-validation)** — метод более надёжной оценки качества модели: данные делятся на k частей (folds), модель обучается на k-1 частях и валидируется на оставшейся, процедура повторяется k раз с усреднением метрики — снижает зависимость оценки от случайного разбиения train/test.

> [!example] Пример
> 
> ```python
> from sklearn.model_selection import cross_val_score, KFold, GridSearchCV, RandomizedSearchCV
> 
> cv = KFold(n_splits=5, shuffle=True, random_state=42)
> scores = cross_val_score(model, X, y, cv=cv, scoring='f1')
> print(scores.mean(), scores.std())
> 
> # GridSearchCV — полный перебор сетки гиперпараметров
> param_grid = {'max_depth': [3, 5, 7], 'n_estimators': [100, 200, 300]}
> grid = GridSearchCV(RandomForestClassifier(), param_grid, cv=5, scoring='f1', n_jobs=-1)
> grid.fit(X_train, y_train)
> print(grid.best_params_, grid.best_score_)
> 
> # RandomizedSearchCV — случайный перебор (быстрее при большой сетке)
> random_search = RandomizedSearchCV(RandomForestClassifier(), param_grid, n_iter=20, cv=5)
> ```

> [!important] Ключевой вывод Для несбалансированных классов используйте `StratifiedKFold` вместо обычного `KFold`, чтобы в каждом фолде сохранялось исходное соотношение классов.

---

## Частые вопросы на собеседовании (РУ)

> [!important] Шпаргалка вопросов
> 
> 1. Чем отличается bagging от boosting?
> 2. Почему Random Forest устойчивее к переобучению, чем одно дерево?
> 3. В чём разница между L1 и L2 регуляризацией?
> 4. Когда accuracy — плохая метрика, и что использовать вместо неё?
> 5. Что такое утечка данных (data leakage) и как её избежать при масштабировании?
> 6. Чем ROC-AUC отличается от PR-AUC, когда предпочесть второй?
> 7. Зачем нужна кросс-валидация вместо одного train/test split?
> 8. Как выбрать число кластеров k в K-Means (метод локтя, silhouette)?
> 9. Почему для KNN и SVM важно масштабирование признаков, а для деревьев/лесов — нет?
> 10. В чём разница bias и variance, и как переобучение/недообучение с ними связаны?

---

## Связанные заметки

- [[Статистика и A B тесты — конспект для собеседований]]
- [[Python для аналитика — конспект для собеседований]]
- [[SQL — конспект для собеседований]]
- [[Теория вероятностей — конспект для собеседований]]
- [[Продуктовые метрики]]
- [[Feature Engineering]]

---

---

# 🤖 Core Machine Learning Algorithms — Interview Study Notes (EN)

#machine-learning #scikit-learn #data-science #analytics #ml-algorithms

> Study notes on core ML algorithms: scikit-learn examples, pros/cons, when to use each, data preprocessing functions, and model evaluation metrics. Focused on Data Scientist / ML Analyst / Data Analyst interviews.

## Table of Contents

- [[#Core Concepts Overfitting Underfitting Bias-Variance]]
- [[#Data Preprocessing scikit-learn]]
- [[#Linear Regression]]
- [[#Logistic Regression]]
- [[#Regularization L1 L2 Ridge Lasso]]
- [[#k-Nearest Neighbors KNN]]
- [[#Naive Bayes]]
- [[#Support Vector Machine SVM]]
- [[#Decision Trees]]
- [[#Random Forest]]
- [[#Gradient Boosting]]
- [[#Ensemble Methods — Summary]]
- [[#K-Means Clustering]]
- [[#Hierarchical Clustering]]
- [[#DBSCAN (EN)]]
- [[#PCA Dimensionality Reduction]]
- [[#How to Choose an Algorithm]]
- [[#Model Evaluation Metrics]]
- [[#Regression Metrics]]
- [[#Classification Metrics]]
- [[#Clustering Metrics]]
- [[#Cross-Validation and Hyperparameter Tuning]]
- [[#Common Interview Questions EN 3]]

---

## Core Concepts: Overfitting, Underfitting, Bias-Variance

> [!definition] Definition
> 
> - **Underfitting** — the model is too simple and describes even the training data poorly (high bias)
> - **Overfitting** — the model is too complex and "memorizes" the training data instead of learning the general pattern (high variance), performing poorly on new data
> - **Bias-variance tradeoff** — increasing model complexity reduces bias but increases variance, and vice versa

> [!important] Key Takeaway The main sign of overfitting is a large gap between training and test/validation performance (high train accuracy, noticeably lower test accuracy). Ways to fight overfitting: regularization, simplifying the model, more data, cross-validation, early stopping, larger sample size, dropout (for neural networks).

---

## Data Preprocessing (scikit-learn)

> [!definition] Definition Before training a model, data almost always needs preprocessing: scaling, encoding categorical features, handling missing values, and splitting into train/test.

> [!example] Example: full preprocessing toolkit
> 
> ```python
> from sklearn.model_selection import train_test_split
> from sklearn.preprocessing import StandardScaler, MinMaxScaler, OneHotEncoder, LabelEncoder
> from sklearn.impute import SimpleImputer
> from sklearn.pipeline import Pipeline
> from sklearn.compose import ColumnTransformer
> 
> # 1. Train/test split
> X_train, X_test, y_train, y_test = train_test_split(
>     X, y, test_size=0.2, random_state=42, stratify=y  # stratify — preserve class proportions
> )
> 
> # 2. Scaling numeric features
> scaler = StandardScaler()           # (x - mean) / std -> mean 0, std 1
> X_train_scaled = scaler.fit_transform(X_train)  # fit_transform ONLY on train
> X_test_scaled = scaler.transform(X_test)          # transform only on test!
> 
> # MinMaxScaler — scales to [0, 1], sensitive to outliers
> minmax = MinMaxScaler()
> 
> # 3. Encoding categorical features
> ohe = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
> X_cat_encoded = ohe.fit_transform(X[['city']])   # for nominal categories
> 
> le = LabelEncoder()
> y_encoded = le.fit_transform(y)                    # for the target / ordinal categories
> 
> # 4. Handling missing values
> imputer = SimpleImputer(strategy='median')          # 'mean', 'median', 'most_frequent', 'constant'
> X_imputed = imputer.fit_transform(X)
> 
> # 5. Pipeline — chains steps into one object (protects against data leakage)
> pipeline = Pipeline([
>     ('imputer', SimpleImputer(strategy='median')),
>     ('scaler', StandardScaler()),
>     ('model', LogisticRegression())
> ])
> pipeline.fit(X_train, y_train)
> ```

> [!important] Key Takeaway `fit_transform()` is applied **only** to the training set; the test set gets only `transform()`. Peeking at test statistics (mean, min/max) when scaling the training data is **data leakage**, which unrealistically inflates validation metrics.

> [!warning] Common Mistake Scaling/encoding **before** the train/test split (on the whole dataset at once) is classic data leakage: the test set "leaks" its statistics into the transformation parameters.

---

## Linear Regression

> [!definition] Definition **Linear regression** predicts a continuous value as a linear combination of features: $\hat{y} = \beta_0 + \beta_1 x_1 + ... + \beta_n x_n$. It's trained by minimizing the sum of squared errors (MSE).

> [!example] Example
> 
> ```python
> from sklearn.linear_model import LinearRegression
> 
> model = LinearRegression()
> model.fit(X_train, y_train)
> y_pred = model.predict(X_test)
> print(model.coef_, model.intercept_)   # coefficients and intercept
> ```

> [!important] Pros and Cons **Pros:** simple, fast, interpretable (coefficients show feature influence), doesn't need much data for baseline quality. **Cons:** assumes a linear relationship, sensitive to outliers and multicollinearity, requires scaling when regularized. **When to use:** a baseline benchmark for nearly any regression task, when interpretability matters, when the relationship between features and target is genuinely close to linear.

---

## Logistic Regression

> [!definition] Definition **Logistic regression** — despite the name, a **classification** algorithm: predicts the probability of class membership via a sigmoid applied to a linear combination of features.

> [!formula] Formula $$P(y=1 \mid x) = \sigma(\beta_0 + \beta_1 x_1 + ... + \beta_n x_n) = \frac{1}{1 + e^{-z}}$$

> [!example] Example
> 
> ```python
> from sklearn.linear_model import LogisticRegression
> 
> model = LogisticRegression(max_iter=1000, C=1.0)  # C — inverse regularization strength
> model.fit(X_train, y_train)
> y_pred = model.predict(X_test)               # predicted classes
> y_proba = model.predict_proba(X_test)[:, 1]    # probability of class 1
> ```

> [!important] Pros and Cons **Pros:** fast, interpretable, outputs probabilities (not just a class label), a solid baseline for binary classification. **Cons:** assumes linear separability of classes (in log-odds space), performs poorly on complex nonlinear boundaries without feature engineering. **When to use:** a baseline classification model, when interpretability and calibrated probabilities matter (e.g., credit scoring, medical diagnosis).

---

## Regularization (L1, L2, Ridge, Lasso)

> [!definition] Definition **Regularization** adds a penalty on the magnitude of model coefficients to the loss function to prevent overfitting.

> [!formula] Formulas
> 
> - **L2 (Ridge):** $Loss = MSE + \alpha \sum \beta_i^2$ — shrinks coefficients toward zero, but doesn't zero them out
> - **L1 (Lasso):** $Loss = MSE + \alpha \sum |\beta_i|$ — can zero out coefficients entirely (built-in feature selection)
> - **ElasticNet:** combines L1 and L2

> [!example] Example
> 
> ```python
> from sklearn.linear_model import Ridge, Lasso, ElasticNet
> 
> ridge = Ridge(alpha=1.0).fit(X_train, y_train)
> lasso = Lasso(alpha=0.1).fit(X_train, y_train)
> elastic = ElasticNet(alpha=0.1, l1_ratio=0.5).fit(X_train, y_train)
> ```

> [!important] Key Takeaway Lasso is useful when you suspect some features are irrelevant (it zeroes out their coefficients — built-in feature selection). Ridge is useful with multicollinear features (it doesn't zero them out, but shrinks correlated coefficients evenly).

---

## k-Nearest Neighbors (KNN)

> [!definition] Definition **KNN** is a "lazy" algorithm (builds no explicit model during training): it classifies/predicts a new point based on its k nearest neighbors in the training set (majority vote for classification, average for regression).

> [!example] Example
> 
> ```python
> from sklearn.neighbors import KNeighborsClassifier
> 
> model = KNeighborsClassifier(n_neighbors=5, metric='euclidean')
> model.fit(X_train_scaled, y_train)  # scaling is REQUIRED!
> y_pred = model.predict(X_test_scaled)
> ```

> [!important] Pros and Cons **Pros:** simple, intuitive, non-parametric (no assumption on data shape), works well on nonlinear boundaries. **Cons:** slow at inference on large data (needs distances to every training point), sensitive to feature scale and the "curse of dimensionality" (in high dimensions all points become "far" from each other), sensitive to outliers and class imbalance. **When to use:** small datasets, a quick baseline, when simplicity matters more than tuning time.

---

## Naive Bayes

> [!definition] Definition **Naive Bayes** is a probabilistic classifier based on Bayes' theorem with a "naive" assumption that features are independent of each other given the class.

> [!formula] Formula $$P(y \mid x_1,...,x_n) \propto P(y) \prod_{i=1}^{n} P(x_i \mid y)$$

> [!example] Example
> 
> ```python
> from sklearn.naive_bayes import GaussianNB, MultinomialNB
> 
> # for continuous features (assumes a normal distribution)
> gnb = GaussianNB().fit(X_train, y_train)
> 
> # for text data / word counts (spam filters, sentiment analysis)
> mnb = MultinomialNB().fit(X_train_counts, y_train)
> ```

> [!important] Pros and Cons **Pros:** very fast (training and inference), works well even on small samples, well suited for text classification (spam, sentiment). **Cons:** the independence assumption is almost always violated in practice (though the model often still performs decently), probability estimates are typically poorly calibrated. **When to use:** a fast baseline for text classification, spam filters, when speed is a priority.

---

## Support Vector Machine (SVM)

> [!definition] Definition **SVM** builds a hyperplane that separates classes with the largest possible margin. Using the **kernel trick** (RBF, polynomial kernel), it can build nonlinear boundaries by implicitly projecting data into a higher-dimensional space.

> [!example] Example
> 
> ```python
> from sklearn.svm import SVC
> 
> model = SVC(kernel='rbf', C=1.0, gamma='scale', probability=True)
> model.fit(X_train_scaled, y_train)   # scaling is required
> ```

> [!important] Pros and Cons **Pros:** effective in high-dimensional spaces, works well when there's a clear margin between classes, resistant to overfitting when C is properly tuned. **Cons:** scales poorly to large datasets (training is $O(n^2)$–$O(n^3)$), requires careful hyperparameter tuning (C, gamma, kernel), doesn't give probabilities out of the box (needs `probability=True`, which slows training). **When to use:** medium-sized datasets with clearly separable classes, high-dimensional feature spaces (text classification, bioinformatics).

---

## Decision Trees

> [!definition] Definition A **decision tree** recursively splits the feature space into regions via a sequence of conditions ("if feature X > threshold"), with predictions (class/value) at the leaves.

> [!example] Example
> 
> ```python
> from sklearn.tree import DecisionTreeClassifier, DecisionTreeRegressor
> 
> model = DecisionTreeClassifier(max_depth=5, min_samples_leaf=10, random_state=42)
> model.fit(X_train, y_train)
> 
> # feature importance
> importances = pd.Series(model.feature_importances_, index=X_train.columns)
> importances.sort_values(ascending=False)
> ```

> [!important] Pros and Cons **Pros:** highly interpretable (you can visualize the tree), doesn't need feature scaling, handles numeric and categorical features, robust to outliers. **Cons:** overfits easily without limiting depth (`max_depth`, `min_samples_leaf`), unstable — a small data change can substantially change the tree structure, generally weaker in quality than ensembles. **When to use:** when interpretability is critical (explaining a decision to business), or as a building block for ensembles (Random Forest, Gradient Boosting).

---

## Random Forest

> [!definition] Definition **Random Forest** is an ensemble of many decision trees trained on random data subsamples (**bagging**, bootstrap aggregating) and random feature subsets, with predictions averaged (voting for classification, mean for regression).

> [!example] Example
> 
> ```python
> from sklearn.ensemble import RandomForestClassifier
> 
> model = RandomForestClassifier(
>     n_estimators=200, max_depth=10, min_samples_leaf=5,
>     max_features='sqrt', random_state=42, n_jobs=-1
> )
> model.fit(X_train, y_train)
> ```

> [!important] Pros and Cons **Pros:** substantially reduces overfitting compared to a single tree (via averaging), works well out of the box with minimal tuning, robust to outliers, doesn't require scaling, provides feature importance. **Cons:** less interpretable than a single tree, heavier and slower at inference (many trees), can underperform modern gradient boosting on tabular data. **When to use:** a strong general-purpose baseline for tabular data, when robustness and minimal tuning matter, tasks with noisy data.

---

## Gradient Boosting

> [!definition] Definition **Gradient boosting** is an ensemble of trees trained **sequentially**: each next tree corrects the errors of the previous ones (trained on the residuals/gradient of the loss function). Unlike bagging, trees are not independent — each depends on the ones before it.

Popular implementations: **XGBoost**, **LightGBM**, **CatBoost** — optimized, fast gradient boosting implementations, frequently winning competitions (Kaggle) on tabular data.

> [!example] Example
> 
> ```python
> from sklearn.ensemble import GradientBoostingClassifier
> import xgboost as xgb
> import lightgbm as lgb
> 
> # sklearn (basic implementation)
> gbc = GradientBoostingClassifier(n_estimators=200, learning_rate=0.05, max_depth=3)
> gbc.fit(X_train, y_train)
> 
> # XGBoost (faster, built-in regularization)
> xgb_model = xgb.XGBClassifier(
>     n_estimators=300, learning_rate=0.05, max_depth=4,
>     subsample=0.8, colsample_bytree=0.8, eval_metric='logloss'
> )
> xgb_model.fit(X_train, y_train, eval_set=[(X_test, y_test)], verbose=False)
> 
> # LightGBM (faster on large data, handles categorical features better)
> lgb_model = lgb.LGBMClassifier(n_estimators=300, learning_rate=0.05, num_leaves=31)
> lgb_model.fit(X_train, y_train)
> ```

> [!important] Pros and Cons **Pros:** typically the best quality among classical ML algorithms on tabular data, flexible tuning, out-of-the-box missing-value handling (XGBoost/LightGBM/CatBoost), built-in regularization. **Cons:** overfits easily without careful tuning (learning_rate, n_estimators, max_depth, early stopping), slower to train than Random Forest (sequential training), harder to interpret, sensitive to noisy data. **When to use:** when maximum quality on tabular data is needed and there's time for hyperparameter tuning (competitions, production scoring/recommendation models).

> [!warning] Common Mistake Not using **early stopping** (`eval_set` + `early_stopping_rounds`) when training a boosted model — the model overfits easily with too large `n_estimators` and no stopping based on a validation metric.

---

## Ensemble Methods — Summary

|Method|Principle|Example|
|---|---|---|
|**Bagging**|Parallel training on random subsamples, averaged|Random Forest|
|**Boosting**|Sequential training, correcting prior models' errors|XGBoost, LightGBM, CatBoost|
|**Stacking**|Training a meta-model on the predictions of several base models|`StackingClassifier` in sklearn|

> [!important] Key Takeaway Bagging primarily reduces **variance** (overfitting); boosting reduces **bias** (underfitting), allowing a strong model to be built from weak learners.

---

## K-Means Clustering

> [!definition] Definition **K-Means** is an unsupervised clustering algorithm: it splits data into k clusters by minimizing the sum of squared distances from points to their cluster's centroid (an iterative algorithm: assign points → recompute centroids → repeat).

> [!example] Example
> 
> ```python
> from sklearn.cluster import KMeans
> from sklearn.preprocessing import StandardScaler
> 
> X_scaled = StandardScaler().fit_transform(X)   # scaling is required
> 
> # elbow method for choosing k
> inertias = []
> for k in range(1, 11):
>     km = KMeans(n_clusters=k, random_state=42, n_init=10).fit(X_scaled)
>     inertias.append(km.inertia_)
> 
> model = KMeans(n_clusters=4, random_state=42, n_init=10)
> labels = model.fit_predict(X_scaled)
> ```

> [!important] Pros and Cons **Pros:** simple, fast, scales well to large data. **Cons:** requires specifying the number of clusters k upfront, assumes roughly spherical clusters of similar size, sensitive to outliers and feature scale, sensitive to initialization (mitigated with `n_init`). **When to use:** customer segmentation, when clusters are roughly convex/spherical, for quick exploratory analysis.

---

## Hierarchical Clustering

> [!definition] Definition Builds a hierarchy of clusters (a dendrogram) through successive merging (agglomerative) or splitting (divisive) of clusters. Doesn't require specifying the number of clusters upfront — the dendrogram can be "cut" at the desired level.

> [!example] Example
> 
> ```python
> from sklearn.cluster import AgglomerativeClustering
> from scipy.cluster.hierarchy import dendrogram, linkage
> 
> model = AgglomerativeClustering(n_clusters=3, linkage='ward')
> labels = model.fit_predict(X_scaled)
> 
> Z = linkage(X_scaled, method='ward')
> dendrogram(Z)   # dendrogram visualization
> ```

> [!important] Key Takeaway Unlike K-Means, it doesn't require fixing k upfront and doesn't depend on random initialization, but scales worse to large datasets ($O(n^2)$ or $O(n^3)$ in time/memory).

---

## DBSCAN (EN)

> [!definition] Definition **DBSCAN** (Density-Based Spatial Clustering) finds clusters of arbitrary shape based on point **density**: it groups points with enough neighbors within radius `eps`, and marks points in sparse regions as outliers (noise).

> [!example] Example
> 
> ```python
> from sklearn.cluster import DBSCAN
> 
> model = DBSCAN(eps=0.5, min_samples=5)
> labels = model.fit_predict(X_scaled)   # -1 = outlier/noise
> ```

> [!important] Pros and Cons **Pros:** doesn't require specifying the number of clusters, finds clusters of arbitrary (not just spherical) shape, automatically identifies outliers. **Cons:** sensitive to the choice of `eps`/`min_samples`, struggles with clusters of varying density, scales poorly to very large datasets. **When to use:** anomaly/outlier detection, geometrically complex clusters (e.g., geospatial data).

---

## PCA (Dimensionality Reduction)

> [!definition] Definition **PCA (Principal Component Analysis)** is a dimensionality reduction method: it finds new axes (principal components) — linear combinations of the original features — along which the data has maximum variance, ordered by decreasing explained variance.

> [!example] Example
> 
> ```python
> from sklearn.decomposition import PCA
> 
> pca = PCA(n_components=0.95)   # keep components explaining 95% of variance
> X_reduced = pca.fit_transform(X_scaled)
> 
> print(pca.explained_variance_ratio_)   # variance share per component
> print(pca.n_components_)                # final number of components
> ```

> [!important] Pros and Cons **Pros:** reduces dimensionality (speeds up training, fights the "curse of dimensionality"), removes multicollinearity (components are orthogonal), useful for visualizing high-dimensional data (2D/3D). **Cons:** new components lose direct interpretability (they're linear combinations of the original features, not the features themselves), assumes linear relationships, requires scaling before applying. **When to use:** visualizing high-dimensional data, preprocessing before modeling with many correlated features, data compression.

---

## How to Choose an Algorithm

> [!formula] Algorithm Selection Cheat Sheet
> 
> |Task / Situation|Recommended Algorithm|
> |---|---|
> |Interpretability required|Linear/logistic regression, decision tree|
> |Maximum quality on tabular data|Gradient boosting (XGBoost/LightGBM/CatBoost)|
> |Small dataset, need a quick baseline|Logistic regression, Naive Bayes, KNN|
> |Lots of noise/outliers|Random Forest, decision trees|
> |Text classification|Naive Bayes, logistic regression on TF-IDF|
> |Complex nonlinear boundaries, medium data size|SVM with RBF kernel, gradient boosting|
> |Unlabeled segmentation|K-Means, hierarchical clustering|
> |Complex-shaped clusters / anomaly detection|DBSCAN|
> |Many correlated features|PCA + any model, Ridge regression|

---

## Model Evaluation Metrics

## Regression Metrics

> [!formula] Formulas
> 
> - **MAE** (Mean Absolute Error): $\frac{1}{n}\sum|y_i - \hat{y}_i|$ — in original units, robust to outliers
> - **MSE** (Mean Squared Error): $\frac{1}{n}\sum(y_i - \hat{y}_i)^2$ — heavily penalizes large errors
> - **RMSE**: $\sqrt{MSE}$ — in original units, sensitive to outliers
> - **$R^2$** (coefficient of determination): share of explained variance, from $-\infty$ to 1 (1 = perfect model, 0 = no better than the mean)
> - **MAPE** (Mean Absolute Percentage Error): average error in percent — convenient for comparing across different scales

> [!example] Example
> 
> ```python
> from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
> 
> mae = mean_absolute_error(y_test, y_pred)
> rmse = mean_squared_error(y_test, y_pred, squared=False)
> r2 = r2_score(y_test, y_pred)
> ```

## Classification Metrics

> [!definition] Confusion Matrix
> 
> ||Predicted 0|Predicted 1|
> |---|---|---|
> |**Actual 0**|TN|FP|
> |**Actual 1**|FN|TP|

> [!formula] Formulas
> 
> - **Accuracy**: $\frac{TP+TN}{TP+TN+FP+FN}$ — share of correct predictions, misleading under class imbalance
> - **Precision**: $\frac{TP}{TP+FP}$ — of those predicted positive, how many actually are
> - **Recall**: $\frac{TP}{TP+FN}$ — of the actual positives, how many were found
> - **F1-score**: $2 \cdot \frac{Precision \cdot Recall}{Precision + Recall}$ — harmonic mean of precision and recall
> - **ROC-AUC**: area under the ROC curve (TPR vs FPR at various thresholds) — robust to class imbalance
> - **PR-AUC**: area under the precision-recall curve — more informative than ROC-AUC under strong imbalance
> - **Log Loss**: penalizes confident but wrong probabilistic predictions

> [!example] Example
> 
> ```python
> from sklearn.metrics import (
>     accuracy_score, precision_score, recall_score, f1_score,
>     roc_auc_score, confusion_matrix, classification_report, log_loss
> )
> 
> accuracy_score(y_test, y_pred)
> precision_score(y_test, y_pred)
> recall_score(y_test, y_pred)
> f1_score(y_test, y_pred)
> roc_auc_score(y_test, y_proba)     # requires probabilities, not classes!
> confusion_matrix(y_test, y_pred)
> print(classification_report(y_test, y_pred))   # all metrics per class at once
> ```

> [!important] Key Takeaway: When to Use What
> 
> - **Class imbalance** (fraud, rare diseases) → accuracy is misleading; use **Precision/Recall/F1/PR-AUC**
> - **Unequal cost of false positives vs. false negatives** → choose the metric reflecting business priority: for medical diagnosis, **Recall** is critical (don't miss the disease); for a spam filter, **Precision** matters more (don't block a wanted email)
> - **Need a threshold-independent metric** → **ROC-AUC** or **PR-AUC**

> [!warning] Common Mistake Using accuracy as the primary metric under strong class imbalance — a model that always predicts "not fraud" when 1% of transactions are fraudulent gets 99% accuracy but is completely useless.

## Clustering Metrics

> [!formula] Formulas (unsupervised, no labels)
> 
> - **Silhouette Score**: -1 to 1, measures how close a point is to its own cluster relative to neighboring clusters (higher is better)
> - **Inertia** (within-cluster sum of squares): used in the elbow method to choose k for K-Means
> - **Davies-Bouldin Index**: lower is better; measures cluster separation

> [!example] Example
> 
> ```python
> from sklearn.metrics import silhouette_score, davies_bouldin_score
> 
> silhouette_score(X_scaled, labels)
> davies_bouldin_score(X_scaled, labels)
> ```

---

## Cross-Validation and Hyperparameter Tuning

> [!definition] Definition **Cross-validation** provides a more reliable estimate of model quality: the data is split into k parts (folds); the model trains on k-1 parts and validates on the remaining one, repeated k times with the metric averaged — reducing dependence of the estimate on a single random train/test split.

> [!example] Example
> 
> ```python
> from sklearn.model_selection import cross_val_score, KFold, GridSearchCV, RandomizedSearchCV
> 
> cv = KFold(n_splits=5, shuffle=True, random_state=42)
> scores = cross_val_score(model, X, y, cv=cv, scoring='f1')
> print(scores.mean(), scores.std())
> 
> # GridSearchCV — exhaustive search over a hyperparameter grid
> param_grid = {'max_depth': [3, 5, 7], 'n_estimators': [100, 200, 300]}
> grid = GridSearchCV(RandomForestClassifier(), param_grid, cv=5, scoring='f1', n_jobs=-1)
> grid.fit(X_train, y_train)
> print(grid.best_params_, grid.best_score_)
> 
> # RandomizedSearchCV — random search (faster with a large grid)
> random_search = RandomizedSearchCV(RandomForestClassifier(), param_grid, n_iter=20, cv=5)
> ```

> [!important] Key Takeaway For imbalanced classes, use `StratifiedKFold` instead of plain `KFold` so each fold preserves the original class ratio.

---

## Common Interview Questions (EN)

> [!important] Cheat Sheet
> 
> 1. What's the difference between bagging and boosting?
> 2. Why is Random Forest more resistant to overfitting than a single tree?
> 3. What's the difference between L1 and L2 regularization?
> 4. When is accuracy a bad metric, and what should you use instead?
> 5. What is data leakage, and how do you avoid it when scaling?
> 6. How does ROC-AUC differ from PR-AUC, and when would you prefer the latter?
> 7. Why is cross-validation needed instead of a single train/test split?
> 8. How do you choose the number of clusters k in K-Means (elbow method, silhouette)?
> 9. Why is feature scaling important for KNN and SVM, but not for trees/forests?
> 10. What's the difference between bias and variance, and how do overfitting/underfitting relate to them?

---

## Related Notes

- [[Statistics & A/B Testing — Interview Study Notes]]
- [[Python for Analysts — Interview Study Notes]]
- [[SQL Interview Study Notes]]
- [[Probability Theory — Interview Study Notes]]
- [[Product Metrics]]
- [[Feature Engineering]]