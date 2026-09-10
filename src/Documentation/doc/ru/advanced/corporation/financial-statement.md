# Финансовая отчётность

## Совокупные активы

`TotalAssets` — это сумма:

- Средств (funds).
- По каждому подразделению:
  - `RecoupableValue` подразделения. Это половина суммы:
    - Стартовой стоимости отрасли.
    - По каждому городу, в который расширилось подразделение (кроме Sector-12):
      - Начальной стоимости офиса.
      - Начальной стоимости склада.
  - Выходных материалов: `material.stored * material.averagePrice`.
  - Продуктов: `product.stored * product.productionCost`.

Это значение отслеживается через `TotalAssets` и `PreviousTotalAssets`.

Средства увеличиваются/уменьшаются функциями `gainFunds`/`loseFunds` при каждом «действии» (покупка чая, вечеринка, покупка апгрейда и т. д.). Каждое действие относится либо к «долгосрочным» (`FundsSourceLongTerm`), либо к «краткосрочным» (`FundsSourceShortTerm`). Если действие «долгосрочное», оно изменяет `totalAssets`.

```typescript
if (LongTermFundsSources.has(source)) {
  this.totalAssets += amt;
}
this.funds += amt;
```

`FundsSourceLongTerm` и `FundsSourceShortTerm` находятся в `FundsSource.ts`.

## Оценка стоимости

Оценка стоимости за цикл:

- AssetDelta:

$$
AssetDelta = \frac{TotalAssets - PreviousTotalAssets}{10}
$$

- До IPO:
  - Если `AssetDelta` больше 0, она участвует в расчёте оценки стоимости.
  - Формула:
    $$Valuation = \left( 10^{10} + \frac{Funds}{3} + AssetDelta\ast 315000 \right)\ast\left( \sqrt[12]{1.1} \right)^{NumberOfOfficesAndWarehouses}$$
  - Оценка округляется вниз до ближайшего миллиона.
- После IPO:
  - На `AssetDelta` влияет `DividendRate`:
    $$AssetDelta = AssetDelta\ast(1 - DividendRate)$$
  - Формула:

$$
Valuation = (Funds + AssetDelta\ast 85000)\ast\left(\sqrt[12]{1.1}\right)^{NumberOfOfficesAndWarehouses}
$$

- Минимальное значение оценки — $10^{10}$.
- Оценка умножается на `CorporationValuation`. Многие BitNode режут корпорацию именно через этот множитель.

Оценка стоимости корпорации — это среднее значение оценок за последние 10 циклов.

Подкуп фракции ради репутации открывается, когда оценка корпорации достигает 100e12. Курс обмена: 1e9 за единицу репутации.

## Инвестиционное предложение

Всего 4 инвестиционных раунда.

У каждого раунда свои `FundingRoundShares` и `FundingRoundMultiplier`.

- $FundingRoundShares = [0.1, 0.35, 0.25, 0.2]$
- $FundingRoundMultiplier = [3, 2, 2, 1.5]$

Формула:

$$
Offer = CorporationValuation\ast FundingRoundShares\ast FundingRoundMultiplier
$$

Разбор:

- Предложение зависит от `Funds`, `AssetDelta` и `NumberOfOfficesAndWarehouses`.
  - `Funds` обычно уходят на улучшение подразделений.
  - `NumberOfOfficesAndWarehouses` — показатель степени в множителе; его можно нарастить, создавая [фиктивные подразделения](./miscellany.md). Это простой способ поднять предложение в раунде 3 и далее, когда средств на это уже хватает.
  - `AssetDelta` умножается на 315000, поэтому она и есть главный источник предложения.
- Если считать, что мы продаём все произведённые единицы и не докупаем бустовые материалы, то `AssetDelta` — это прирост средств, а прирост средств — это прибыль. Вот почему мы изо всех сил стараемся увеличить прибыль.

## Дивиденды и Tribute modifier

Ваши дивиденды режет штрафной модификатор `TributeModifier`. `TributeModifier` зависит от `CorporationSoftcap`. В BN3 `CorporationSoftcap` равен 1.

$$
TributeModifier = 1.15 - CorporationSoftcap
$$

`ShadyAccounting` снижает `TributeModifier` на 0.05.

`GovernmentPartnership` снижает `TributeModifier` на 0.1.

Формула:

$$
TotalDividends = DividendRate\ast(Revenue - Expenses)\ast 10
$$

$$
Dividend = \left(OwnedShares\ast\frac{TotalDividends}{TotalShares}\right)^{1 - TributeModifier}
$$

Нераспределённая прибыль:

$$
RetainedEarning = (1 - DividendRate)\ast(Revenue - Expenses)\ast 10
$$

Дивиденды идут в деньги игрока. Нераспределённая прибыль идёт в средства корпорации. Это значит, что при повышении `DividendRate` оценка стоимости корпорации проседает.

## Акции

Собственное финансирование:

- Стоит 150b.
- Всего акций: 1b.
- Изначально во владении: 1b.

Стартовый капитал (seed money):

- Денег не стоит.
- Всего акций: 1.5b.
- Изначально во владении: 1b.

В каждом инвестиционном раунде инвесторы забирают процент от изначально принадлежащих вам акций. Процент для каждого раунда задан в `FundingRoundShares`.

Если ваша корпорация на собственном финансировании и вы продаёте должность CEO, на создание следующей корпорации понадобится всего 50b.

`TargetSharePrice`:

$$
OwnershipPercentage = \frac{OwnedShares}{TotalShares}
$$

$$
TargetSharePrice = \frac{CorporationValuation*\left(0.5+\sqrt{OwnershipPercentage}\right)}{TotalShares}
$$

Когда корпорация выходит на биржу, начальная цена акции равна `TargetSharePrice`.

Цена акции обновляется в состоянии START.

$$
SharePrice = \begin{cases} SharePrice\ast(1 + Math.random()\ast 0.01), & SharePrice \leq TargetSharePrice \newline SharePrice\ast(1 - Math.random()\ast 0.01), & SharePrice > TargetSharePrice\end{cases}
$$

Минимальная цена акции — 0.01.

Выпуск новых акций:

- Максимальное количество новых акций — 20% от общего числа акций.
- Количество выпускаемых новых акций должно быть кратно 10 миллионам.
- Цена новых акций:

$$
NewOwnershipPercentage = \frac{OwnedShares}{TotalShares+NewShares}
$$

$$
NewSharePrice = \frac{CorporationValuation\ast\left(0.5+\sqrt{NewOwnershipPercentage}\right)}{TotalShares}
$$

- Выручка:

$$
Profit = {NewShares\ast(SharePrice + NewSharePrice)}\ast{0.5}
$$

- Выручка идёт в средства корпорации.
- `DefaultCooldown` — 4 часа.
- Перезарядка:

$$
Cooldown = DefaultCooldown\ast\frac{TotalShares}{10^{9}}
$$

- Часть новых акций добавляется к `InvestorShares`. Остальные — к `IssuedShares`.
  - `MaxPrivateShares`:
    $$MaxPrivateShares = {NewShares}\ast{0.5}\ast\frac{InvestorShares}{TotalShares}$$
  - `PrivateShares` выбирается случайно между 0 и `MaxPrivateShares` и округляется до ближайших 10 миллионов.
  - `InvestorShares`:
    $$InvestorShares = InvestorShares + PrivateShares$$
  - `IssuedShares`:
    $$IssuedShares = IssuedShares + NewShares - PrivateShares$$

Продажа акций:

- Продать все свои акции нельзя.
- За раз нельзя продать больше $10^{14}$ акций.
- Перезарядка — 1 час.
- Проданные акции добавляются к `IssuedShares`.

Выкуп акций:

- Выкупать можно только выпущенные акции. Акции, принадлежащие государству (если вы использовали seed money) и инвесторам, выкупить нельзя.
- Акции выкупаются с премией 10% к рыночной цене.
- Использовать средства корпорации для выкупа акций нельзя. Выкуп идёт за ваши личные деньги.
- За раз нельзя выкупить больше $10^{14}$ акций.

Продажа и выкуп акций обрабатываются в несколько «итераций».

- Количество акций, обрабатываемых за одну итерацию, задаётся shareSalesUntilPriceUpdate. Значение по умолчанию — $10^6$.
- Цена акции пересчитывается на каждой итерации.

$$
OwnershipPercentage = \frac{OwnedShares - ProcessedShares}{TotalShares}
$$

$$
TargetSharePrice = \frac{CorporationValuation\ast\left(0.5 + \sqrt{OwnershipPercentage}\right)}{TotalShares}
$$

$$
SharePrice = \begin{cases} SharePrice\ast 1.005, SharePrice \leq TargetSharePrice \newline SharePrice\ast 0.995, SharePrice > TargetSharePrice\end{cases}
$$
