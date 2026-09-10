# Качество

## Базовые понятия

Определим несколько терминов:

- `AvgInputQuality` — среднее качество входных материалов.
- `MaxOutputQuality` — максимальное значение качества выходного материала.
- `OutputQuality` — итоговое качество выходного материала. Это значение всегда меньше либо равно `MaxOutputQuality`.
- `MaxOutputRating` — максимальное значение рейтинга выходного продукта.
- `OutputRating` — итоговый рейтинг выходного продукта. Это значение всегда меньше либо равно `MaxOutputRating`.

У каждой отрасли свой набор входных материалов и их коэффициентов: например, Agriculture нужны Water и Chemicals с коэффициентами [0.5, 0.2] соответственно. На `AvgInputQuality` эти коэффициенты не влияют. `AvgInputQuality` — это среднее арифметическое качеств входных материалов, лежащих на складе. Например, если на складе подразделения Agriculture есть Water (качество 1) и Chemicals (качество 11), то `AvgInputQuality` равно (1+11)/2.

Купленный материал низкокачественный. Его качество всегда равно 1.

При импорте/экспорте материалов между подразделениями видно, что качество некоторых входных материалов постоянно меняется. После состояния EXPORT качество высокое, но после состояния PURCHASE оно снижается. Качества материалов на складе пересчитываются именно в этих 2 состояниях.

В состоянии PURCHASE качество материала «разбавляется» низкокачественным купленным материалом (качество 1).

$$
Quality = \frac{Quality\ast CurrentQuantity + BuyAmount}{CurrentQuantity + BuyAmount}
$$

В состоянии PRODUCTION для расчёта `AvgInputQuality` используется уже «разбавленное» значение качества.

В состоянии EXPORT:

$$
Quality = \frac{Quality\ast CurrentQuantity + ImportQuality\ast ImportAmount}{CurrentQuantity + ImportAmount}
$$

Производственная мощность вспомогательного подразделения должна быть сбалансированной. `ImportAmount` (количество единиц материала, которое экспортирует вспомогательное подразделение) не обязано равняться требуемому количеству единиц входного материала, но и слишком маленьким быть не должно.

## Материалы

`MaxOutputQuality` — это сумма 3 слагаемых:

- Слагаемое от инженеров:
  - `EngineerProduction = office.employeeProductionByJob["Engineer"]`

$$
EngineerSummand = \frac{EngineerProduction}{90}
$$

- Слагаемое от очков исследований:

$$
ResearchPointSummand = (RP)^{IndustryScienceFactor}
$$

- Слагаемое от AI Cores (если AI Cores есть на складе):

$$
AICoresSummand = AICoresQuantity^{IndustryAICoreFactor}\ast{0.001}
$$

Качество на выходе:

$$
OutputQuality = \sqrt{MaxOutputQuality}\ast AvgInputQuality
$$

Из формул выше следуют такие выводы:

- В ранних раундах лучший способ повысить `MaxOutputQuality` — наращивать RP. Особенно это верно для отраслей с высоким science factor, таких как Chemical.
- В поздних раундах (3 и далее) у нас достаточно средств, чтобы улучшать офисы. В этом случае важнейший фактор `MaxOutputQuality` — это `EngineerProduction`. Должность «Engineer» значимее, чем «Research & Development».
- `OutputQuality` начинается с квадратного корня из `MaxOutputQuality`. `AvgInputQuality` поднимает его, пока оно не достигнет `MaxOutputQuality`.
- Простая стратегия проверки, нужно ли повышать `AvgInputQuality`:
  - Если квадрат `AvgInputQuality` больше либо равен текущему выходному качеству — всё в порядке.
  - Если нет — нужно повышать качество входных материалов. Обычно это означает, что нужно улучшать вспомогательное подразделение.

## Продукты

`MaxOutputRating` — это product.rating.

В интерфейсе игры `OutputRating` отображается как «Effective rating».

Рейтинг на выходе:

$$
OutputRating = \sqrt{MaxOutputRating}\ast AvgInputQuality
$$

Для проверки `AvgInputQuality` используйте ту же стратегию, что и для материалов.
