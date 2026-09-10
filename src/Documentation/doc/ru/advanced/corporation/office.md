# Офис

## Общая информация

Характеристики сотрудников отслеживаются как средние значения. Всего таких средних 6: `AvgEnergy`, `AvgMorale`, `AvgIntelligence`, `AvgCharisma`, `AvgCreativity`, `AvgEfficiency`. При каждом найме нового сотрудника эти средние пересчитываются. Они изменяются на случайное число от 50 до 100:

```typescript
this.avgMorale =
  (this.avgMorale * this.numEmployees + getRandomInt(averageStat, averageStat)) / (this.numEmployees + 1);
```

Назначение должностей:

- У каждого офиса есть 2 записи: `employeeJobs` и `employeeNextJobs`. Данные из `employeeJobs` (количество сотрудников на каждой должности) используются для расчёта всего остального: `EmployeeProductionByJob`, `AvgEnergy`, `AvgMorale`, `TotalExperience`. При вызове `setJobAssignment` его параметр вычисляется и записывается в `employeeNextJobs`. В состоянии START следующего цикла данные из `employeeNextJobs` копируются в `employeeJobs`.
- Поведение `setJobAssignment` на первый взгляд может сбивать с толку. Допустим, вы вызвали его так: `ns.corporation.setJobAssignment("Agriculture","Sector-12","Operations", 5)`
  - Если у вас 5 сотрудников на \"Operations\", не произойдёт ничего.
  - Если у вас 7 сотрудников на \"Operations\", их количество снизится до 5, а 2 сотрудника станут \"Unassigned\".
  - Если у вас 2 сотрудника на \"Operations\", проверяется, есть ли хотя бы 3 сотрудника \"Unassigned\". Если да, количество на \"Operations\" станет 5, а \"Unassigned\" уменьшится на 3. Если нет, будет выброшена ошибка. По сути, функция пытается перевести сотрудников из \"Unassigned\" на \"Operations\".
- Отсюда правильный порядок работы с `setJobAssignment`:
  - Сначала обнулить все должности.
  - Затем выставить на всех должностях нужные вам значения.

Суммарный опыт растёт в таких случаях:

- Найм нового сотрудника. Каждый новый сотрудник увеличивает суммарный опыт на `getRandomInt(50, 100)`.
- В состоянии START. Прирост за цикл:

$$
TotalExperienceGain = 0.0015\ast(TotalEmployees - UnassignedEmployees + InternEmployees\ast 9)
$$

- Если в офисе 100 сотрудников и все они на должностях, кроме intern, прирост составит 0.15 опыта за цикл. Без бонусного времени это 54 опыта в час.

Зарплата за цикл:

$$
Salary = 3\ast TotalEmployees\ast\left(AvgIntelligence+AvgCharisma+AvgCreativity+AvgEfficiency+\frac{TotalExperience}{TotalEmployees}\right)
$$

## Апгрейды

Стоимость апгрейда:

$$
UpgradeCost = BasePrice\ast\left( \frac{\sqrt[3]{1.09} - 1}{0.09} \right)\ast{1.09}^{\frac{CurrentSize}{3}}
$$

Стоимость апгрейда с размера 3 до размера n:

$$
UpgradeCost_{From\ 3\ to\ n} = \sum_{k = 3}^{n - 1}{BasePrice\ast\left( \frac{\sqrt[3]{1.09} - 1}{0.09} \right)\ast{1.09}^{\frac{k}{3}}}
$$

≡

$$
UpgradeCost_{From\ 3\ to\ n} = \sum_{k = 3}^{n - 1}{BasePrice\ast\left( \frac{\sqrt[3]{1.09} - 1}{0.09} \right)\ast\left( \sqrt[3]{1.09} \right)^{k}}
$$

≡

$$
UpgradeCost_{From\ 3\ to\ n} = BasePrice\ast\left( \frac{\sqrt[3]{1.09} - 1}{0.09} \right)\ast\left( \frac{\left( \sqrt[3]{1.09} \right)^{n} - 1.09}{\sqrt[3]{1.09} - 1} \right)
$$

≡

$$
UpgradeCost_{From\ 3\ to\ n} = BasePrice\ast\left( \frac{{1.09}^{\frac{n}{3}} - 1.09}{0.09} \right)
$$

Стоимость апгрейда с размера a до размера b:

$$
UpgradeCost_{From\ a\ to\ b} = BasePrice\ast\left( \frac{{1.09}^{\frac{b}{3}} - {1.09}^{\frac{a}{3}}}{0.09} \right)
$$

Максимальный размер при заданном `MaxCost`:

$$
MaxSize = 3\ast\log_{1.09}\left( MaxCost\ast\frac{0.09}{BasePrice} + {1.09}^{\frac{CurrentSize}{3}} \right)
$$

## Энергия и мораль

Они рассчитываются в состоянии START.

Они начинают падать, когда количество сотрудников в офисе достигает 9. Минимальное значение — 10.

PerfMult — это множитель, повышающий или понижающий энергию и мораль.

$$
InternMultiplier = 0.002\ast Min\left(\frac{1}{9},\frac{InternEmployees}{TotalEmployees}-\frac{1}{9}\right)\ast 9
$$

$$
PenaltyMultiplier = \begin{cases}0, & (CorpFunds > 0) \vee (DivisionLastCycleRevenue > DivisionLastCycleExpenses) \newline 0.001, & (CorpFunds < 0) \land (DivisionLastCycleRevenue < DivisionLastCycleExpenses)\end{cases}
$$

$$
PerfMult = \begin{cases}1.002, & TotalEmployees < 9 \newline 1 + InternMultiplier - PenaltyMultiplier, & TotalEmployees \geq 9\end{cases}
$$

Покупка чая даёт фиксированные +2 к энергии. Стоит 500e3 на сотрудника.

При проведении вечеринки вычисляется `PartyMult`. Он используется при расчёте морали в следующем цикле.

$$
PartyMult = 1 + \frac{PartyCostPerEmployee}{10^{7}}
$$

На `PartyMult` количество сотрудников не влияет. Поэтому можно устроить «большую вечеринку» (высокий `PartyCostPerEmployee`), когда у вас всего 1 сотрудник, — общая стоимость будет низкой (потому что сотрудник один), — а остальных нанять позже.

Каждый цикл энергия и мораль дополнительно снижаются на небольшую случайную величину, не превышающую 0.002 за цикл. Величина крошечная, так что проблемы это не создаёт.

Если `PartyMult` больше 1, мораль получает фиксированную прибавку. `PartyMult` зависит от `PartyCostPerEmployee`, поэтому и прибавка зависит от `PartyCostPerEmployee`.

$$
IncreaseOfMorale = (PartyMult - 1)\ast 10
$$

≡

$$
IncreaseOfMorale = \frac{PartyCostPerEmployee}{10^{6}}
$$

```typescript
const reduction = 0.002 * marketCycles;
const increase = this.partyMult > 1 ? (this.partyMult - 1) * 10 : 0;
this.avgEnergy = (this.avgEnergy - reduction * Math.random()) * perfMult + (this.teaPending ? 2 : 0);
this.avgMorale = ((this.avgMorale - reduction * Math.random()) * perfMult + increase) * this.partyMult;
```

Противодействовать падению энергии и морали можно 3 способами:

- Покупать чай и устраивать вечеринки. Этот вариант следует использовать всегда. Автоматизирующий скрипт пишется очень легко.
- Назначать Intern. Многие называют соотношение 1/9 как способ компенсировать падение энергии и морали. Но это соотношение работает, только когда с корпорацией/подразделением всё в порядке. Если нет, включается штрафной множитель (0.001), и тогда нужно использовать 1/6.
- Купить 2 исследования: AutoBrew и AutoPartyManager. Они держат энергию и мораль на максимуме. Но покупать их не стоит никогда: всегда выгоднее потратить RP на другие полезные исследования или просто копить его.

При найме нового сотрудника `AvgEnergy` и `AvgMorale` увеличиваются на случайную величину.

```typescript
this.avgMorale = (this.avgMorale * this.numEmployees + getRandomInt(50, 100)) / (this.numEmployees + 1);
this.avgEnergy = (this.avgEnergy * this.numEmployees + getRandomInt(50, 100)) / (this.numEmployees + 1);
```

Оптимальный `PartyCostPerEmployee`:

- Фиксированное случайное снижение крошечное, поэтому им можно пренебречь.
- Мы хотим поднять `AvgMorale` с `CurrentMorale` до `MaxMorale`:

$$
\left( CurrentMorale\ast PerfMult + \frac{PartyCostPerEmployee}{10^{6}} \right)\ast\left( 1 + \frac{PartyCostPerEmployee}{10^{7}} \right) = MaxMorale
$$

- Обозначим:

$$
a = CurrentMorale
$$

$$
b = MaxMorale
$$

$$
k = PerfMult
$$

$$
x = PartyCostPerEmployee
$$

- Получаем уравнение:

$$
\left( a\ast k + \frac{x}{10^{6}} \right)\ast\left( 1 + \frac{x}{10^{7}} \right) = b
$$

≡

$$
x_{1} = - 500000\ast\left( \sqrt{(a\ast k - 10)^{2} + 40\ast b} + a\ast k + 10 \right)
$$

$$
x_{2} = 500000\ast\left( \sqrt{(a\ast k - 10)^{2} + 40\ast b} - a\ast k - 10 \right)
$$

- $x_{1}$ всегда отрицателен, поэтому единственное решение — $x_{2}$.

Одна большая вечеринка менее выгодна, чем несколько маленьких. Например: 1 большая вечеринка для подъёма морали с 70 до 100 обойдётся дороже, чем 3 маленькие: 70→80, 80→90, 90→100.

Не скупитесь на чай и вечеринки. Энергия и мораль критически важны для эффективного офиса. Формулы — в следующей части.

- На вечеринку вполне нормально тратить 500e3 на сотрудника. При желании можно и больше.
- Старайтесь постоянно держать энергию и мораль на максимуме. Лично я всегда покупаю чай / устраиваю вечеринку, когда энергия или мораль опускается до 99.5 (или до 109.5, если куплены соответствующие исследования).
- В раундах 1 и 2 офис маленький, обычно меньше 9 сотрудников, поэтому энергия и мораль не проблема. Начиная с раунда 3 покупать чай и устраивать вечеринки нужно каждый цикл.

## Производительность сотрудников по должностям

В состоянии START каждого цикла все характеристики используются для расчёта значений «производительности». Эти значения сохраняются в `office.employeeProductionByJob` и затем используются для расчёта:

- RP.
- Качества материалов.
- Характеристик продуктов.
- Сырого производства подразделения.
- MaxSalesVolume материалов/продуктов.

Формулы:

- Вычислите множители Intelligence, Charisma, Creativity и Efficiency. Каждый из них равен произведению среднего значения, эффекта апгрейдов и эффекта исследований.
- База производительности:

$$
ProductionBase = AvgMorale\ast AvgEnergy\ast 10^{-4}
$$

- Опыт:

$$
Exp = \frac{TotalExperience}{TotalEmployees}
$$

- Множитель производительности:
  - Operations: $$ProductionMultiplier = 0.6\ast IntelligenceMult + 0.1\ast CharismaMult + Exp + 0.5\ast CreativityMult + EfficiencyMult$$
  - Engineer: $$ProductionMultiplier = IntelligenceMult + 0.1\ast CharismaMult + 1.5\ast Exp + EfficiencyMult$$
  - Business: $$ProductionMultiplier = 0.4\ast IntelligenceMult + CharismaMult + 0.5\ast Exp$$
  - Management: $$ProductionMultiplier = 2\ast CharismaMult + Exp + 0.2\ast CreativityMult + 0.7\ast EfficiencyMult$$
  - Research and Development: $$ProductionMultiplier = 1.5\ast IntelligenceMult + 0.8\ast Exp + CreativityMult + 0.5\ast EfficiencyMult$$
- $EmployeesJobCount = office.employeeJobs[JobName]$
- Производительность сотрудников по должности:

$$
EmployeeProductionByJob = EmployeesJobCount\ast ProductionMultiplier\ast ProductionBase
$$

## Расчёт характеристик сотрудника

4 характеристики — `AvgIntelligence`, `AvgCharisma`, `AvgCreativity`, `AvgEfficiency` — недоступны через NS API.

Их можно вычислить по формулам из предыдущей части с помощью [Ceres Solver](./miscellany.md). Из 5 должностей (Operations, Engineer, Business, Management и Research & Development) для применения этого решения нужно, чтобы как минимум на 4 должностях был хотя бы 1 сотрудник.
