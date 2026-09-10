# Бустовые материалы

## Производственный множитель подразделения

У каждой отрасли свой набор коэффициентов бустовых материалов. Например:

- Agriculture:
  - AI Cores: 0.3
  - Hardware: 0.2
  - Real Estate: 0.72
  - Robots: 0.3
- Chemical:
  - AI Cores: 0.2
  - Hardware: 0.2
  - Real Estate: 0.25
  - Robots: 0.25
- Tobacco:
  - AI Cores: 0.15
  - Hardware: 0.15
  - Real Estate: 0.15
  - Robots: 0.25

Производственный множитель подразделения используется при расчёте [сырого производства подразделения](./division-raw-production.md) в состоянии PRODUCTION. Он равен сумме `cityMult` всех складов, а `cityMult` вычисляется из количества каждого бустового материала и его коэффициента.

В коде этот множитель — `this.productionMult` в `Division.ts`.

```typescript
calculateProductionFactors(): void {
  let multSum = 0;
  for (const warehouse of getRecordValues(this.warehouses)) {
    const materials = warehouse.materials;

    const cityMult =
      Math.pow(0.002 * materials["Real Estate"].stored + 1, this.realEstateFactor) *
      Math.pow(0.002 * materials.Hardware.stored + 1, this.hardwareFactor) *
      Math.pow(0.002 * materials.Robots.stored + 1, this.robotFactor) *
      Math.pow(0.002 * materials["AI Cores"].stored + 1, this.aiCoreFactor);
    multSum += Math.pow(cityMult, 0.73);
  }

  multSum < 1 ? (this.productionMult = 1) : (this.productionMult = multSum);
}
```

Именно поэтому нужно расширяться во все 6 городов. Больше городов → выше `this.productionMult` → выше сырое производство → больше произведённых материалов/продуктов → выше прибыль на город → выше общая прибыль.

Расширение на 6 городов означает, что `this.productionMult` умножается на 6, и городов у нас 6, — то есть производство фактически умножается на 36. Строго говоря, это не совсем так, поскольку на сырое производство каждого города влияет и многое другое, но x36 можно считать грубой оценкой выгоды, особенно в ранних раундах. В этих раундах производственный множитель подразделения — самое важное.

## Оптимизатор

Чтобы увеличить `this.productionMult`, нужно покупать бустовые материалы. Вопрос в том, сколько каждого материала покупать при заданном ограничении на место на складе.

У каждого бустового материала есть коэффициент («factor» в исходном коде) и базовый размер (место, занимаемое 1 единицей на складе).

Введём обозначения:

- 4 коэффициента: ${c_{1}}$, ${c_{2}}$, ${c_{3}}$, ${c_{4}}$
- 4 базовых размера: ${s_{1}}$, ${s_{2}}$, ${s_{3}}$, ${s_{4}}$
- Количества каждого бустового материала: x, y, z, w

Если во всех городах склады настроены одинаково, производственный множитель подразделения равен:

$$
F(x,y,z,w) = \sum_{i = 1}^{6}\left( (1 + 0.002\ast x)^{c_{1}}\ast(1 + 0.002\ast y)^{c_{2}}{\ast(1 + 0.002\ast z)}^{c_{3}}{\ast(1 + 0.002\ast w)}^{c_{4}} \right)^{0.73}
$$

Чтобы найти максимум функции выше, достаточно найти максимум этой функции:

$$
F(x,y,z,w) = (1 + 0.002\ast x)^{c_{1}}\ast(1 + 0.002\ast y)^{c_{2}}{\ast(1 + 0.002\ast z)}^{c_{3}}{\ast(1 + 0.002\ast w)}^{c_{4}}
$$

Функция ограничения (S — место на складе):

$$
G(x,y,z,w) = s_{1}\ast x + s_{2}\ast y + s_{3}\ast z + s_{4}\ast w = S
$$

Задача: найти максимум $F(x,y,z,w)$ при ограничении $G(x,y,z,w)$.

## Решение

### Метод множителей Лагранжа

Дисклеймер: основано на обсуждении между \@Jesus и \@yichizhng в Discord. Вся заслуга принадлежит им.

Применяя метод [множителей Лагранжа](https://en.wikipedia.org/wiki/Lagrange_multiplier), получаем систему:

$$
\begin{cases} \frac{\partial F}{\partial x} &= \lambda\frac{\partial G}{\partial x} \newline \frac{\partial F}{\partial y} &= \lambda\frac{\partial G}{\partial y} \newline \frac{\partial F}{\partial z} &= \lambda\frac{\partial G}{\partial z} \newline \frac{\partial F}{\partial w} &= \lambda\frac{\partial G}{\partial w} \newline G(x,y,z,w) &= S\end{cases}
$$

Решить эту систему можно 2 способами:

- Решить систему с помощью [Ceres Solver](./miscellany.md).
- Проделать работу вручную средствами базового матанализа и алгебры. Это оптимальный путь и по точности, и по производительности, поэтому на нём мы и сосредоточимся. В следующих разделах приведено доказательство этого решения.

$$
x\ast s_{1} = \frac{S - 500\ast\left( \frac{s_{1}}{c_{1}}\ast\left( c_{2} + c_{3} + c_{4} \right) - \left( s_{2} + s_{3} + s_{4} \right) \right)}{\frac{c_{1} + c_{2} + c_{3} + c_{4}}{c_{1}}}
$$

$$
y\ast s_{2} = \frac{S - 500\ast\left( \frac{s_{2}}{c_{2}}\ast\left( c_{1} + c_{3} + c_{4} \right) - \left( s_{1} + s_{3} + s_{4} \right) \right)}{\frac{c_{1} + c_{2} + c_{3} + c_{4}}{c_{2}}}
$$

$$
z\ast s_{3} = \frac{S - 500\ast\left( \frac{s_{3}}{c_{3}}\ast\left( c_{1} + c_{2} + c_{4} \right) - \left( s_{1} + s_{2} + s_{4} \right) \right)}{\frac{c_{1} + c_{2} + c_{3} + c_{4}}{c_{3}}}
$$

$$
w\ast s_{4} = \frac{S - 500\ast\left( \frac{s_{4}}{c_{4}}\ast\left( c_{1} + c_{2} + c_{3} \right) - \left( s_{1} + s_{2} + s_{3} \right) \right)}{\frac{c_{1} + c_{2} + c_{3} + c_{4}}{c_{4}}}
$$

## Доказательство

Обозначим: $k = 0.002$

$$
\begin{cases}\frac{\partial F}{\partial x} = \left( k\ast c_{1}\ast(1 + k\ast x)^{c_{1} - 1} \right)\ast(1 + k\ast y)^{c_{2}}\ast(1 + k\ast z)^{c_{3}}\ast(1 + k\ast w)^{c_{4}} = \lambda\ast s_{1} \newline \frac{\partial F}{\partial y} = (1 + k\ast x)^{c_{1}}\ast\left( k\ast c_{2}\ast(1 + k\ast y)^{c_{2} - 1} \right)\ast(1 + k\ast z)^{c_{3}}\ast(1 + k\ast w)^{c_{4}} = \lambda\ast s_{2} \end{cases}
$$

≡

$$
k\ast c_{1}\ast(1 + k\ast x)^{- 1}\ast s_{2} = k\ast c_{2}\ast(1 + k\ast y)^{- 1}\ast s_{1}
$$

≡

$$
c_{1}\ast s_{2}\ast(1 + k\ast y) = c_{2}\ast s_{1}\ast(1 + k\ast x)
$$

≡

$$
1 + k\ast y = \frac{c_{2}\ast s_{1}}{c_{1}\ast s_{2}}\ast(1 + k\ast x)
$$

≡

$$
y = \frac{c_{2}\ast s_{1} + k\ast x\ast c_{2}\ast s_{1} - c_{1}\ast s_{2}}{k\ast c_{1}\ast s_{2}}
$$

≡

$$
y\ast s_{2} = \frac{c_{2}\ast s_{1}\ast s_{2} + k\ast x\ast c_{2}\ast s_{1}\ast s_{2} - c_{1}\ast s_{2}\ast s_{2}}{k\ast c_{1}\ast s_{2}}
$$

≡

$$
y\ast s_{2} = \frac{c_{2}\ast s_{1}}{k\ast c_{1}} + \frac{x\ast c_{2}\ast s_{1}}{c_{1}} - \frac{s_{2}}{k}
$$

≡

$$
y\ast s_{2} = \frac{c_{2}}{c_{1}}\ast x\ast s_{1} + \frac{1}{k}\ast\frac{c_{2}\ast s_{1} - c_{1}\ast s_{2}}{c_{1}}
$$

≡

$$
y\ast s_{2} = \frac{c_{2}}{c_{1}}\ast x\ast s_{1} + 500\ast\frac{c_{2}\ast s_{1} - c_{1}\ast s_{2}}{c_{1}}
$$

Повторяя те же шаги, получаем:

$$
z\ast s_{3} = \frac{c_{3}}{c_{1}}\ast x\ast s_{1} + 500\ast\frac{c_{3}\ast s_{1} - c_{1}\ast s_{3}}{c_{1}}
$$

$$
w\ast s_{4} = \frac{c_{4}}{c_{1}}\ast x\ast s_{1} + 500\ast\frac{c_{4}\ast s_{1} - c_{1}\ast s_{4}}{c_{1}}
$$

Подставляя в функцию ограничения:

$$
x\ast s_{1} + y\ast s_{2} + z\ast s_{3} + w\ast s_{4} = S
$$

≡

$$
x\ast s_{1} + \frac{c_{2}}{c_{1}}\ast x\ast s_{1} + 500\ast\frac{c_{2}\ast s_{1} - c_{1}\ast s_{2}}{c_{1}} + \frac{c_{3}}{c_{1}}\ast x\ast s_{1} + 500\ast\frac{c_{3}\ast s_{1} - c_{1}\ast s_{3}}{c_{1}} + \frac{c_{4}}{c_{1}}\ast x\ast s_{1} + 500\ast\frac{c_{4}\ast s_{1} - c_{1}\ast s_{4}}{c_{1}} = S
$$

≡

$$
\frac{x\ast s_{1}\ast\left( c_{1} + c_{2} + c_{3} + c_{4} \right)}{c_{1}} + \frac{500}{c_{1}}\ast\left( c_{2}\ast s_{1} - c_{1}\ast s_{2} + c_{3}\ast s_{1} - c_{1}\ast s_{3} + c_{4}\ast s_{1} - c_{1}\ast s_{4} \right) = S
$$

≡

$$
\frac{x\ast s_{1}\ast\left( c_{1} + c_{2} + c_{3} + c_{4} \right)}{c_{1}} + \frac{500}{c_{1}}\ast\left( s_{1}\ast\left( c_{2} + c_{3} + c_{4}\  \right) - c_{1}\ast\left( s_{2} + s_{3} + s_{4} \right) \right) = S
$$

≡

$$
x\ast s_{1}\ast\frac{c_{1} + c_{2} + c_{3} + c_{4}}{c_{1}} + \frac{500}{c_{1}}\ast\left( s_{1}\ast\left( c_{2} + c_{3} + c_{4}\  \right) - c_{1}\ast\left( s_{2} + s_{3} + s_{4} \right) \right) = S
$$

≡

$$
x\ast s_{1} = \frac{S - 500\ast\left( \frac{s_{1}}{c_{1}}\ast\left( c_{2} + c_{3} + c_{4} \right) - \left( s_{2} + s_{3} + s_{4} \right) \right)}{\frac{c_{1} + c_{2} + c_{3} + c_{4}}{c_{1}}}
$$

Те же шаги можно проделать для y, z, w.

## Что делать при нехватке места на складе

При малом S любая из переменных (x, y, z, w) может оказаться отрицательной. В этом случае мы убираем переменную, ушедшую в минус, и повторяем шаги выше. При реализации этого решения такие случаи удобно обрабатывать рекурсивной функцией.
