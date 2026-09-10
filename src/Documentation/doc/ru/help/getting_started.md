# Руководство для начинающих программистов

_Примечание_: [скрипты](../basic/scripts.md) и стратегии в этом руководстве не обязательно оптимальны или исчерпывающи.
Оно рассчитано на то, чтобы помочь людям с минимальным опытом программирования освоиться в Bitburner на ранних этапах игры.

Если вас смущает или пугает игра, особенно её кодовая и скриптовая сторона, — это руководство как раз для вас!

## Введение

Bitburner — инкрементальная RPG в киберпанковском сеттинге.
Вы будете продвигаться, повышая свои [характеристики](../basic/stats.md), зарабатывая деньги и, с практикой, прокачивая свои реальные навыки программирования.
Достигнув определённых условий, вы будете получать приглашения от внутриигровых [фракций](../basic/factions.md).
Вступление во [фракции](../basic/factions.md) и работа на них открывает разные [аугментации](../basic/augmentations.md),
которые покупаются и «устанавливаются», добавляя постоянный бонус к [характеристикам](../basic/stats.md) и другим способностям. Работа с фракциями и установка аугментаций — базовый шаг для продвижения в Bitburner.

У игры открытый, минималистичный сюжет, который можно проходить разными способами, добиваясь своих целей.
Поскольку это руководство написано как базовое введение в Bitburner, оно не раскрывает весь объём игры или сюжета.

## Первые шаги

Будем считать, что вы прошли вводный туториал в самом начале игры.
В этом туториале вы создали [скрипт](../basic/scripts.md) под названием `n00dles.js` и запустили его на сервере `n00dles`.
Теперь давайте остановим этот [скрипт](../basic/scripts.md). Сделать это можно двумя способами:

- Перейти в Терминал и ввести: `kill n00dles.js`
- Перейти на страницу `Active Scripts` (Alt + s) и нажать кнопку `Kill Script` для `n00dles.js`.

Если вы пропустили вводный туториал, часть выше можете пропустить.
Вместо этого перейдите на страницу `Hacknet Nodes` (Alt + h) и купите [Hacknet-ноду](../basic/hacknet_nodes.md), чтобы начать получать пассивный доход.

## Создаём наш первый скрипт

Теперь напишем универсальный [хакерский](../basic/hacking.md) [скрипт](../basic/scripts.md), который можно использовать на раннем этапе игры (а при желании — и на протяжении всей игры). Такой скрипт обычно называют «early hack template» или EHT.

Прежде чем писать [скрипт](../basic/scripts.md), стоит познакомиться с этими вещами:

- `hacking`
- `security`
- `hack`
- `grow`
- `weaken`
- `brutessh`
- `nuke`

Вкратце: у каждого [сервера](../basic/servers.md) есть уровень защиты, влияющий на то, насколько трудно его взломать.
У каждого [сервера](../basic/servers.md) также есть определённая сумма денег и максимум денег, который он может хранить.
[Взлом](../basic/hacking.md) [сервера](../basic/servers.md) похищает процент денег этого [сервера](../basic/servers.md).
Функция `hack()` используется для взлома [сервера](../basic/servers.md).
Функция `grow()` увеличивает количество доступных денег на [сервере](../basic/servers.md).
Функция `weaken()` снижает уровень защиты [сервера](../basic/servers.md).

Теперь перейдём непосредственно к созданию [скрипта](../basic/scripts.md).
Перейдите на домашний компьютер и создайте [скрипт](../basic/scripts.md) с именем `early-hack-template.js`, введя в [Терминале](../basic/terminal.md) следующие две команды:

    $ home
    $ nano early-hack-template.js

Это откроет редактор [скриптов](../basic/scripts.md), в котором можно писать и создавать [скрипты](../basic/scripts.md).

Введите в редактор [скриптов](../basic/scripts.md) следующий код:

    /** @param {NS} ns */
    export async function main(ns) {
        // Defines the "target server", which is the server
        // that we're going to hack. In this case, it's "n00dles"
        const target = "n00dles";

        // Defines how much money a server should have before we hack it
        // In this case, it is set to the maximum amount of money.
        const moneyThresh = ns.getServerMaxMoney(target);

        // Defines the minimum security level the target server can
        // have. If the target's security level is higher than this,
        // we'll weaken it before doing anything else
        const securityThresh = ns.getServerMinSecurityLevel(target);

        // If we have the BruteSSH.exe program, use it to open the SSH Port
        // on the target server
        if (ns.fileExists("BruteSSH.exe", "home")) {
            ns.brutessh(target);
        }

        // Get root access to target server
        ns.nuke(target);

        // Infinite loop that continously hacks/grows/weakens the target server
        while(true) {
            if (ns.getServerSecurityLevel(target) > securityThresh) {
                // If the server's security level is above our threshold, weaken it
                await ns.weaken(target);
            } else if (ns.getServerMoneyAvailable(target) < moneyThresh) {
                // If the server's money is less than our threshold, grow it
                await ns.grow(target);
            } else {
                // Otherwise, hack it
                await ns.hack(target);
            }
        }
    }

[Скрипт](../basic/scripts.md) выше уже прокомментирован, но всё же разберём его по шагам.

    const target = "n00dles";

Эта первая команда задаёт строку с нашим целевым [сервером](../basic/servers.md).
Это тот [сервер](../basic/servers.md), который мы будем [взламывать](../basic/hacking.md).
Пока что это `"n00dles"`, потому что это единственный [сервер](../basic/servers.md) с требуемым уровнем хакинга `1`.
Если хотите [взламывать](../basic/hacking.md) другой [сервер](../basic/servers.md), просто измените эту переменную на hostname другого [сервера](../basic/servers.md).

    const moneyThresh = ns.getServerMaxMoney(target);

Вторая команда задаёт числовое значение — минимальную сумму денег на целевом [сервере](../basic/servers.md), при которой наш [скрипт](../basic/scripts.md) станет его [взламывать](../basic/hacking.md).
Если денег на целевом [сервере](../basic/servers.md) меньше этого значения, [скрипт](../basic/scripts.md) вызовет `grow()`, а не будет [взламывать](../basic/hacking.md) сервер.
Значение выставлено в максимум денег, который может быть на [сервере](../basic/servers.md).
Найти это значение помогает функция `getServerMaxMoney()`.

    const securityThresh = ns.getServerMinSecurityLevel(target);

Третья команда задаёт числовое значение — минимальный уровень защиты целевого [сервера](../basic/servers.md).
Если уровень защиты целевого [сервера](../basic/servers.md) выше этого значения, наш [скрипт](../basic/scripts.md) сначала применит `weaken()`, прежде чем делать что-либо ещё.

    if (ns.fileExists("BruteSSH.exe", "home")) {
        ns.brutessh(target);
    }

    ns.nuke(target);

Этот участок кода получает root-доступ на целевом [сервере](../basic/servers.md).
Он необходим для [взлома](../basic/hacking.md).

    while (true) {
        if (ns.getServerSecurityLevel(target) > securityThresh) {
            // If the server's security level is above our threshold, weaken it
            await ns.weaken(target);
        } else if (ns.getServerMoneyAvailable(target) < moneyThresh) {
            // Otherwise, if the server's money is less than our threshold, grow it
            await ns.grow(target);
        } else {
            // Otherwise, hack it
            await ns.hack(target);
        }
    }

Это главный раздел, приводящий в движение наш [скрипт](../basic/scripts.md).
Он задаёт логику [скрипта](../basic/scripts.md) и выполняет операции [взлома](../basic/hacking.md).
`while (true)` создаёт бесконечный цикл, который непрерывно выполняет логику [взлома](../basic/hacking.md), пока [скрипт](../basic/scripts.md) не будет остановлен.

Ключевое слово await нужно для `hack()` / `grow()` / `weaken()`, потому что, в отличие от других команд, их выполнение занимает время.
Если забыть про await, вы получите исключение о попытке сделать несколько вещей одновременно, потому что ваш код тут же завершит вызов функции, не дожидаясь окончания операции.
Также важно, что await можно использовать только внутри функций, помеченных `async` (обратите внимание, что `main()` помечена `async`).

## Запускаем наши скрипты

Теперь запустим наш [хакерский](../basic/hacking.md) [скрипт](../basic/scripts.md), чтобы он начал приносить нам деньги и опыт.
На нашем домашнем компьютере всего 8 ГБ [RAM](../basic/ram.md), и позже она понадобится нам для другого.
Вместо этого мы задействуем [RAM](../basic/ram.md) на других машинах.

Перейдите в `Terminal` и введите команду:

    $ scan-analyze 2

Она покажет подробную информацию о некоторых [серверах](../basic/servers.md) сети.

**_Сеть генерируется случайно, поэтому у каждого игрока она своя._**

Вот что показала команда у меня на момент написания:

    [home ~]> scan-analyze 2
    ┕ home
      ┃   Root Access: YES, Required hacking skill: 1
      ┃   Number of open ports required to NUKE: 5
      ┃   RAM: 8.00GB
      ┣ n00dles
      ┃ ┃   Root Access: YES, Required hacking skill: 1
      ┃ ┃   Number of open ports required to NUKE: 0
      ┃ ┃   RAM: 4.00GB
      ┃ ┕ nectar-net
      ┃       Root Access: NO, Required hacking skill: 20
      ┃       Number of open ports required to NUKE: 0
      ┃       RAM: 16.00GB
      ┣ foodnstuff
      ┃ ┃   Root Access: NO, Required hacking skill: 1
      ┃ ┃   Number of open ports required to NUKE: 0
      ┃ ┃   RAM: 16.00GB
      ┃ ┕ zer0
      ┃       Root Access: NO, Required hacking skill: 75
      ┃       Number of open ports required to NUKE: 1
      ┃       RAM: 32.00GB
      ┣ sigma-cosmetics
      ┃ ┃   Root Access: NO, Required hacking skill: 5
      ┃ ┃   Number of open ports required to NUKE: 0
      ┃ ┃   RAM: 16.00GB
      ┃ ┕ max-hardware
      ┃       Root Access: NO, Required hacking skill: 80
      ┃       Number of open ports required to NUKE: 1
      ┃       RAM: 32.00GB
      ┣ joesguns
      ┃     Root Access: NO, Required hacking skill: 10
      ┃     Number of open ports required to NUKE: 0
      ┃     RAM: 16.00GB
      ┣ hong-fang-tea
      ┃     Root Access: NO, Required hacking skill: 30
      ┃     Number of open ports required to NUKE: 0
      ┃     RAM: 16.00GB
      ┣ harakiri-sushi
      ┃     Root Access: NO, Required hacking skill: 40
      ┃     Number of open ports required to NUKE: 0
      ┃     RAM: 16.00GB
      ┕ iron-gym
        ┃   Root Access: NO, Required hacking skill: 100
        ┃   Number of open ports required to NUKE: 1
        ┃   RAM: 32.00GB
        ┕ CSEC
              Root Access: NO, Required hacking skill: 55
              Number of open ports required to NUKE: 1
              RAM: 8.00GB

Обратите внимание на следующие серверы:

- `sigma-cosmetics`
- `joesguns`
- `nectar-net`
- `hong-fang-tea`
- `harakiri-sushi`
- `foodnstuff`

У всех этих серверов по 16 ГБ [RAM](../basic/ram.md).
Более того, ни одному из этих серверов не нужны открытые порты для NUKE.
Иначе говоря, мы можем получить root-доступ на всех этих серверах и запускать на них [скрипты](../basic/scripts.md).

Сначала выясним, сколько потоков нашего [хакерского](../basic/hacking.md) [скрипта](../basic/scripts.md) мы можем запустить.
(Подробнее о многопоточности — на странице про [скрипты](../basic/scripts.md).)

Написанный нами [скрипт](../basic/scripts.md) занимает 2.6 ГБ [RAM](../basic/ram.md).
Проверить это можно командой `Terminal`:

    $ mem early-hack-template.js

Значит, на сервере с 16 ГБ мы можем запустить 6 потоков.
Теперь, чтобы запустить наши [скрипты](../basic/scripts.md) на всех этих серверах, нужно сделать следующее:

1. Скопировать наш [скрипт](../basic/scripts.md) на каждый сервер командой `scp`.
2. Подключиться к серверу командой `connect`.
3. Запустить программу `NUKE.exe` командой `run`, чтобы получить root-доступ.
4. Снова использовать команду `run`, чтобы запустить наш [скрипт](../basic/scripts.md).
5. Повторить шаги 2-4 для каждого сервера.

Вот последовательность команд `Terminal`, которую я использовал для этого:

    $ home
    $ scp early-hack-template.js n00dles
    $ scp early-hack-template.js sigma-cosmetics
    $ scp early-hack-template.js joesguns
    $ scp early-hack-template.js nectar-net
    $ scp early-hack-template.js hong-fang-tea
    $ scp early-hack-template.js harakiri-sushi
    $ scp early-hack-template.js foodnstuff
    $ connect n00dles
    $ run NUKE.exe
    $ run early-hack-template.js -t 1
    $ home
    $ connect sigma-cosmetics
    $ run NUKE.exe
    $ run early-hack-template.js -t 6
    $ home
    $ connect joesguns
    $ run NUKE.exe
    $ run early-hack-template.js -t 6
    $ home
    $ connect hong-fang-tea
    $ run NUKE.exe
    $ run early-hack-template.js -t 6
    $ home
    $ connect harakiri-sushi
    $ run NUKE.exe
    $ run early-hack-template.js -t 6
    $ home
    $ connect n00dles
    $ connect nectar-net
    $ run NUKE.exe
    $ run early-hack-template.js -t 6
    $ home
    $ connect foodnstuff
    $ run NUKE.exe
    $ run early-hack-template.js -t 6

Нажатие клавиши `Tab` посреди команды Терминала попытается автодополнить её.
Например, если набрать `scp ea` и нажать `Tab`, остаток имени [скрипта](../basic/scripts.md) должен подставиться автоматически.
Это работает для большинства команд игры!

Команда `home` подключает вас к домашнему компьютеру. При запуске наших [скриптов](../basic/scripts.md) командой `run early-hack-template.js -t 6` параметр `-t 6` указывает, что [скрипт](../basic/scripts.md) нужно запустить в 6 потоков.

Обратите внимание, что [сервер](../basic/servers.md) `nectar-net` не входит в ближайшую сеть домашнего компьютера.
Это значит, что подключиться к нему напрямую из дома нельзя. Его придётся искать внутри сети.
Результаты выполненной ранее команды `scan-analyze 2` покажут, где он находится.
У меня получилось подключиться к нему через `n00dles` -> `nectar-net`.
Впрочем, у вас, скорее всего, будет по-другому.

После выполнения всех этих команд `Terminal` наши [скрипты](../basic/scripts.md) уже работают.
Они начнут приносить деньги и опыт хакинга со временем.
Сейчас прирост будет совсем небольшим, но он вырастет, когда поднимется наш навык хакинга и мы запустим больше [скриптов](../basic/scripts.md).

## Повышаем уровень хакинга

Кроме `n00dles`, есть много других [серверов](../basic/servers.md), которые можно взламывать, но у них выше требуемый уровень хакинга.
Поэтому стоит поднять свой уровень хакинга.
Это не только позволит взламывать больше [серверов](../basic/servers.md), но и повысит эффективность нашего [взлома](../basic/hacking.md) `n00dles`.

Проще всего тренировать уровень хакинга, посетив Rothman University.
Сделать это можно на вкладке `City` (Alt + w) в левом навигационном меню.
Rothman University должен быть отмечен буквой «U» внизу справа.
Нажмите на «U», чтобы перейти туда.

Придя в Rothman University, вы увидите экран с несколькими вариантами.
Эти варианты описывают разные курсы, которые можно пройти.
Нажмите на первую кнопку: `Study Computer Science (free)`.

После нажатия вы начнёте учиться и получать опыт хакинга.
Пока вы этим занимаетесь, вы не сможете взаимодействовать с остальной игрой, пока не нажмёте `Stop taking course` или `Do something else simultaneously`.

Сейчас нам нужен уровень хакинга 10.
Для достижения 10 уровня нужно примерно 174 очка опыта хакинга.
Проверить, сколько у вас опыта хакинга, можно на вкладке `Stats` (Alt + c) в левом навигационном меню.
Поскольку учёба в Rothman University даёт 1 опыт в секунду, это займёт 174 секунды, то есть около 3 минут.
Можете пока заняться чем-нибудь ещё!

## Редактируем наш хакерский скрипт

Теперь, когда у нас уровень хакинга 10, мы можем взламывать [сервер](../basic/servers.md) `joesguns`.
Этот [сервер](../basic/servers.md) немного прибыльнее, чем `n00dles`.
Поэтому изменим наш [хакерский](../basic/hacking.md) [скрипт](../basic/scripts.md), чтобы он целился в `joesguns` вместо `n00dles`.

Перейдите в `Terminal` и отредактируйте [хакерский](../basic/hacking.md) [скрипт](../basic/scripts.md), введя:

    $ home
    $ nano early-hack-template.js

В начале [скрипта](../basic/scripts.md) измените переменную `target` на `"joesguns"`:

    const target = "joesguns";

Учтите, что это **НЕ** повлияет на уже запущенные экземпляры [скрипта](../basic/scripts.md).
Это затронет только те экземпляры [скрипта](../basic/scripts.md), которые будут запущены после этого изменения.

## Создаём новый скрипт для доступа к облачным серверам

Далее мы напишем [скрипт](../basic/scripts.md), автоматически покупающий доступ к дополнительным облачным [серверам](../basic/servers.md).
Эти облачные [серверы](../basic/servers.md) пригодятся для запуска множества [скриптов](../basic/scripts.md).
Поначалу запуск этого [скрипта](../basic/scripts.md) обойдётся довольно дорого, поскольку покупка облачного [сервера](../basic/servers.md) стоит денег, но в долгосрочной перспективе она окупится.

Чтобы написать этот [скрипт](../basic/scripts.md), стоит познакомиться со следующими функциями — часть из них находится в [Cloud API](../../../../../markdown/bitburner.cloud.md):

- `cloud.purchaseServer()`
- `cloud.getServerCost()`
- `cloud.getServerLimit()`
- `cloud.getRamLimit()`
- `getServerMoneyAvailable()`
- `scp()`
- `exec()`

Создайте [скрипт](../basic/scripts.md), перейдя в `Terminal` и введя:

    $ home
    $ nano purchase-server-8gb.js

Вставьте в редактор [скрипта](../basic/scripts.md) следующий код:

    /** @param {NS} ns */
    export async function main(ns) {
        // How much RAM each cloud server will have. In this case, it'll be 8GB.
        const ram = 8;

        // Iterator we'll use for our loop
        let i = ns.cloud.getServerNames().length;

        // Continuously try to purchase cloud servers until we've reached the maximum
        // amount of servers
        while (i < ns.cloud.getServerLimit()) {
            // Check if we have enough money to purchase access to a server
            if (ns.getServerMoneyAvailable("home") > ns.cloud.getServerCost(ram)) {
                // If we have enough money, then:
                //  1. Purchase the server
                //  2. Copy our hacking script onto the newly purchased cloud server
                //  3. Run our hacking script on the newly purchased cloud server with 3 threads
                //  4. Increment our iterator to indicate that we've bought a new server
                const hostname = ns.cloud.purchaseServer("cloud-server-" + i, ram);
                ns.scp("early-hack-template.js", hostname);
                ns.exec("early-hack-template.js", hostname, 3);
                ++i;
            }
            // Make the script wait for a second before looping again.
            // Removing this line will cause an infinite loop and crash the game.
            await ns.sleep(1000);
        }
    }

Этот код с помощью цикла while покупает максимум облачных [серверов](../basic/servers.md) функцией `purchaseServer()`.
У каждого из этих [серверов](../basic/servers.md) будет по 8 ГБ [RAM](../basic/ram.md), как задано в переменной `ram`.
Обратите внимание, что [скрипт](../basic/scripts.md) использует команду `getServerMoneyAvailable("home")`, чтобы узнать, сколько у вас сейчас денег.
Это значение затем используется для проверки, хватает ли средств на покупку облачного [сервера](../basic/servers.md).

Каждый раз, покупая новый облачный [сервер](../basic/servers.md), скрипт использует функцию `scp()`, чтобы скопировать на него наш [скрипт](../basic/scripts.md), а затем функцию `exec()`, чтобы запустить его на этом облачном [сервере](../basic/servers.md).

Чтобы запустить этот [скрипт](../basic/scripts.md), перейдите в `Terminal` и введите:

    $ run purchase-server-8gb.js

Покупка будет продолжаться, пока не будет куплено максимальное количество облачных [серверов](../basic/servers.md).
Когда это произойдёт, у вас появится целая куча новых [серверов](../basic/servers.md), на всех из которых работают [хакерские](../basic/hacking.md) [скрипты](../basic/scripts.md), нацеленные на сервер `joesguns`!

Мы используем так много [скриптов](../basic/scripts.md) для взлома именно `joesguns`, а не других [серверов](../basic/servers.md), потому что так эффективнее.
На этом раннем этапе игры у нас не хватает [RAM](../basic/ram.md), чтобы эффективно взламывать несколько целей сразу, и попытка сделать это была бы медленной из-за распыления сил.
Позже стоит определённо этим заняться!

Учтите, что покупка облачного [сервера](../basic/servers.md) довольно дорога, а покупка максимального их числа — ещё дороже.
На момент написания этого руководства скрипту выше требовалось \$11 миллионов, чтобы полностью докупить все 8-гигабайтные [серверы](../basic/servers.md).
Поэтому нам нужны дополнительные способы зарабатывать деньги, чтобы ускорить процесс!
Они разобраны в следующем разделе.

## Дополнительные источники дохода

Помимо [скриптов](../basic/scripts.md) и [взлома](../basic/hacking.md), в игре есть и другие способы заработать деньги.

## Hacknet-ноды

Если вы прошли вводный туториал, с этим способом вы уже знакомы: [Hacknet-ноды](../basic/hacknet_nodes.md).
Как только денег станет достаточно, можно начать улучшать свои [Hacknet-ноды](../basic/hacknet_nodes.md), чтобы увеличить поток пассивного дохода.
Это полностью необязательно.
Поскольку каждое улучшение [Hacknet-ноды](../basic/hacknet_nodes.md) окупается за определённое время, не факт, что оно вам вообще нужно.

Тем не менее, [Hacknet-ноды](../basic/hacknet_nodes.md) — хороший источник дохода на раннем этапе игры, хотя позже их эффективность падает.
Если вы всё же решите покупать и улучшать [Hacknet-ноды](../basic/hacknet_nodes.md), пока советую вкладываться только в уровень.
Не стал бы пока трогать улучшения [RAM](../basic/ram.md) и ядер.

## Преступления

Сейчас лучший источник дохода — это [преступления](../basic/crimes.md).
Дело в том, что они не только приносят много денег, но и поднимают уровень хакинга.
Чтобы совершать [преступления](../basic/crimes.md), перейдите на вкладку `City` (Alt + w).
Затем нажмите на ссылку `The Slums`.

В The Slums можно попытаться совершить разные [преступления](../basic/crimes.md), каждое из которых при успехе даёт определённый опыт и деньги.
Подробнее — на странице [преступления](../basic/crimes.md).

Попытка преступления не всегда успешна.
При провале [преступления](../basic/crimes.md) ничего плохого не случится, но денег вы не получите, а опыта получите меньше.
Прокачка характеристик повышает шанс успешно совершить [преступление](../basic/crimes.md).

Сейчас лучший вариант — [преступление](../basic/crimes.md) `Rob Store`.
Попытка занимает 60 секунд, при успехе даёт \$400 тыс. и опыт хакинга (что сейчас очень важно).

В качестве альтернативы можно взять [преступление](../basic/crimes.md) `Shoplift`.
Попытка занимает 2 секунды и при успехе даёт \$15 тыс.
Это [преступление](../basic/crimes.md) чуть проще и прибыльнее, чем `Rob Store`, но не даёт опыта хакинга.

## Работа на компанию

Если не хотите совершать [преступления](../basic/crimes.md), есть другой вариант — работа на [компанию](../basic/companies.md).
Она не так прибыльна, как [преступления](../basic/crimes.md), но даёт [репутацию](../basic/reputation.md) в [компании](../basic/companies.md).

Перейдите на вкладку `City` в левом навигационном меню, а затем в `Joe's Guns`.
В `Joe's Guns` будет пункт `Apply to be an Employee`.
Нажмите на него, чтобы получить работу.
После этого появится новый пункт `Work`.
Нажмите на него, чтобы начать работать.
Работа в `Joe's Guns` приносит \$110 в секунду, а также немного опыта по всем характеристикам, кроме хакинга.

Работа на [компанию](../basic/companies.md), как и [преступления](../basic/crimes.md), полностью пассивна.
Можно сосредоточиться на работе, заниматься чем-то ещё одновременно или переключаться между этим.
Пока вы сосредоточены на работе, вы не сможете заниматься ничем другим в игре.
Если вы занимаетесь чем-то ещё одновременно, [репутация](../basic/reputation.md) набирается медленнее.
Работу можно прекратить в любой момент.

Как только ваш хакинг достигнет 75 уровня, можно посетить `Carmichael Security` в городе и устроиться туда программистом.
Эта работа платит больше и также даёт опыт хакинга.

На вкладке `City` есть и другие компании, предлагающие более высокую оплату и дополнительные игровые возможности.
Не стесняйтесь исследовать!

## После покупки новых облачных серверов

Накопив в сумме \$11 миллионов, вы завершите работу скрипта автоматической покупки облачных [серверов](../basic/servers.md).
Это освободит часть [RAM](../basic/ram.md) на домашнем компьютере.
Не хочется, чтобы эта [RAM](../basic/ram.md) простаивала, так что используем её.
Перейдите в `Terminal` и введите:

    $ home
    $ run early-hack-template.js -t 3

## Достигаем 50 уровня хакинга

Как только вы достигнете 50 уровня хакинга, откроются две новые важные части игры.

## Создаём свою первую программу: BruteSSH.exe

В левом навигационном меню вы заметите вкладку `Create Program` (Alt + p) с красным значком уведомления.
Это означает, что доступны программы для создания.
Перейдите на эту вкладку — там будет список всех программ, которые можно создать прямо сейчас.
Наведя курсор на программу, вы увидите краткое описание её назначения.
Просто кликните по программе, чтобы начать её создавать.

Сейчас нам нужна программа `BruteSSH.exe`.
Она используется для открытия SSH-портов на [серверах](../basic/servers.md).
Это позволит взламывать больше [серверов](../basic/servers.md), поскольку многим [серверам](../basic/servers.md) в игре требуется определённое число открытых портов, чтобы `NUKE.exe` мог получить root-доступ.

Работу над созданием программы можно отменить в любой момент — прогресс сохраняется, и продолжить можно позже.
`BruteSSH.exe` создаётся примерно 10 минут.

## Необязательно: создать AutoLink.exe

На странице `Create Programs` вы также заметите программу `AutoLink.exe`.
Если не жалко подождать ещё 10-15 минут, стоит её создать.
Она сильно упрощает подключение к другим [серверам](../basic/servers.md), но для прохождения не обязательна.

## Вступаем в первую фракцию: CyberSec

Вскоре после достижения 50 уровня хакинга вы должны были получить сообщение такого содержания:

    Message received from unknown sender:

    We've been watching you. Your skills are very impressive. But you're wasting your talents.
    If you join us, you can put your skills to good use and change the world for the better.
    If you join us, we can unlock your full potential.

    But first, you must pass our test. Find and install the backdoor on our server.

    -CyberSec

    This message was saved as csec-test.msg onto your home computer.

Если вы его не получили или случайно закрыли — не страшно!
Сообщения сохраняются на домашнем компьютере.
Введите такие команды `Terminal`, чтобы посмотреть сообщение:

    $ home
    $ cat csec-test.msg

Это сообщение — часть основной сюжетной линии игры.
Оно от [фракции](../basic/factions.md) `CyberSec`, которая просит вас пройти их испытание.
Пройти его просто: нужно найти их [сервер](../basic/servers.md), взломать его и установить бэкдор через [Терминал](../basic/terminal.md).
Их [сервер](../basic/servers.md) называется `CSEC`.
Для этого воспользуемся командой Терминала `scan-analyze`, как и раньше:

    $ home
    $ scan-analyze 2

Она покажет сеть всех [серверов](../basic/servers.md) в пределах 2 «узлов» от домашнего компьютера.
Помните, что сеть генерируется случайно, поэтому у каждого игрока она своя.
Вот соответствующая часть результата `scan-analyze` у меня:

    ┕ home
      ┃   Root Access: YES, Required hacking skill: 1
      ┃   Number of open ports required to NUKE: 5
      ┃   RAM: 8.00GB
      ┣ harakiri-sushi
      ┃     Root Access: NO, Required hacking skill: 40
      ┃     Number of open ports required to NUKE: 0
      ┃     RAM: 16.00GB
      ┕ iron-gym
        ┃   Root Access: NO, Required hacking skill: 100
        ┃   Number of open ports required to NUKE: 1
        ┃   RAM: 32.00GB
        ┕ CSEC
                  Root Access: NO, Required hacking skill: 55
              Number of open ports required to NUKE: 1
              RAM: 8.00GB

По этому результату видно, что до `CSEC` можно добраться через `iron-gym`:

    $ connect iron-gym
    $ connect CSEC

Если вы ранее создали программу `AutoLink.exe`, есть более простой способ подключиться к `CSEC`.
В результатах `scan-analyze` все hostname [серверов](../basic/servers.md) выделены белым и подчёркнуты.
Достаточно кликнуть по одному из hostname [сервера](../basic/servers.md), чтобы к нему подключиться.
Так что просто кликните `CSEC`!

Обратите внимание на требуемый уровень хакинга для [сервера](../basic/servers.md) `CSEC`.
Это случайное значение от 51 до 60.
Хотя сообщение от CSEC вы получаете уже при 50 уровне хакинга, пройти их испытание получится, только когда ваш хакинг будет достаточно высок, чтобы установить бэкдор на их [сервере](../basic/servers.md).

Подключившись к [серверу](../basic/servers.md) `CSEC`, можно поставить на него бэкдор.
Учтите, что этому [серверу](../basic/servers.md) для получения root-доступа нужен один открытый порт.
Открыть SSH-порт можно программой `BruteSSH.exe`, созданной ранее.
В `Terminal`:

    $ run BruteSSH.exe
    $ run NUKE.exe
    $ backdoor

Успешно установив бэкдор, вы вскоре должны получить приглашение от [фракции](../basic/factions.md) `CyberSec`.
Примите его.
Если вы случайно отклонили приглашение — не страшно.
Просто перейдите на вкладку `Factions` (Alt + f), там должна быть возможность принять приглашение.

Поздравляю!
Вы только что вступили в свою первую [фракцию](../basic/factions.md).
Пока не нужно ничего делать в этой [фракции](../basic/factions.md) — мы вернёмся к ней позже.

## Используем дополнительные серверы для взлома Joesguns

Получив программу `BruteSSH`, вы сможете получить root-доступ к нескольким дополнительным [серверам](../basic/servers.md).
На этих [серверах](../basic/servers.md) больше [RAM](../basic/ram.md), которую можно использовать для запуска [скриптов](../basic/scripts.md).
Мы используем [RAM](../basic/ram.md) этих [серверов](../basic/servers.md), чтобы запустить больше [скриптов](../basic/scripts.md), нацеленных на `joesguns`.

## Копируем наши скрипты

[Серверы](../basic/servers.md), которые мы будем использовать для запуска [скриптов](../basic/scripts.md):

- `neo-net`
- `zer0`
- `max-hardware`
- `iron-gym`

У всех этих [серверов](../basic/servers.md) по 32 ГБ [RAM](../basic/ram.md).
Убедиться в этом можно командой `Terminal` `scan-analyze 3`.
Чтобы скопировать наши [хакерские](../basic/hacking.md) [скрипты](../basic/scripts.md) на эти [серверы](../basic/servers.md), перейдите в `Terminal` и выполните:

    $ home
    $ scp early-hack-template.js neo-net
    $ scp early-hack-template.js zer0
    $ scp early-hack-template.js max-hardware
    $ scp early-hack-template.js iron-gym

Поскольку у каждого из этих [серверов](../basic/servers.md) по 32 ГБ [RAM](../basic/ram.md), наш [хакерский](../basic/hacking.md) скрипт можно запускать на каждом из них в 12 потоков.
К этому моменту вы уже должны уметь подключаться к [серверам](../basic/servers.md).
Найдите и подключитесь к каждому из перечисленных выше [серверов](../basic/servers.md) с помощью команды `Terminal` `scan-analyze 3`.
Затем запустите наш [хакерский](../basic/hacking.md) скрипт в 12 потоков такой командой `Terminal`:

    $ run early-hack-template.js -t 12

Помните: если у вас есть программа `AutoLink`, для подключения к [серверу](../basic/servers.md) достаточно кликнуть по его hostname после `scan-analyze`.

## Извлекаем прибыль из скриптов и набираем репутацию в CyberSec

Теперь пора немного подождать.
Нашим [скриптам](../basic/scripts.md) понадобится время, чтобы начать приносить деньги.
Помните, что большинство наших [скриптов](../basic/scripts.md) нацелены на `joesguns`.
Прежде чем начать [взлом](../basic/hacking.md), им нужно время, чтобы `grow()` и `weaken()` довели [сервер](../basic/servers.md) до нужных значений.
Но как только это произойдёт, [скрипты](../basic/scripts.md) станут очень прибыльными.

Для ориентира: примерно через два часа после запуска моего первого [скрипта](../basic/scripts.md) мои [скрипты](../basic/scripts.md) приносили \$20 тыс. в секунду и в сумме заработали \$70 миллионов.
(Эту статистику можно посмотреть на вкладке `Active Scripts`.)

Ещё через 15 минут скорость выработки выросла до \$25 тыс. в секунду, а [скрипты](../basic/scripts.md) заработали ещё \$55 миллионов.

Ваши результаты будут отличаться в зависимости от того, насколько быстро вы зарабатывали деньги [преступлениями](../basic/crimes.md)/[работой](../basic/companies.md)/[hacknet-нодами](../basic/hacknet_nodes.md), но это должно дать общее представление о том, сколько могут приносить [скрипты](../basic/scripts.md).

Тем временем мы будем набирать репутацию во [фракции](../basic/factions.md) `CyberSec`.
Перейдите на вкладку `Factions` (Alt + f) в левом навигационном меню и выберите там `CyberSec`.
В середине страницы будет кнопка `Hacking Contracts`.
Нажмите её, чтобы начать набирать [репутацию](../basic/reputation.md) во [фракции](../basic/factions.md) `CyberSec` (а заодно немного опыта хакинга).
Чем выше ваш уровень хакинга, тем больше [репутации](../basic/reputation.md) вы получите.
Учтите: пока вы работаете на [фракцию](../basic/factions.md), можно вообще не взаимодействовать с остальной игрой, чтобы набирать [репутацию](../basic/reputation.md) на полной скорости.
Также можно выбрать «делать что-то ещё одновременно» — тогда [репутация](../basic/reputation.md) набирается чуть медленнее, пока вы снова не сосредоточитесь.
Работу на [фракцию](../basic/factions.md) можно отменить в любой момент без потери уже набранной [репутации](../basic/reputation.md).

## Покупаем улучшения и аугментации

Как я уже упоминал, за 1-2 часа я заработал больше \$200 миллионов.
Пора потратить все эти деньги на постоянные улучшения, чтобы двигаться дальше!

## Улучшаем RAM домашнего компьютера

Сейчас важнее всего улучшить [RAM](../basic/ram.md) на домашнем компьютере.
Это позволит запускать больше [скриптов](../basic/scripts.md).

Чтобы улучшить [RAM](../basic/ram.md), перейдите на вкладку `City` и посетите компанию `Alpha Enterprises`.
Там будет кнопка `Upgrade 'home' RAM (8.00GB -> 16.00GB) - $1.010m`.
Нажмите её, чтобы улучшить [RAM](../basic/ram.md).

Рекомендую довести [RAM](../basic/ram.md) домашнего компьютера **как минимум** до 128 ГБ.
Больше — тоже хорошо.

## Покупаем свои первые аугментации

Набрав примерно 1000 [репутации](../basic/reputation.md) во [фракции](../basic/factions.md) `CyberSec`, можно купить у неё первую [аугментацию](../basic/augmentations.md).

Для этого перейдите на вкладку `Factions` в левом навигационном меню (Alt + f) и выберите `CyberSec`.
Внизу будет кнопка `Purchase Augmentations`.
Она откроет страницу со всеми [аугментациями](../basic/augmentations.md), доступными от `CyberSec`.
Часть из них сейчас может быть заблокирована.
Чтобы их открыть, понадобится больше [репутации](../basic/reputation.md) в `CyberSec`.

[Аугментации](../basic/augmentations.md) дают постоянные улучшения в виде множителей.
На раннем этапе игры они не слишком мощные, потому что множители невелики.
Однако эффекты [аугментаций](../basic/augmentations.md) перемножаются **друг с другом**, так что по мере установки всё большего числа [аугментаций](../basic/augmentations.md) их суммарный эффект заметно растёт.

Поэтому на раннем этапе я бы рекомендовал вкладываться скорее в улучшение [RAM](../basic/ram.md) домашнего компьютера, чем в [аугментации](../basic/augmentations.md).
Достаточная [RAM](../basic/ram.md) для запуска множества [скриптов](../basic/scripts.md) позволит зарабатывать намного больше денег, а к [аугментациям](../basic/augmentations.md) можно вернуться позже.

Прямо сейчас советую купить у `CyberSec` хотя бы аугментацию `Neurotrainer I`.
Если есть лишние деньги, также советую взять `BitWire` и несколько уровней аугментации `NeuroFlux Governor` (`NFG`).
Учтите: с каждой купленной [аугментацией](../basic/augmentations.md) **цена следующей растёт на 90%**, так что сначала покупайте самую дорогую [аугментацию](../basic/augmentations.md).
Не переживайте: как только вы установите [аугментации](../basic/augmentations.md), их цены сбросятся к изначальным значениям.

## Дальнейшие шаги

На этом пошаговая часть руководства заканчивается!
Продолжайте исследовать, что может предложить игра.
В этом руководстве не разобрана добрая часть возможностей, а ещё больше откроется по мере игры!

Также загляните в документацию API, чтобы узнать, что она предлагает.
Написание [скриптов](../basic/scripts.md) для выполнения и автоматизации разных задач — как по мне, именно в этом и заключается основное удовольствие от игры!

Вот несколько вещей, которыми стоит заняться в ближайшее время.

## Устанавливаем аугментации (и сбрасываемся)

Если вы купили какие-то [аугментации](../basic/augmentations.md), их нужно установить, чтобы получить их эффект.
Установка [аугментаций](../basic/augmentations.md) — это игровая механика «мягкого сброса», она же «престиж».

Чтобы установить [аугментации](../basic/augmentations.md), перейдите на вкладку `Augmentations` (Alt + a) в левом навигационном меню.
Вы увидите список всех купленных вами [аугментаций](../basic/augmentations.md).
Ниже будет кнопка `Install Augmentations`.
Предупреждаю: после нажатия отменить это нельзя (разве что загрузить более раннее сохранение).

## Автоматизируем запуск скриптов

При каждой установке [аугментаций](../basic/augmentations.md) все ваши [скрипты](../basic/scripts.md) останавливаются, и их придётся запускать заново.
Делать это вручную при каждой установке [аугментаций](../basic/augmentations.md) было бы утомительно и раздражающе, так что стоит написать [скрипт](../basic/scripts.md) для автоматизации этого процесса.
Вот простой пример стартового [скрипта](../basic/scripts.md).
Можете подогнать его под себя.

    /** @param {NS} ns */
    export async function main(ns) {
        // Array of all servers that don't need any ports opened
        // to gain root access. These have 16 GB of RAM
        const servers0Port = ["sigma-cosmetics",
                            "joesguns",
                            "nectar-net",
                            "hong-fang-tea",
                            "harakiri-sushi"];

        // Array of all servers that only need 1 port opened
        // to gain root access. These have 32 GB of RAM
        const servers1Port = ["neo-net",
                            "zer0",
                            "max-hardware",
                            "iron-gym"];

        // Copy our scripts onto each server that requires 0 ports
        // to gain root access. Then use nuke() to gain admin access and
        // run the scripts.
        for (let i = 0; i < servers0Port.length; ++i) {
            const serv = servers0Port[i];

            ns.scp("early-hack-template.js", serv);
            ns.nuke(serv);
            ns.exec("early-hack-template.js", serv, 6);
        }

        // Wait until we acquire the "BruteSSH.exe" program
        while (!ns.fileExists("BruteSSH.exe")) {
            await ns.sleep(60000);
        }

        // Copy our scripts onto each server that requires 1 port
        // to gain root access. Then use brutessh() and nuke()
        // to gain admin access and run the scripts.
        for (let i = 0; i < servers1Port.length; ++i) {
            const serv = servers1Port[i];

            ns.scp("early-hack-template.js", serv);
            ns.brutessh(serv);
            ns.nuke(serv);
            ns.exec("early-hack-template.js", serv, 12);
        }
    }

## Разные советы

- На раннем этапе игры лучше тратить деньги на улучшение [RAM](../basic/ram.md) и покупку новых облачных [серверов](../basic/servers.md), а не на [аугментации](../basic/augmentations.md)
- Чем больше денег доступно на [сервере](../basic/servers.md), тем эффективнее функции `hack()` и `grow()`.
  Дело в том, что обе эти функции оперируют процентами, а не фиксированными значениями.
  `hack()` похищает процент от всех доступных денег [сервера](../basic/servers.md), а `grow()` увеличивает деньги [сервера](../basic/servers.md) на X%.
- Существует предел того, сколько денег может быть на [сервере](../basic/servers.md).
  Это значение своё для каждого [сервера](../basic/servers.md).
  Узнать это максимальное значение можно функцией `getServerMaxMoney()`.
- На этом этапе игры ваши боевые характеристики (strength, defense и т. д.) далеко не так полезны, как характеристика хакинга.
  Не стоит тратить много времени или денег на набор опыта боевых характеристик.
- Как правило, целью взлома должен быть [сервер](../basic/servers.md) с наибольшим отношением `MaxMoney / MinimumSecurityLevel`, у которого `RequiredHackingLevel` меньше половины вашего уровня хакинга.
