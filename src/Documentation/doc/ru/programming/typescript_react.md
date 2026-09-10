# Как использовать TypeScript и React в игре

Bitburner поддерживает TypeScript и React «из коробки».

Скрипты на TypeScript пишутся в файлах `.ts`, а синтаксис jsx доступен в файлах `.jsx` и `.tsx`. Запускать их можно командой `run`, как обычные JS-файлы. Например, файл `timer.tsx` из следующего раздела запускается командой `run timer.tsx` на вкладке терминала.

## Пример

Используйте `ns.printRaw` и `ns.tprintRaw`, чтобы отрисовывать React-элементы в логах и терминале.

```tsx
// timer.tsx
function Timer() {
  const [seconds, setSeconds] = React.useState(0);

  React.useEffect(() => {
    const interval = setInterval(() => {
      setSeconds((seconds) => seconds + 1);
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  return <div>Seconds: {seconds}</div>;
}

export async function main(ns: NS) {
  ns.ui.openTail();
  ns.printRaw(<Timer />);
  await ns.asleep(10000);
}
```
