# Автодополнение

Терминал BitBurner поддерживает автодополнение по Tab: нажав Tab после ввода команды, вы получите подсказки по возможным аргументам. Для своих скриптов это поведение можно настраивать.

Работает это через экспортируемую функцию с именем «autocomplete», размещённую _вне_ main, в базовой области видимости скрипта.

Эта функция должна возвращать массив, содержимое которого и станет вариантами автодополнения.

Простой пример в виде готового скрипта:

```javascript
/**
 * @param {AutocompleteData} data - context about the game, useful when autocompleting
 * @param {string[]} args - current arguments, not including "run script.js"
 * @returns {string[]} - the array of possible autocomplete options
 */
export function autocomplete(data, args) {
  return ["argument0", "argument1", "argument2"];
}

/** @param {NS} ns */
export function main(ns) {
  const args = ns.args;
  ns.tprint(args[0], args[1], args[2]);
}
```

Если запустить этот скрипт из терминала как `run script.js` или `./script.js` и нажать Tab, вариантами автодополнения будут «argument0», «argument1» и «argument2».

## AutocompleteData

Чтобы эта возможность стала полезнее, в функцию autocomplete передаётся объект [AutocompleteData](../../../../../markdown/bitburner.autocompletedata.md), содержащий информацию, которую обычно передают скриптам в аргументах: имена серверов, имена файлов и т. п.

AutocompleteData — это объект со следующими свойствами:

```javascript
  {
    command:    // Выполняемая команда в том виде, в котором она набрана в терминале.
    enums:      // Объект ns.enums с различными внутриигровыми строками.
    filename:   // Имя файла скрипта, содержащего функцию autocomplete.
    hostname:   // Имя сервера, на котором будет запущен скрипт.
    processes:  // Список всех процессов, запущенных на текущем сервере.
    servers:    // Список всех серверов в игре. Некоторые серверы скрыты, пока вы не выполните их требования. Если требования не выполнены, таких серверов в этом массиве не будет.
    txts:       // Список всех текстовых файлов на текущем сервере.
    scripts:    // Список всех скриптов на текущем сервере.
    flags:      // Функция, аналогичная ns.flags(). Её вызов добавляет все флаги в варианты автодополнения.
  }
```

Вот более полный пример, использующий и возвращающий информацию из объекта AutocompleteData.

```javascript
/**
 * @param {AutocompleteData} data - context about the game, useful when autocompleting
 * @param {string[]} args - current arguments, not including "run script.js"
 * @returns {string[]} - the array of possible autocomplete options
 */
export function autocomplete(data, args) {
  const scripts = data.scripts;
  const servers = data.servers;

  const gymTypesObject = data.enums.GymType; // The data.enums holds the enum information as objects.
  const gymTypes = Object.values(gymTypesObject); // We are only interested in the string values from the enums object.

  return [...scripts, ...servers, ...gymTypes]; // Offer a list of all servers, all scripts on the current server, and gym jobs ("str", "agi" etc) as autocomplete options.
}
```

## args

Вторым параметром в функцию autocomplete передаётся массив args. Как и ns.args, передаваемый в `main` обычных скриптов, этот массив содержит аргументы, введённые в терминал на данный момент.

С его помощью можно убирать из подсказок уже введённые аргументы.

Например:

```javascript
/**
 * @param {AutocompleteData} data - context about the game, useful when autocompleting
 * @param {string[]} args - current arguments, not including "run script.js"
 * @returns {string[]} - the array of possible autocomplete options
 */
export function autocomplete(data, args) {
  const servers = data.servers;
  const serversWithArgsRemoved = servers.filter((server) => !args.includes(server));

  return serversWithArgsRemoved;
}
```

В этом примере, если набрать `run script.js` и нажать Tab, сначала в подсказках будут все серверы. Затем, если добавить в аргументы «n00dles» и снова нажать Tab, «n00dles» в последующих подсказках уже не появится.

## data.flags

Это функция, работающая почти так же, как `ns.flags()`. Единственное отличие — она допускает неизвестные опции. Например:

```js
export function autocomplete(data, args) {
  const parsedFlags = data.flags([["foo", true]]);
  return [];
}

/** @param {NS} ns */
export async function main(ns) {
  const parsedFlags = ns.flags([["foo", true]]);
}
```

Если набрать в терминале `run a.js --f` и нажать Tab, то `parsedFlags` внутри `autocomplete` будет равен `{_: ["--f"], foo: true}`.

- `f` не определён в схеме, поэтому попадает в `_`.
- В команде не указан `foo`, поэтому `foo` принимает значение по умолчанию.

Если же набрать `run a.js --f` и нажать Enter, будет выброшена ошибка:

```
ArgError: unknown or unexpected option: --f
```

Так происходит потому, что `f` не определён в схеме, а `ns.flags` неизвестные опции не допускает.

# Примечания

- Функция autocomplete в файле вызывается каждый раз при нажатии Tab после `run file.js` или `./file.js` в терминале.
- Функция autocomplete отделена от `main` и не получает `ns` в параметрах. Это значит, что никакие игровые команды `ns` в функциях autocomplete не работают.
- Если возвращён массив из нескольких элементов, показывается несколько вариантов. Если массив из одного элемента, этот элемент автоматически подставляется в терминал. Это удобно, например, для аргумента запуска «--tail».
