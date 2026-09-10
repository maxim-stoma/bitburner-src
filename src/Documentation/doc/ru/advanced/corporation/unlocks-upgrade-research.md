# Unlocks, апгрейды, исследования

## Unlocks

| **Название**              | **Цена** | **Описание**                                                                                                                                              |
| ------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Export                    | 20e9     | Разрешает экспорт материалов между подразделениями. Крайне важен. Покупайте в начале раунда 2.                                                             |
| Smart Supply              | 25e9     | Включает функцию «Smart Supply». Покупайте, только если не пишете собственный скрипт [Smart Supply](./smart-supply.md).                                    |
| Market Research - Demand  | 5e9      | Открывает доступ к данным [Demand](./demand-competition.md). Нужен, чтобы написать свой скрипт [Market-TA2](./optimal-selling-price-market-ta2.md).        |
| Market Data - Competition | 5e9      | Открывает доступ к данным [Competition](./demand-competition.md). Нужен, чтобы написать свой скрипт [Market-TA2](./optimal-selling-price-market-ta2.md).   |
| Shady Accounting          | 500e12   | Снижает [TributeModifier](./financial-statement.md) на 0.05                                                                                                |
| Government Partnership    | 2e15     | Снижает [TributeModifier](./financial-statement.md) на 0.1                                                                                                 |

&nbsp;

## Апгрейды

У каждого апгрейда свои `BasePrice`, `PriceMult`, `Benefit`.

Большинство апгрейдов действуют на все подразделения.

Есть 3 особых апгрейда. Они действуют только на своё подразделение и считаются по другим формулам стоимости и эффекта.

- Warehouse. См. [этот раздел](./warehouse.md).
- Office. См. [этот раздел](./office.md).
- Advert. См. [этот раздел](./wilson-analytics-advert.md).

Формулы обычных апгрейдов:

- Стоимость апгрейда:

$$
UpgradeCost = BasePrice\ast{PriceMult}^{CurrentLevel}
$$

- Стоимость апгрейда с уровня 0 до уровня n:

$$
UpgradeCost_{From\ 0\ to\ n} = \sum_{k = 0}^{n - 1}{BasePrice\ast {PriceMult}^k}
$$

≡

$$
UpgradeCost_{From\ 0\ to\ n} = BasePrice\ast\left( \frac{1 - {PriceMult}^{n}}{1 - PriceMult} \right)
$$

≡

$$
UpgradeCost_{From\ 0\ to\ n} = BasePrice\ast\left( \frac{{PriceMult}^{n} - 1}{PriceMult - 1} \right)
$$

- Стоимость апгрейда с уровня a до уровня b:

$$
UpgradeCost_{From\ a\ to\ b} = \sum_{k = 0}^{b - 1}{BasePrice\ast {PriceMult}^k} - \sum_{k = 0}^{a - 1}{BasePrice\ast {PriceMult}^k}
$$

≡

$$
UpgradeCost_{From\ a\ to\ b} = BasePrice\ast\left( \frac{{PriceMult}^{b} - 1}{PriceMult - 1} \right) - BasePrice\ast\left( \frac{{PriceMult}^{a} - 1}{PriceMult - 1} \right)
$$

≡

$$
UpgradeCost_{From\ a\ to\ b} = BasePrice\ast\left( \frac{{PriceMult}^{b} - {PriceMult}^{a}}{PriceMult - 1} \right)
$$

- Максимальный уровень апгрейда при заданном `MaxCost`:

$$
MaxUpgradeLevel = \log_{PriceMult}\left( MaxCost\ast\frac{PriceMult - 1}{BasePrice} + (PriceMult)^{CurrentLevel} \right)
$$

- Эффект: все эффекты — множители. `BaseBenefit` равен 1.

$$
Benefit = BaseBenefit + Benefit\ast CurrentLevel
$$

&nbsp;  
Обычные апгрейды:

| **Название**                       | **Базовая цена** | **Множитель цены** | **Эффект** | **Тип**                     |
| ---------------------------------- | ---------------- | ------------------ | ---------- | --------------------------- |
| SmartFactories                     | 2e9              | 1.06               | 0.03       | Производство                |
| SmartStorage                       | 2e9              | 1.06               | 0.1        | Вместимость склада          |
| WilsonAnalytics                    | 4e9              | 2                  | 0.005      | Эффект Advert               |
| NuoptimalNootropicInjectorImplants | 1e9              | 1.06               | 0.1        | Креативность сотрудников    |
| SpeechProcessorImplants            | 1e9              | 1.06               | 0.1        | Charisma сотрудников        |
| NeuralAccelerators                 | 1e9              | 1.06               | 0.1        | Интеллект сотрудников       |
| FocusWires                         | 1e9              | 1.06               | 0.1        | Эффективность сотрудников   |
| ABCSalesBots                       | 1e9              | 1.07               | 0.01       | Продажи                     |
| ProjectInsight                     | 5e9              | 1.07               | 0.05       | RP                          |

&nbsp;  
Особые апгрейды:

| **Название** | **Базовая цена** | **Множитель цены** | **Тип**              |
| ------------ | ---------------- | ------------------ | -------------------- |
| Warehouse    | 1e9              | 1.07               | Вместимость склада   |
| Office       | 4e9              | 1.09               | Размер офиса         |
| Advert       | 1e9              | 1.06               | Awareness/Popularity |

&nbsp;  
Советы:

- Раунд 1:
  - SmartStorage и Warehouse — важнейшие апгрейды в этом раунде.
  - Покупайте только 1–2 уровня Advert.
- Раунд 2:
  - SmartFactories, SmartStorage и Warehouse — важнейшие апгрейды в этом раунде.
  - Для подразделения Agriculture купите только 1 уровень Office и пару уровней Advert.
  - Для подразделения Chemical Office/Advert не покупайте.
- Больше советов, особенно для раунда 3 и далее, — в [этом разделе](./general-advice.md).

## Исследования

У каждого исследования свой набор множителей. Например: `sciResearchMult`, `productionMult` и т. д.

Эффект по типу исследования равен произведению всех множителей этого типа от всех исследований.

| **Тип**               | **Исследования**                              | **Множитель** | **Эффект**                 |
| --------------------- | --------------------------------------------- | ------------- | -------------------------- |
| advertisingMult       | Нет исследований                              | 1             | Эффект Advert              |
| employeeChaMult       | CPH4 Injections                               | 1.1           | Charisma сотрудников       |
| employeeCreMult       | CPH4 Injections                               | 1.1           | Креативность сотрудников   |
| employeeEffMult       | CPH4 Injections, Overclock                    | 1.1\*1.25     | Эффективность сотрудников  |
| employeeIntMult       | CPH4 Injections, Overclock                    | 1.1\*1.25     | Интеллект сотрудников      |
| productionMult        | Drones -- Assembly Self-Correcting Assemblers | 1.2\*1.1      | Производство               |
| productProductionMult | uPgrade: Fulcrum                              | 1.05          | Производство продуктов     |
| salesMult             | Нет исследований                              | 1             | Продажи                    |
| sciResearchMult       | Hi-Tech R&D Laboratory                        | 1.1           | RP                         |
| storageMult           | Drones - Transport                            | 1.5           | Вместимость склада         |

&nbsp;  
Список исследований:

| **Название**                  | **Стоимость** | **Описание**                                                                                                                                     |
| ----------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hi-Tech R&D Laboratory        | 5000          | Наивысший приоритет. Ускоряет набор RP. Предварительное условие для всех остальных исследований.                                                  |
| Market-TA.I                   | 20000         | Бесполезен сам по себе. Предварительное условие для Market-TA.II.                                                                                 |
| Market-TA.II                  | 50000         | Наивысший приоритет, если не пишете свой скрипт. Как написать свой — см. [этот раздел](./optimal-selling-price-market-ta2.md).                    |
| Automatic Drug Administration | 10000         | Предварительное условие для Go-Juice и CPH4 Injections.                                                                                           |
| Go-Juice                      | 25000         | Полезно. Повышает максимум энергии.                                                                                                              |
| CPH4 Injections               | 25000         | Полезно. Повышает характеристики сотрудников.                                                                                                    |
| Overclock                     | 15000         | Полезно. Повышает характеристики сотрудников. Предварительное условие для Sti.mu.                                                                 |
| Sti.mu                        | 30000         | Полезно. Повышает максимум морали.                                                                                                               |
| Drones                        | 5000          | Предварительное условие для Drones - Assembly и Drones - Transport.                                                                              |
| Drones - Assembly             | 25000         | Полезно. Увеличивает всё производство.                                                                                                           |
| Drones - Transport            | 30000         | Полезно. Увеличивает вместимость склада.                                                                                                         |
| Self-Correcting Assemblers    | 25000         | Полезно. Увеличивает всё производство.                                                                                                           |
| uPgrade: Fulcrum              | 10000         | Полезно. Увеличивает производство продуктов.                                                                                                     |
| uPgrade: Capacity.I           | 20000         | Не полезно. Слишком дорого для посредственного эффекта. Увеличивает максимальное число продуктов на 1 (с 3 до 4).                                 |
| uPgrade: Capacity.II          | 30000         | Не полезно. Слишком дорого для посредственного эффекта. Увеличивает максимальное число продуктов на 1 (с 4 до 5).                                 |
| uPgrade: Dashboard            | 5000          | Бесполезно.                                                                                                                                      |
| AutoBrew                      | 12000         | Бесполезно.                                                                                                                                      |
| AutoPartyManager              | 15000         | Бесполезно.                                                                                                                                      |
| HRBuddy-Recruitment           | 15000         | Бесполезно.                                                                                                                                      |
| HRBuddy-Training              | 20000         | Бесполезно.                                                                                                                                      |

&nbsp;  
Советы:

- Не опустошайте весь запас RP ради исследований. Покупать исследование стоит, только если оно стоит меньше половины запаса RP. Лично мои условия для покупки исследований такие:
  - Для энергии/морали и характеристик сотрудников: если стоит меньше 20% запаса RP.
  - Для производства: если стоит меньше 10% запаса RP.
- Если у вас нет собственного скрипта Market-TA2, приоритет нужно отдать Market-TA1 и Market-TA2. Сам по себе Market-TA1 бесполезен, покупать его есть смысл лишь как предварительное условие для Market-TA2. Если берёте их, накопите RP и покупайте оба сразу. Но я рекомендую как можно скорее написать свой Market-TA2. Market-TA1 и Market-TA2 стоят 70000 RP, а это огромное количество RP в начале раунда 3. **Собственный скрипт Market-TA2 — лучшая оптимизация в раунде 3 и далее.**
- Затем приоритет стоит отдавать исследованиям на максимум энергии/морали и характеристики сотрудников, а не на производство. Исследования на производство приятны, но куда менее важны, чем энергия/мораль/характеристики сотрудников.
- Мой порядок исследований для повышения максимума энергии/морали и характеристик сотрудников: Overclock → Sti.mu → Automatic Drug Administration → Go-Juice → CPH4 Injections.
- Не покупайте эти бесполезные:
  - uPgrade: Dashboard
  - AutoBrew
  - AutoPartyManager
  - HRBuddy-Recruitment
  - HRBuddy-Training
- В большинстве случаев uPgrade: Capacity.I и uPgrade: Capacity.II бесполезны. Новые продукты обычно намного лучше старых, так что увеличивать максимальное число продуктов смысла нет. Единственное исключение — эндгейм. В эндгейме новые продукты лишь незначительно лучше старых, поэтому дополнительные слоты под продукты могут пригодиться. Но даже в эндгейме эти исследования могут навредить сильнее, чем помочь. В эндгейме вместимость склада и высококачественные входные материалы — серьёзные узкие места. Больше слотов под продукты означает потребность в большем свободном месте на складе и в большем количестве единиц входных материалов. В некоторых случаях увеличение числа слотов под продукты фактически снижает общую прибыль. Тут нужно подбирать под конкретную ситуацию.

Если у вас есть SF9, RP можно выменивать на хеши. Это количество RP добавляется всем подразделениям.

Скорость набора RP:

- RP увеличивается в 4 состояниях: PURCHASE, PRODUCTION, EXPORT и SALE.
- Прирост RP на город за одно состояние:
  - `RnDProduction = office.employeeProductionByJob["Research & Development"]`

$$
RPGain = 0.004\ast(RnDProduction)^{0.5}\ast UpgradeMultiplier\ast ResearchMultiplier
$$

- `ScienceFactor` отрасли на скорость набора RP не влияет.
