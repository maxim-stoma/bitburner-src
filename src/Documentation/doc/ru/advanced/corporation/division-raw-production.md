# Сырое производство подразделения

## Определение

Каждой отрасли нужны свои входные материалы. У каждого требуемого материала свой коэффициент. Это значение не то же самое, что коэффициент бустового материала, — это разные вещи. Например:

- Agriculture: { Water: 0.5, Chemicals: 0.2 }
- Chemical: { Plants: 1, Water: 0.5 }
- Tobacco: { Plants: 1 }

У каждого подразделения есть число, которое я называю «сырым производством подразделения». Это значение отражает производственную мощность подразделения. Назовём его `RawProduction`. Оно используется для:

- Расчёта необходимого количества входных материалов. Оно умножается на коэффициент входного материала, чтобы получить требуемое количество этого материала.
- Расчёта того, сколько материала/продукта подразделение может произвести. Оно умножается на `ProducibleFrac`. `ProducibleFrac` начинается с 1 и уменьшается, если входных материалов не хватает.

Например, для Agriculture: пусть `RawProduction` равно 1000 — тогда мы потребляем 500 единиц Water и 200 единиц Chemicals
и производим 1000 единиц Plants и 1000 единиц Food.

## Снижение производства из-за нехватки места на складе

`RawProduction` будет понижено, если на складе недостаточно свободного места. Игра вычисляет чистое
изменение занятого места, а затем понижает `RawProduction` исходя из этого изменения и объёма свободного места.

Например, для Agriculture на каждую единицу `RawProduction` мы потребляем 0.5 единицы Water и 0.2 единицы Chemicals,
чтобы произвести 1 единицу Plants и 1 единицу Food. Размеры этих материалов:

- Water: 0.05
- Chemicals: 0.05
- Plants: 0.05
- Food: 0.03

Чистое изменение занятого места: `0.05 + 0.03 - (0.5 * 0.05 + 0.2 * 0.05) = 0.045`.

То есть на каждую единицу `RawProduction` нужно 0.045 единицы свободного места. Пусть `RawProduction` равно 1000, а
свободного места — 22.5. Для 1000 единиц `RawProduction` нужно 45 единиц свободного места, но доступно только 22.5,
поэтому фактическое `RawProduction` понижается до 500.

## Формула

`RawProduction` — это произведение 4 множителей:

- Множитель офиса:
  - Производительность сотрудников на 3 должностях (Operations, Engineer, Management) и её сумма:
    - `OperationsProd = office.employeeProductionByJob.Operations`
    - `EngineerProd = office.employeeProductionByJob.Engineer`
    - `ManagementProd = office.employeeProductionByJob.Management`
    - $TotalEmployeesProd = OperationsProd + EngineerProd + ManagementProd$
  - Фактор менеджмента:
    $$ManagementFactor = 1 + \frac{ManagementProd}{1.2\ast TotalEmployeesProd}$$
  - Множитель производительности сотрудников:
    $$EmployeeProductionMultiplier = \left( (OperationsProd)^{0.4} + (EngineerProd)^{0.3} \right)\ast ManagementFactor$$
  - Балансировочный множитель:
    $$BalancingMultiplier = 0.05$$
  - Если на выходе материал:
    $$OfficeMultiplier = BalancingMultiplier\ast EmployeeProductionMultiplier$$
  - Если на выходе продукт:
    $$OfficeMultiplier = 0.5\ast BalancingMultiplier\ast EmployeeProductionMultiplier$$
- Производственный множитель подразделения: см. предыдущий [раздел](./boost-material.md).
- Множитель апгрейдов: множитель от [Smart Factories](./unlocks-upgrade-research.md).
- Множитель исследований: множитель от [исследований](./unlocks-upgrade-research.md).
