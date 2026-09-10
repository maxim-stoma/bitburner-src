# Склад

После покупки склад имеет уровень 1. Начальная цена — 5e9.

`BasePrice` в формулах ниже — это базовая цена апгрейда (1e9), а не начальная цена, указанная выше.

Стоимость улучшения склада: формула немного отличается от других апгрейдов (показатель степени равен `CurrentLevel+1`, а не `CurrentLevel`):

$$
UpgradeCost = BasePrice\ast{1.07}^{CurrentLevel + 1}
$$

Стоимость улучшения с уровня 1 до уровня n:

$$
UpgradeCost_{From\ 1\ to\ n} = \sum_{k = 2}^{n}{BasePrice\ast {1.07}^k}
$$

≡

$$
UpgradeCost_{From\ 1\ to\ n} = BasePrice\ast\left( \frac{{1.07}^{n + 1} - {1.07}^{2}}{0.07} \right)
$$

Стоимость улучшения с уровня a до уровня b:

$$
UpgradeCost_{From\ a\ to\ b} = BasePrice\ast\left( \frac{{1.07}^{b + 1} - {1.07}^{a + 1}}{0.07} \right)
$$

Максимальный уровень при заданном `MaxCost`:

$$
MaxLevel = (log_{1.07}\left(MaxCost\ast\frac{0.07}{BasePrice} + {1.07}^{CurrentLevel+1} \right)) - 1
$$

Вместимость склада:

- Множитель апгрейда: множитель от Smart Storage.
- Множитель исследований: множитель от исследований.

$$
WarehouseSize = WarehouseLevel\ast 100\ast UpgradeMultiplier\ast ResearchMultiplier
$$
