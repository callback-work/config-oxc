# config-oxc

Общие конфиги [oxlint](https://oxc.rs/docs/guide/usage/linter), разбитые на слои.

| Слой | Что внутри | Подключать |
|---|---|---|
| `core` | eslint, typescript, unicorn, oxc, import, promise, jsdoc | неявно, через `app` |
| `app` | `core` + прикладные запреты (`no-console`, `no-alert`) | всегда |
| `react` | react, react-perf (+ PascalCase для имён файлов компонентов) | React-проекты |
| `next` | nextjs | Next.js-проекты, вместе с `react` |
| `a11y` | jsx-a11y | проекты с JSX |
| `test` | vitest + ослабления `core` — только для `*.test.*` и `__tests__/` | проекты с тестами |

Каждое правило каждого плагина прописано в конфигах явно (`error` / `warn` / `off`), поэтому поведение не зависит от дефолтов oxlint. Vue и jest не используются.

## Установка

```sh
npm i -D oxlint git+ssh://git@github.com/callback-work/config-oxc.git#v0.1.0
# для type-aware правил (рекомендуется):
npm i -D oxlint-tsgolint
```

## Подключение

`.oxlintrc.json` в корне проекта:

```jsonc
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "extends": [
    "./node_modules/config-oxc/configs/app.json",
    "./node_modules/config-oxc/configs/react.json",
    "./node_modules/config-oxc/configs/next.json",
    "./node_modules/config-oxc/configs/a11y.json",
    "./node_modules/config-oxc/configs/test.json"
  ],
  "options": { "typeAware": true }
}
```

- **Указывайте пути к файлам, а не имена пакетов.** oxlint не резолвит `"config-oxc/app"` в `extends`.
- **`app` всегда первым.** Только он наследует `core`; остальные слои — добавочные и `core` не тянут, иначе они сбросили бы настройки `app`.
- Лишние слои просто убирайте из списка.

### Type-aware правила

51 правило из `core` (`no-floating-promises`, `no-unsafe-*`, `switch-exhaustiveness-check` и др.) требует информации о типах. Они работают, только если:

1. в проекте установлен `oxlint-tsgolint`;
2. включён `"options": { "typeAware": true }` (или флаг `--type-aware`).

Без этого они молча не запускаются — остальные правила работают как обычно.

### Свои компоненты и a11y

a11y-правила не знают, что `<Button>` — это кнопка. Сопоставьте компоненты с HTML-тегами в конфиге проекта:

```jsonc
"settings": { "jsx-a11y": { "components": { "Button": "button", "Link": "a" } } }
```

### React Compiler

Если проект использует React Compiler, включите у себя:

```jsonc
"rules": {
  "react/preserve-manual-memoization": "error",
  "react/incompatible-library": "error"
}
```

## Ключевые соглашения

- Имена файлов — `kebab-case`; в React-проектах разрешён и `PascalCase`.
- `interface` вместо `type` для объектных типов; без parameter properties, без `const enum`, без `namespace`.
- Без `any`, `!` (non-null assertion), `!!`, `++`, `forEach`.
- Обработчики событий именуются `onX`.
- Тестовые файлы — `*.test.ts(x)`, импорты из `vitest` явные.
- `complexity` ≤ 15, `max-params` ≤ 4.

## Разработка

```sh
npm i
npx oxlint -c configs/app.json some-file.ts   # проверить слой на файле
```

При обновлении oxlint проверьте новые правила: `npx oxlint --rules --format json` и сверьте со списком в конфигах — новые правила не попадут ни в один слой, пока их явно не добавить.
