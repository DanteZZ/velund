# Базовая настройка `vite.config.ts`

После установки Velund необходимо подключить плагин и зарегистрировать рендереры/генераторы в `vite.config.ts`.  
Ниже приведён базовый пример для связки **Twig + PHP**.

```ts
import { defineConfig } from 'vite';
import velund from 'velund';
import twigRenderer from '@zebrains/velund-twig';
import phpGenerator from '@zebrains/velund-php';

export default defineConfig({
  plugins: [
    velund({
      // Основной рендерер (Twig)
      renderer: 'twig',

      // Основной генератор (PHP)
      generator: 'php',

      // Подключаем сторонние рендереры (Twig)
      renderers: [twigRenderer()],

      // Подключаем сторонние генераторы (PHP)
      generators: [phpGenerator()],

      // URL, по которому будут доступны ассеты
      assetsUrl: '/assets',

      // API-роут для runtime-рендеринга
      renderUrl: '/__render',

      // Если true — будет строгая проверка расширений шаблонов
      strictTemplateExtensions: false,
    }),
  ],
});
```

---

## Объяснение ключевых опций

| Поле                       | Описание                                                                                                                                   |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `renderer`                 | Указывает основной рендерер, например `twig`, `jinja` или `html`.                                                                          |
| `generator`                | Определяет основной генератор backend-библиотеки (`php`, `node`, `python`).                                                                |
| `renderers`                | Список подключаемых сторонних рендереров (например, Twig или Jinja). ⚠️ Рендерер **HTML встроен по умолчанию** и его указывать не нужно.   |
| `generators`               | Список подключаемых сторонних генераторов (например, PHP или Python). ⚠️ Генератор **Node встроен по умолчанию** и его указывать не нужно. |
| `assetsUrl`                | Путь, по которому будут отдаваться статические файлы (CSS/JS).                                                                             |
| `renderUrl`                | URL-эндпоинт, через который можно рендерить компоненты в рантайме.                                                                         |
| `strictTemplateExtensions` | Если `true`, Velund будет требовать строгое совпадение расширений файлов шаблонов.                                                         |

---

## Пример: Twig + Node.js

Так как генератор **Node.js** встроен в Velund, его не нужно импортировать или регистрировать вручную:

```ts
import { defineConfig } from 'vite';
import velund from 'velund';
import twigRenderer from '@zebrains/velund-twig';

export default defineConfig({
  plugins: [
    velund({
      renderer: 'twig',
      generator: 'node',
      renderers: [twigRenderer()],
    }),
  ],
});
```

👉 Для **HTML**-шаблонов вообще не нужно регистрировать рендереры — достаточно указать `renderer: 'html'`.
