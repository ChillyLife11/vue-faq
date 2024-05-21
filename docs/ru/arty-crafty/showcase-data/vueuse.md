---
pageClass: arty-crafty
head:
  - - meta
    - name: og:site_name
      content: Arty-Crafty - проект онлайн магазина
  - - meta
    - name: og:image
      content: /images/arty-crafty-logo.png
  - - meta
    - name: twitter:image
      content: /images/arty-crafty-logo.png
---

# VueUse

## useLocalStorage

В проекте у нас есть несколько переменных, значения которых надо сохранять в `LocalStorage` - например, `isDarkMode`, `favourites`, `shoppingCart`. Также, для UX полезно, чтобы эта связь была реактивной - тогда при нескольких открытых вкладках изменение корзины или цветовой темы в одной сразу будет отражаться в другой.

Для этого была реализована [своя версия](https://github.com/vuesence/arty-crafty/blob/main/src/app/composables/useLocalStorage.ts) `useLocalStorage`.

Любую реактивную переменную проекта можно зарегистрировать в этом композабле, и её значение станет динамично отображаться в `LocalStorage` и обратно:

```js
// init.ts
import { useLocalStorage } from "@/app/composables/useLocalStorage";

const ls = useLocalStorage();

export function init() {
  ls.init();
  ls.observe("favourites", favourites);
  ls.observe("isDarkTheme", isDarkTheme);
}
```

Все переменные будут храниться в одном объекте в `LocalStorage` , а не каждая в своем.

## Библиотека VueUse

VueUse - популярная библиотека, состоящая из небольших утилит. Нисколько не умаляя вклад Anthony Fu в экосистему Vue, опишу, почему я не использую её и не рекомендую для серьезных проектов и опытных разработчиков.

1. Все утилиты в ней - **небольшие**. Под "небольшие" я подразумеваю, что аналогичный свой код будет в 10-50 строк. В зависимости от обстоятельств и требований, свой код может быть и в пять строк (аналог `onClickOutside`), и в две (использование `LocalStorage`). Кроме того, на проект нужно обычно всего 2-3 подобных утилит оттуда. **Подключать из-за этого всю библиотеку нецелесообразно**, несмотря на tree-shaking.

2. **Это зависимость**, и как все зависимости может измениться, устареть, получить уязвимость, тормозить развитие проекта и прочее, прочее. С большинством активно развивавшихся библиотек такое случалось.

3. Утилиты в ней **не делают то, что надо именно тебе**. Например, мне для адаптивности нужно ставить соответствующий CSS класс на `<body>` при изменении ширины экрана. В библиотеке есть две утилиты, работающие с шириной экрана, но ни одна не дает именно то, что надо. Приходится преобразовывать данные от `useBreakponts` в свои, и весь код занимает половину кода [своей реализации](https://github.com/vuesence/arty-crafty/blob/main/src/app/composables/useScreenWidth.ts), которая достаточно проста и прямолинейна, и вполне может быть даже сгенерирована AI ботом.

4. В утилитах `VueUse` содержится **избыточная и чрезмерная логика и код** (с точки зрения вашего конкретного проекта). Если бы это был простой js код, было бы еще ничего, но **это реактивный код**. Если открыть исходники `useBreakponts`, то можно увидеть с десяток `computed` переменных, которые ты не используешь. Но учитывая, что только в версии 3.4 Vue пофиксила баг, при котором компьютеды пересчитывались, хотя ни одна из входящих в них переменная не менялась, нет никаких гарантий, что при изменении экрана или сдвиге мышки не будет производиться множество не нужных тебе пересчетов. Это влияет на производительность.

5. Указанная выше проблема, это не недостаток именно VueUse. Попробуйте написать универсальный компонент, который любой разработчик мог бы использовать. Примерно за 20% времени вы напишите вполне рабочий, подходящий вашему проекту компонент, и оставшиеся 80% будете лепить на него обвесы, чтобы он был универсальный и опциональный, учитывал Vue 2 и Vue 3, был SSR-safe и еще много чего, что конкретно вам не нужно. В итоге лаконичный красивый эффективный код превратится во **множество проверок, адаптеров и прочей не функциональной логики**. Поэтому написать свой код всегда намного эффективней использования чужого универсального. Речь сейчас идет конкретно про подобные утилитарные библиотеки, а не, к примеру, UI пакеты.

6. `useLocalStorage` от VueUse возвращает реактивную переменную, которую, по идее, нужно напрямую использовать в проекте. Другими словами, если мне надо сохранять в `LocalStorage` `shoppingCart`, то ее нужно создать через вызов `useLocalStorage()`. То есть, одна из основных бизнес-сущностей моего проекта должна создаваться какой-то сторонней библиотекой. **Чужой код простой утилиты диктует реализацию элементов архитектуры всего проекта**. Мне такое кажется абсолютно неприемлимым.

7. Можно создать отдельно `shoppingCart` и отдельно переменную через `useLocalStorage()` и затем заморочится с их синхронизацией через `watch` и предотвращением бесконечных циклов. И это для каждой такой переменной. Можно совсем довериться VueUse и использовать их `syncRef`. Но это уже даже не неприемлимо, а смешно, не говоря о том, что код в итоге получится больше, чем своё решение. Таким образом, утилиты VueUse бывают **просто неудобными в применении**. `Lodash` при использовании был очень удобным - берешь её функцию, и она максимально эффективно делает то, что надо. Из того, что я пытался применить в своих проектах, VueUse неудобен.

8. Есть большая разница между популяризацией (хорошо) и популизмом (плохо). Советы по использованию и даже навязывание VueUse разработчикам, без понимания ими того, что именно они получают и за какую цену, смахивает на второе.

::: info Wikipedia
Попули́зм (от лат. populus — народ) — политика, апеллирующая к широким массам и обещающая им скорое и лёгкое решение острых социальных проблем.
В основе популизма лежит стремление той или иной политической силы завоевать доверие и поддержку масс, понравиться народу. При этом реальные цели политиков-популистов (борьба за власть, обогащение и тому подобное), как правило, прикрываются социально-привлекательными идеями.
:::