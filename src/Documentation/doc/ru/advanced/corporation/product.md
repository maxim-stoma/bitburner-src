# Продукты

## Обзор

На поздних фазах продуктовые отрасли гораздо выгоднее материальных, потому что продукты можно продавать по неприлично высоким ценам.

Market-TA2 автоматически выставляет оптимальные цены на продукты. Как написать собственный скрипт Market-TA2, см. в [этом разделе](./optimal-selling-price-market-ta2.md). Собственный скрипт Market-TA2 — лучшая оптимизация в раунде 3 и далее.

Прежде чем производить продукт, его нужно разработать. В очередь разработки можно поставить несколько продуктов, но одновременно разрабатывается только один.

Количество продуктов у подразделения ограничено. По умолчанию предел — 3. Есть 2 исследования, повышающие этот предел. Но они почти бесполезны, поскольку в большинстве случаев увеличивать максимальное число продуктов смысла нет. Подробнее — в [этом разделе](./unlocks-upgrade-research.md). Достигнув предела, нужно снять с производства один продукт, прежде чем разрабатывать новый.

Новые продукты нужно разрабатывать непрерывно. Новые продукты почти всегда лучше старых и приносят намного больше прибыли.

Наценка и эффективный рейтинг продукта исключительно важны, поскольку участвуют в расчёте [MaxSalesVolume](./optimal-selling-price-market-ta2.md).

Эффективный рейтинг продукта зависит от рейтинга продукта и качества входных материалов. Как качество входных материалов влияет на рейтинг и эффективный рейтинг продукта, см. в [этом разделе](./quality.md). Именно поэтому нужно вспомогательное подразделение, производящее высококачественный материал для продуктового подразделения.

Наценка и рейтинг продукта зависят от:

- `CreationJobFactors[JobName]`. Подробнее об этом в следующей части.
- RP. Вот почему RP стоит копить.
- `ResearchFactor`. Это `scienceFactor` отрасли.
- Инвестиций в дизайн и рекламу. В интерфейсе игры это «Design investment» и «Marketing investment». Эти две инвестиции не слишком важны, потому что показатели степени у них в формулах очень малы. Достаточно тратить на них 1% текущих средств.

Апгрейды офиса и характеристик сотрудников очень важны для продуктов, поскольку повышают производительность сотрудников. Высокая производительность сотрудников означает высокие `CreationJobFactors` и RP. Апгрейды и продукты образуют мощный цикл: больше апгрейдов → лучше продукт → выше прибыль → больше апгрейдов.

Правильная настройка офиса важна для эффективной разработки новых продуктов. Советы по настройке офиса — в [этом разделе](./general-advice.md).

## Формула

`CreationJobFactors[JobName]` — это значения, накапливаемые за всё время разработки продукта. `DevelopmentProgress` начинается с 0. В каждом цикле:

- Суммарная производительность сотрудников:

$$
TotalEmployeeProd = OperationsProd + EngineerProd + ManagementProd
$$

- Фактор менеджмента:

$$
ManagementFactor = 1 + \frac{ManagementProd}{1.2\ast TotalEmployeeProd}
$$

- Множитель разработки продукта:

$$
ProductDevelopmentMultiplier = \left( (EngineerProd)^{0.34} + (OperationsProd)^{0.2} \right)\ast ManagementFactor
$$

- Прогресс:

$$
Progress = 0.01\ast ProductDevelopmentMultiplier
$$

- Прогресс разработки:

$$
DevelopmentProgress = DevelopmentProgress + Progress
$$

- `CreationJobFactors[JobName]`:

$$
CreationJobFactors\lbrack JobName\rbrack = CreationJobFactors\lbrack JobName\rbrack + {\lbrace EmployeeJob\rbrace Prod\ast Progress}\ast{0.01}
$$

&nbsp;  
Когда `DevelopmentProgress` достигает 100, продукт готов.

- Обозначим:

$$
A = \ CreationJobFactors\lbrack Engineer\rbrack
$$

$$
B = \ CreationJobFactors\lbrack Management\rbrack
$$

$$
C = \ CreationJobFactors\lbrack RnD\rbrack
$$

$$
D = \ CreationJobFactors\lbrack Operations\rbrack
$$

$$
E = \ CreationJobFactors\lbrack Business\rbrack
$$

$$
TotalCreationJobFactors = A + B + C + D + E
$$

- {JobName}Ratio (доли должностей):

$$
EngineerRatio = \frac{A}{TotalCreationJobFactors}
$$

$$
ManagementRatio = \frac{B}{TotalCreationJobFactors}
$$

$$
RnDRatio = \frac{C}{TotalCreationJobFactors}
$$

$$
OperationsRatio = \frac{D}{TotalCreationJobFactors}
$$

$$
BusinessRatio = \frac{E}{TotalCreationJobFactors}
$$

- Множитель инвестиций в дизайн:

$$
DesignInvestMult = 1 + {(DesignInvestment)^{0.1}}\ast{0.01}
$$

- Научный множитель:

$$
ScienceMult = 1 + {(RP)^{ResearchFactor}}\ast{0.00125}
$$

- Балансировочный множитель:

$$
BalanceMult = 1.2\ast EngineerRatio + 0.9\ast ManagementRatio + 1.3\ast RnDRatio + 1.5\ast OperationsRatio + BusinessRatio
$$

- Итоговый множитель:

$$
TotalMult = BalanceMult\ast DesignInvestMult\ast ScienceMult
$$

- Quality продукта:

$$
TotalMult\ast (0.1\ast A + 0.05\ast B + 0.05\ast C + 0.02\ast D + 0.02\ast E)
$$

- Performance продукта:

$$
TotalMult\ast (0.15\ast A + 0.02\ast B + 0.02\ast C + 0.02\ast D + 0.02\ast E)
$$

- Durability продукта:

$$
TotalMult\ast (0.05\ast A + 0.02\ast B + 0.08\ast C + 0.05\ast D + 0.05\ast E)
$$

- Reliability продукта:

$$
TotalMult\ast (0.02\ast A + 0.08\ast B + 0.02\ast C + 0.05\ast D + 0.08\ast E)
$$

- Aesthetics продукта:

$$
TotalMult\ast (0.08\ast B + 0.05\ast C + 0.02\ast D + 0.1\ast E)
$$

- Features продукта:

$$
TotalMult\ast (0.08\ast A + 0.05\ast B + 0.02\ast C + 0.05\ast D + 0.05\ast E)
$$

- Рейтинг продукта:
  - У каждой отрасли, производящей продукты, есть свои `RatingWeights` для её продукта. `RatingWeights` содержит коэффициенты 6 характеристик: quality, performance, durability, reliability, aesthetics, features. Например, `RatingWeights` для Tobacco:
    - Коэффициент quality: 0.7.
    - Коэффициент durability: 0.1.
    - Коэффициент aesthetics: 0.2.
  - `RatingWeights` — это `industryData.product.ratingWeights`.
  - Формула:

$$
ProductRating = \sum_{i = 1}^{6}{{ProductStat}_i\ast{StatCoefficient}_i}
$$

- Множитель инвестиций в рекламу:

$$
AdvertInvestMult = 1 + {(AdvertisingInvestment)^{0.1}}\ast{0.01}
$$

- Соотношение Business и Management:

$$
BusinessManagementRatio = Max\left( BusinessRatio + ManagementRatio,\ \left( \frac{1}{TotalCreationJobFactors} \right) \right)
$$

- Наценка продукта:

$$
ProductMarkup = \frac{100}{AdvertInvestMult\ast(ProductQuality + 0.001)^{0.65}\ast BusinessManagementRatio}
$$

- Спрос на продукт:

$$
Demand = \begin{cases}Min(100,AdvertInvestMult\ast(100\ast(Popularity/Awareness))), & Awareness \neq 0 \newline 20, & Awareness = 0 \end{cases}
$$

- Конкуренция по продукту:

$$
Competition = Random(0,70)
$$

- Размер продукта:
  - Это `product.size`.
  - Формула:

$$
ProductSize = \sum_{i = 1}^{NumberOfInputMaterials}{{InputMaterialSize}_i\ast{InputMaterialCoefficient}_i}
$$

## Приближённое значение наценки продукта

Чтобы вычислить наценку продукта, нужны:

- `CreationJobFactors[JobName]`
- `RP`
- `ResearchFactor`
- `DesignInvestment`
- `AdvertisingInvestment`

Наценка продукта вычисляется в момент завершения продукта. И тут есть одна вещь, которую мы получить не можем, — `CreationJobFactors[JobName]`, потому что запросить её через NS API нельзя. Есть 2 подхода к этой проблеме:

- Считать их самостоятельно. То есть моделировать `product.creationJobFactors` у себя. Подход простой, но с серьёзной проблемой: стоит пропустить хотя бы один цикл — и данные становятся неверными.
- Вычислить их напрямую. Характеристики продукта — публичные данные, поэтому по формулам выше получается система из 6 уравнений всего с 5 неизвестными. Найти её решение можно с помощью [Ceres Solver](./miscellany.md).
