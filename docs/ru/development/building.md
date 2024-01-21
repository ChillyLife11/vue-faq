# Создание и сборка приложения

::: details Сборка приложения

Чтобы собрать Vue проект в файлы, понятные браузеру, нужен сборщик. Самые популярные - `Vite` и `Webpack`

Vite более новый и использует современные технологии, включая `esbuild` для `dev` сборок для транспиляции TypeScript в JavaScript, который в 20~30 раз быстрее чем обычный tsc, что делает HMR в десятки раз быстрей, чем на Webpack. Для билда JavaScript для `production` используется сборщик `Rollup`.

Нужно учитывать, что каждый сборщик собирает ресурсы приложения (иконки, SVG и пр.) по-своему, используя свои плагины, и требует в этом настройки.

Оба сборщика позволяют расширять свои возможности кастомными плагинами.

:::

::: details Vite или Webpack?

Vite.

Если вам понадобится Webpack, вы будете знать это сами.

:::

::: details npm, yarn или pnpm?

Рекомендуется `pnpm` как наиболее современный

Кроме того, он, используя жесткие ссылки вместо копирования файлов, очень сильно сохраняет дисковое пространство на компьютере в случае нескольких проектов с `node_modules` и одинаковыми пакетами в них.

:::

::: details Создание Vue приложения

`Vue-cli` - устаревший способ

Создание напрямую через Vite:

```sh
pnpm create vite
```

Либо в онлайн песочнице.

Подробнее - на [сайте Vite](https://vitejs.dev/guide/)

:::

::: details Как настроить алиас @?

Алиасы - настройка сборщика, и для каждого сборщика устанавливается по-своему.

Для того, чтобы использовать алиас @ в путях в проекте с Vite

```js
import BaseIcon from "@/components/ui/BaseIcon.vue";
```

нужно настроить его в `vite.config.j(t)s`:

```js
// vite.config.js
import path from "node:path";

export default defineConfig({
  build: {
    target: "esnext",
  },
  plugins: [...],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },

// package.json
"devDependencies": {
    "@types/node": "^20.3.1",
    ...
}

```

Если проект с поддержкой TypeScript, то и в `tsconfig.json`:

```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": { "@/*": ["./src/*"] }
    // ...
  }
}
```

Также может требуеться настройка алиаса в ESLint конфиге для плагинов типа `eslint-plugin-import` и `eslint-import-resolver-typescript`, если они используются.

:::

::: details Как посмотреть, сколько в моем js бандле занимают разные библиотеки?

[rollup-plugin-visualizer](https://github.com/btd/rollup-plugin-visualizer) - отличное средство для этого

Запускается генерация статистики из командной строки так (один из вариантов):

```sh
npx vite-bundle-visualizer
npx vite-bundle-visualizer -t list
npx vite-bundle-visualizer -t raw-data
```

:::
