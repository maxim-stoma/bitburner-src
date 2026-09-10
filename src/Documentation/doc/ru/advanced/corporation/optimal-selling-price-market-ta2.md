# Оптимальная цена продажи и Market-TA2

## Рыночная цена и предел наценки

Рыночная цена:

- Материал: `material.marketPrice`.
- Продукт: `product.productionCost`. Это значение зависит от `ProductMarketPriceMult`, а также от `MarketPrice` и `Coefficient` входных материалов.
  - $n = {Количество\ входных\ материалов}$
  - $ProductMarketPriceMult = 5$

$$
ProductMarketPrice = ProductMarketPriceMult\ast\sum_{i = 1}^{n}{MaterialMarketPrice_i\ast MaterialCoefficient_i}
$$

Предел наценки (markup limit): насколько выше рыночной цены можно поднять цену, прежде чем это начнёт бить по объёму продаж.
Например: пусть у продукта MarketPrice = 5000 и MarkupLimit = 700. Если выставить цену не выше 5700, объём продаж этого продукта штрафоваться не будет.

- Материал:

$$
MaterialMarkupLimit = \frac{MaterialQuality}{MaterialMarkup}
$$

- Продукт:

$$
ProductMarkupLimit = \frac{Max(ProductEffectiveRating,0.001)}{ProductMarkup}
$$

## Объём продаж

`MaxSalesVolume` — максимальное количество единиц, которое можно продать в состоянии SALE.

`PotentialSalesVolume` — теоретический объём продаж.

`MarkupMultiplier` задаётся кусочной функцией от цены продажи, рыночной цены и предела наценки.

$$
MaxSalesVolume = PotentialSalesVolume\ast MarkupMultiplier
$$

### Потенциальный объём продаж

`PotentialSalesVolume` зависит от:

- Качества материалов и эффективного рейтинга продуктов.
- Количества сотрудников Business.
- Advert.
- Спроса и конкуренции.
- ABC SalesBots.

Это произведение 6 множителей:

$$
PotentialSalesVolume = \ ItemMultiplier\ast BusinessFactor\ast AdvertFactor\ast MarketFactor\ast SaleBotsBonus\ast ResearchBonus
$$

- Множитель качества/эффективного рейтинга:
  - Материал:
    $$ItemMultiplier = MaterialQuality + 0.001$$
  - Продукт:
    $$ItemMultiplier = 0.5\ast(ProductEffectiveRating)^{0.65}$$
- Фактор Business:
  - `BusinessProduction = 1 + office.employeeProductionByJob["Business"]`

$$
{BusinessFactor = (BusinessProduction)}^{0.26} + \left({BusinessProduction}\ast{0.0001}\right)
$$

- Фактор Advert:

$$
AwarenessFactor = (Awareness + 1)^{IndustryAdvertisingFactor}
$$

$$
PopularityFactor = (Popularity + 1)^{IndustryAdvertisingFactor}
$$

$$
RatioFactor = \begin{cases}Max(0.01,\frac{Popularity + 0.001}{Awareness}), & Awareness \neq 0 \newline 0.01, & Awareness = 0 \end{cases}
$$

$$
AdvertFactor = (AwarenessFactor\ast PopularityFactor\ast RatioFactor)^{0.85}
$$

- Рыночный фактор:

$$
MarketFactor = Max\left(0.1,{Demand\ast(100 - Competition)}\ast{0.01}\right)
$$

- Бонус от апгрейда корпорации: бонус `SalesBots`.
- Бонус от исследований подразделения: всегда равен 1. На данный момент нет ни одного исследования, повышающего бонус к продажам.

### Множитель наценки

$$
MarkupMultiplier = \begin{cases}10^{12} & SellingPrice \in (-\infty, 0] \newline \frac{MarketPrice}{SellingPrice} & SellingPrice \in (0, MarketPrice] \newline 1 & SellingPrice \in (MarketPrice, MarketPrice + MarkupLimit] \newline \left(\frac{MarkupLimit}{SellingPrice - MarketPrice}\right)^{2} & SellingPrice \in (MarketPrice + MarkupLimit, \infty) \end{cases}
$$

Разбор всех 4 диапазонов в том же порядке, что и в формуле выше:

- Диапазон 1: можно выставить `SellingPrice` в 0 и получить колоссальный `MarkupMultiplier`. С таким значением продаётся всё, независимо от прочих факторов. Это самый быстрый способ избавиться от накопленных единиц.
- Диапазон 2: `MarkupMultiplier` работает как «бонусный множитель». Он увеличивает `PotentialSalesVolume`. То есть объём продаж можно поднять, выставив `SellingPrice` ниже `MarketPrice`.
- Диапазон 3: `MaxSalesVolume` = `PotentialSalesVolume` (ни бонуса, ни штрафа). Market TA1 всегда выставляет `SellingPrice` равным `MarketPrice + MarkupLimit`. То есть вы продаёте по цене выше рыночной, не портя при этом объём продаж.
- Диапазон 4: `MarkupMultiplier` работает как штрафной модификатор. Подробнее об этом случае ниже.

### Максимизация объёма продаж

Чтобы увеличить `MaxSalesVolume`, можно:

- Повысить качество материалов и эффективный рейтинг продуктов.
- Задействовать больше сотрудников Business.
- Поднять уровень Advert.
- Поднять уровень ABC SalesBots.
- Выставить цену ниже рыночной. Учтите, что в большинстве случаев так делать НЕ следует. Если вам это понадобилось, скорее всего, ваша стратегия ошибочна и её нужно чинить.

## Оптимальная цена продажи

Допустим, мы хотим продать все накопленные единицы. Определим:

$$
ExpectedSalesVolume = \frac{StoredUnits}{10}
$$

Предположим, что мы можем продать все накопленные единицы.

$$
MaxSalesVolume = ExpectedSalesVolume
$$

≡

$$
PotentialSalesVolume\ast MarkupMultiplier = ExpectedSalesVolume
$$

≡

$$
PotentialSalesVolume\ast\left(\frac{MarkupLimit}{SellingPrice - MarketPrice}\right)^{2} = ExpectedSalesVolume
$$

≡

$$
\frac{MarkupLimit}{SellingPrice - MarketPrice} = \sqrt{\frac{ExpectedSalesVolume}{PotentialSalesVolume}}
$$

≡

$$
SellingPrice = \frac{MarkupLimit\ast\sqrt{PotentialSalesVolume}}{\sqrt{ExpectedSalesVolume}} + MarketPrice
$$

Есть 2 случая:

- Когда `PotentialSalesVolume` > `ExpectedSalesVolume`, мы можем смириться со штрафным модификатором (`MarkupMultiplier` < 1) и поднять цену выше `MarketPrice + MarkupLimit`.
- Когда `PotentialSalesVolume` <= `ExpectedSalesVolume`: `MarketPrice` <= `SellingPrice` <= `MarketPrice + MarkupLimit`.
  - Цена продажи всё равно выше рыночной.
  - Штрафного модификатора нет. В этом случае мы и так не можем продать всё, поэтому отсутствие штрафа означает, что хуже, по крайней мере, не станет.

Именно это и делает Market-TA2. Он исходит из того, что мы можем без проблем продать все накопленные единицы (`PotentialSalesVolume` > `ExpectedSalesVolume`) и готовы принять штрафной модификатор. Если это так, он принимает `MaxSalesVolume = ExpectedSalesVolume`, «эксплуатирует» диапазон 4 из предыдущей части и находит максимально возможную цену. В противном случае цена оказывается в диапазоне 3, и `MaxSalesVolume` не страдает.

По этой же причине не стоит возиться с Market-TA1. Он просто выставляет `SellingPrice = MarketPrice + MarkupLimit`. То есть Market-TA1 лишь задаёт «безопасную» `SellingPrice` и гарантирует, что вас не оштрафуют за слишком высокую цену. Но в большинстве случаев (высококачественные материалы, хорошие продукты, высокий Advert и т. д.) `PotentialSalesVolume` намного выше `ExpectedSalesVolume`. Тогда «безопасная» `SellingPrice` от Market-TA1 оказывается слишком низкой, и с Market-TA2 можно найти куда более высокую `SellingPrice`.

Чтобы воспользоваться формулой Market-TA2, нам нужен `MarkupLimit`. Для продуктов для расчёта `MarkupLimit` нужен `ProductMarkup`, но `ProductMarkup` недоступен через NS API. Есть два решения:

- Вычислить приближённое значение. Как это сделать, см. в предыдущем разделе.
- Вычислить `MarkupLimit` напрямую:
  - Выставить `SellingPrice` в очень большое значение. Настолько большое, чтобы мы не могли продать все произведённые единицы (`MaxSalesVolume < ExpectedSalesVolume`). Это заставит игру применить штрафной модификатор, содержащий `MarkupLimit`.
  - Подождать 1 цикл, чтобы получить `ActualSalesVolume`. Это `product.actualSellAmount` и `material.actualSellAmount`.
  - Подставить `ActualSalesVolume` вместо `ExpectedSalesVolume` в формулу выше: $MarkupLimit = (SellingPrice - MarketPrice)\ast\sqrt{\frac{ActualSalesVolume}{PotentialSalesVolume}}$
  - Вычислить `ProductMarkup` из `MarkupLimit` и сохранить `ProductMarkup` для дальнейшего использования. `ProductMarkup` никогда не меняется.
