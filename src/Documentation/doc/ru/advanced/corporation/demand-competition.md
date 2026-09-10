# Спрос и конкуренция

## Для чего нужны

Они используются при расчёте `MaxSalesVolume` материалов и продуктов.

`Market Research - Demand` открывает доступ к данным `Demand`.

`Market Data - Competition` открывает доступ к данным `Competition`.

## Материалы

У каждого материала свои `demandBase`, `demandRange`, `competitionBase`, `competitionRange` и `maxVolatility`. И `demand`, и `competition` стартуют со своих базовых значений и всегда остаются в пределах соответствующих диапазонов.

Вот что неочевидно: `demand` и `competition` _не_ используются при расчёте `marketPrice`.

В состоянии START игра вычисляет 6 переменных:

```typescript
const priceVolatility: number = (Math.random() * this.maxVolatility) / 300;
const priceChange: number = 1 + priceVolatility;
const compVolatility: number = (Math.random() * this.maxVolatility) / 100;
const compChange: number = 1 + compVolatility;
const dmdVolatility: number = (Math.random() * this.maxVolatility) / 100;
const dmdChange: number = 1 + dmdVolatility;
```

`priceChange`, `compChange` и `dmdChange` — это величины, на которые изменятся `marketPrice`, `competition` и `demand` на следующих шагах.

После этого дважды бросается случайное число:

- Первый бросок: `Math.random()` < 0.5. Если да — `competition` и `marketPrice` увеличиваются. Если нет — уменьшаются.
- Второй бросок: `Math.random()` < 0.5. Если да — `demand` и `marketPrice` увеличиваются. Если нет — уменьшаются.

## Продукты

Начальные значения задаются в момент завершения разработки продукта. Формулы см. в следующем [разделе](./product.md).

В состоянии START игра снижает `demand` и повышает `competition` продукта.

- Величина изменения:

$$
AmountOfChange = Random(0,3)*0.0004
$$

- Эта величина умножается на 3, если отрасль — Pharmaceutical, Software или Robotics.

Минимальное значение `Demand` — 0.001. Максимальное значение `Competition` — 99.99.
