# @velund/core

Core абстракции и интерфейсы для системы UI-компонентов Velund.

Этот пакет предоставляет базовые типы и вспомогательные функции (`defineVelundRenderer`, `defineVelundGenerator`, `defineComponent`), которые служат основой для создания UI-компонентов, рендереров и генераторов, совместимых с Velund. Он определяет универсальный язык для взаимодействия внутри экосистемы Velund, обеспечивая совместимость между различными шаблонизаторами и бэкенд-интеграциями.

## 🚀 Ключевые концепции

- **`VelundRendererDescriptor`**: Интерфейс для любого шаблонизатора, который может рендерить компоненты Velund (например, Twig, Jinja, HTML).
- **`VelundGeneratorDescriptor`**: Интерфейс для модулей, которые генерируют код или библиотеки, специфичные для бэкенда, для взаимодействия с компонентами Velund (например, Python, PHP, Node.js).
- **`VelundComponentDescriptor`**: Описывает UI-компонент Velund, включая его имя, шаблон, схемы данных (`propsSchema`, `contextSchema`) и опциональную функцию `prepare`.
- **`defineVelundRenderer(...)`**: Вспомогательная функция для корректного определения и типизации рендерера Velund.
- **`defineVelundGenerator(...)`**: Вспомогательная функция для корректного определения и типизации генератора Velund.
- **`defineComponent(...)`**: Вспомогательная функция для определения UI-компонента Velund.

## 📦 Установка

```bash
npm install @velund/core
# или
yarn add @velund/core
# или
pnpm add @velund/core
```

Этот пакет обычно является зависимостью для `velund` (основного плагина Vite) и специфических пакетов рендереров/генераторов, поэтому вам, возможно, не потребуется устанавливать его напрямую, если только вы не разрабатываете собственные расширения Velund.

## 🛠️ Использование

Вы будете в основном использовать типы и вспомогательные функции, предоставляемые этим пакетом, при разработке пользовательских рендереров, генераторов или самого основного плагина `velund`.

### `defineVelundRenderer(...)`

**Пример использования defineVelundRenderer**

```typescript
// my-custom-renderer/src/index.ts
import { defineVelundRenderer, VelundComponentDescriptor } from '@velund/core';

const myCustomRenderer = defineVelundRenderer(() => {
  const components = new Map<string, VelundComponentDescriptor>();
  return {
    id: 'my-custom-templating-engine',
    templateExtensions: ['.myhmtl'], // Расширения файлов шаблонов, которые обрабатывает этот рендерер
    setComponents(items: VelundComponentDescriptor[]) {
      // Метод для установки компонентов, которые этот рендерер будет обрабатывать
      components.clear();
      items.forEach((comp) => components.set(comp.name, comp));
    },
    async render(name: string, context: Record<string, any>, meta?: boolean) {
      // Основной метод рендеринга компонента
      const component = components.get(name);
      if (!component) throw new Error(`Компонент не найден: ${name}`);
      let finalContext = { ...context };
      if (component.prepare) {
        finalContext = await component.prepare(context);
      }
      // Логика для рендеринга component.template с использованием 'my-custom-templating-engine'
      const html = `<h1>Привет от ${name}!</h1><pre>${JSON.stringify(finalContext, null, 2)}</pre>`;

      return meta ? html : { html, context }; // Должен возвращать объект строку html или объект {html, context} (если указан meta)
    },
  };
});

export default myCustomRenderer;
```

Хорошо, без глобального туториала. Вот описание для `defineVelundGenerator`, которое можно вставить в `core/README.md`:

---

### `defineVelundGenerator(...)`

`defineVelundGenerator` — это вспомогательная функция, которая используется для корректного определения и типизации генератора Velund. Она принимает функцию, которая возвращает дескриптор генератора (`VelundGeneratorDescriptor`). Этот дескриптор описывает поведение вашего генератора для основного плагина `velund`.

**Основные свойства дескриптора генератора:**

- **`id: string`**: Уникальный идентификатор вашего генератора (например, `'python'`, `'node'`, `'php'`). Этот ID будет использоваться в конфигурации `velund` в `vite.config.ts`.
- **`initialize?(options?: any): void`**: (Опционально) Метод, который вызывается один раз при инициализации плагина `velund`. Здесь можно выполнить настройку генератора, используя переданные опции.
- **`generate(components: VelundComponentDescriptor[], outputDir: string): Promise<void>`**: Основной асинхронный метод генерации. Он вызывается плагином `velund` во время сборки (`vite build`).
  - `components`: Массив объектов `VelundComponentDescriptor`, содержащих всю информацию о фронтенд-компонентах (их имена, ссылки на шаблоны, `propsSchema`, `contextSchema` и ссылку на метод `prepare`).
  - `outputDir`: Полный путь к директории, куда генератор должен сохранить все сгенерированные файлы (библиотеки, DTO, вспомогательные файлы).

**Пример использования `defineVelundGenerator`:**

```typescript
import { defineVelundGenerator, VelundComponentDescriptor } from '@velund/core';
import { promises as fs } from 'fs';
import { join } from 'path';

// Опции для вашего генератора
export interface MyCustomGeneratorOptions {
  namespace: string;
}

// Фасадная функция, которую будут импортировать в vite.config.ts
export default function myCustomGenerator(options?: MyCustomGeneratorOptions) {
  // defineVelundGenerator ожидает функцию, возвращающую дескриптор
  return defineVelundGenerator(() => {
    let settings = { namespace: 'Default' }; // Базовые настройки генератора

    return {
      id: 'my-custom-lang', // Уникальный ID генератора

      // Метод инициализации
      initialize(opts?: MyCustomGeneratorOptions) {
        if (opts) {
          settings = { ...settings, ...opts };
        }
        console.log(
          `[MyCustomGenerator] Инициализация с настройками: ${settings.namespace}`
        );
      },

      // Основной метод генерации кода
      async generate(
        components: VelundComponentDescriptor[],
        outputDir: string
      ) {
        const langSpecificDir = join(
          outputDir,
          settings.namespace.toLowerCase()
        );
        await fs.mkdir(langSpecificDir, { recursive: true });

        const generatedFiles: string[] = [];
        for (const component of components) {
          // Здесь будет логика преобразования VelundComponentDescriptor
          // в код целевого языка (например, Python, PHP, JS, C#)
          // Включая генерацию DTO на основе propsSchema/contextSchema
          // и код для Render-класса, который будет вызывать prepare-методы.

          const content =
            `// Сгенерированный код для ${component.name} в пространстве имен ${settings.namespace}\n` +
            `// props: ${JSON.stringify(component.propsSchema)}\n` +
            `// context: ${JSON.stringify(component.contextSchema)}`;
          const filePath = join(
            langSpecificDir,
            `${component.name}.custom_lang`
          );
          await fs.writeFile(filePath, content);
          generatedFiles.push(filePath);
        }
        console.log(
          `[MyCustomGenerator] Сгенерированы файлы: ${generatedFiles.join(', ')}`
        );
      },
    };
  });
}
```
