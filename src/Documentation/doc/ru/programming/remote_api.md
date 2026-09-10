# Remote API

Bitburner может подключаться к WebSocket-серверу, и такой сервер сможет читать и записывать данные Bitburner через ряд API. Чаще всего эта возможность используется для синхронизации файлов между Bitburner и внешней системой. С инструментом Remote API можно писать скрипты в любом текстовом редакторе и синхронизировать их с Bitburner.

Нужно сделать всего 2 вещи:

- Запустить инструмент Remote API.
- В Bitburner: Options -> Remote API. Укажите «hostname» и «port», затем нажмите «Connect».

## Инструменты сообщества

Все эти инструменты поддерживают синхронизацию скриптов с Bitburner. Некоторые из них умеют транспилировать TypeScript/JSX в JavaScript.
Учтите, что сама Bitburner поддерживает TypeScript/JSX нативно.

Ссылки:

- [typescript-template](https://github.com/bitburner-official/typescript-template): шаблон для синхронизации Typescript/Javascript с вашего компьютера в игру.
- [viteburner](https://github.com/Tanimodori/viteburner): демон-инструменты для bitburner на базе vite: преобразование скриптов, синхронизация файлов, мониторинг RAM и многое другое!
- [bb-external-editor](https://github.com/shyguy1412/bb-external-editor): использует esbuild для транспиляции и сборки ваших скриптов. Из коробки поддерживает JS, TS и React, а также импорт из любой npm-библиотеки, совместимой с браузером.
- [BitburnerGoFilesync](https://github.com/CTNOriginals/BitburnerGoFilesync): самостоятельный бинарный CLI-инструмент, не требующий настройки или сторонних библиотек. Спроектирован максимально минималистичным и простым в использовании из коробки.
- [VS Code Extension: Bitburner File Sync Plugin](https://github.com/ficocelliguy/bitburner-file-sync-plugin): расширение для VS Code, синхронизирующее ваши локальные файлы скриптов с Bitburner.

У `typescript-template` и `BitburnerGoFilesync` небольшой набор опций и функций — их простота задумана намеренно.  
У `viteburner`, `bb-external-editor` и `VS Code Extension: Bitburner File Sync Plugin` возможностей больше, и они могут дать больше контроля для конкретных сценариев использования.

## Советы по устранению неполадок

- Попробуйте обновить инструмент и перезапустить его. Посмотрите сообщения об ошибках в терминале, чтобы понять, что пошло не так.
- При выключении машины или переходе в спящий режим соединение между Bitburner и инструментом разрывается. Придётся подключиться заново.
- Некоторые внешние программы или расширения браузера могут мешать соединению. Например, некоторые антивирусы и блокировщики рекламы могут блокировать WebSocket-соединение.
- Некоторые инструменты поддерживают функцию, которую обычно называют «зеркалированием» (mirroring). Перед её использованием обязательно внимательно прочитайте инструкцию. Эта функция даёт двустороннюю синхронизацию, но при неверной настройке может перезаписать ваши скрипты или другие файлы _на вашем компьютере_.
- Если нужна дополнительная помощь, спрашивайте нас в канале [external-editors](https://discord.com/channels/415207508303544321/923428435618058311).

## Как это работает

![remote-file-api-sequence-diagram.svg](../../../images/remote-file-api-sequence-diagram.svg)

## Спецификация API

### Обзор

Все API используют формат запрос/ответ, похожий на протокол JSON RPC 2.0.

Неизвестные параметры в запросах игнорируются.

Постраничная выдача не поддерживается.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": string,
            "params": any
        }

Успешный ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": any
        }

Ответ с ошибкой:

        {
            "jsonrpc": "2.0",
            "id": number,
            "error": string
        }

### Список API

#### pushFile

Создаёт или обновляет файл.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "pushFile",
            "params": {
                "filename": string,
                "content": string,
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": "OK"
        }

#### getFile

Читает файл и его содержимое.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getFile",
            "params": {
                "filename": string,
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": string
        }

#### getFileMetadata

Читает метаданные файла.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getFileMetadata",
            "params": {
                "filename": string,
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": {
                "filename": string,
                "size": number,
                "atime": number,
                "btime": number,
                "mtime": number
            }
        }

#### deleteFile

Удаляет файл.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "deleteFile",
            "params": {
                "filename": string,
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": "OK"
        }

#### getFileNames

Возвращает список имён всех файлов на сервере.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getFileNames",
            "params": {
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": string[]
        }

#### getAllFiles

Получает содержимое всех файлов на сервере.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getAllFiles",
            "params": {
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": {
                "filename": string,
                "content": string
            }[]
        }

#### getAllFileMetadata

Получает содержимое всех файлов на сервере.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getAllFileMetadata",
            "params": {
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": {
                "filename": string,
                "size": number,
                "atime": number,
                "btime": number,
                "mtime": number
            }[]
        }

#### calculateRam

Вычисляет внутриигровой расход RAM скрипта.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "calculateRam",
            "params": {
                "filename": string,
                "server": string
            }
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": number
        }

#### getDefinitionFile

Получает файл определений NS API.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getDefinitionFile"
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": string
        }

#### getSaveFile

Получает данные сохранения.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getSaveFile"
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": {
                "identifier": string,
                "binary": boolean,
                "save": string
            }
        }

#### getAllServers

Получает список всех серверов.

Запрос:

        {
            "jsonrpc": "2.0",
            "id": number,
            "method": "getAllServers"
        }

Ответ:

        {
            "jsonrpc": "2.0",
            "id": number,
            "result": {
                "hostname": string,
                "hasAdminRights": boolean,
                "purchasedByPlayer": boolean
            }[]
        }
