## 
Вы правы, в документации Geoff указан именно npm-вариант установки. Проблема была в том, что я изначально повёл вас по пути компиляции из Rust, что и вызвало ошибку. Ниже — исправленный файл workflow, который использует `npm install -g @chapeaux/geoff`, как и написано в официальном репозитории.

Актуальная версия Node.js на сентябрь 2026 года: **24.x — Active LTS**, **26.x — Current** . Для GitHub Actions надёжнее использовать LTS-линию, то есть Node.js 24. Если npm-пакет Geoff потребует более новую версию, можно будет переключиться на 26.

### Полный файл `.github/workflows/deploy.yml` для замены

```yaml
name: Deploy Geoff Semantic Wiki

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '24'

      - name: Install Geoff
        run: npm install -g @chapeaux/geoff

      - name: Verify Geoff
        run: geoff --version

      - name: Build Site
        run: geoff build --output-dir dist

      - name: Upload Pages Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: "dist"

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Что здесь важно

| Элемент | Значение | Пояснение |
|---|---|---|
| `node-version` | `'24'` | Active LTS на сентябрь 2026 года. Стабильнее, чем Current (26.x)  |
| `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24` | `true` | Убирает предупреждение о том, что `actions/checkout@v4` и `actions/setup-node@v4` всё ещё используют Node.js 20 под капотом |
| `npm install -g @chapeaux/geoff` | официальный npm-способ | Именно так указано в репозитории Geoff  |
| `geoff --version` | проверка установки | Покажет в логах, что CLI доступен и какой версии. Если шаг упадёт — сразу увидите, что пакет не установился |
| `geoff build --output-dir dist` | сборка сайта | Создаёт статику в папке `dist`, которую затем публикует GitHub Pages |

### Что делать, если npm-пакет снова упадёт

Если `npm install -g @chapeaux/geoff` завершится ошибкой, проверьте на вкладке **Actions** текст ошибки в шаге **Install Geoff**. Возможные причины:

1. **Пакет не опубликован в npm-реестре** (только в crates.io). Тогда npm-вариант не сработает, и придётся возвращаться к Rust. Но поскольку в документации Geoff npm-установка указана явно, пакет должен существовать.
2. **Пакет требует Node.js новее 24**. Тогда замените `node-version: '24'` на `'26'` (Current) и попробуйте снова.
3. **Пакет является обёрткой над Rust-бинарником и требует системных зависимостей**. В этом случае в логе будет что-то про `node-gyp`, `prebuild` или отсутствующие библиотеки. Тогда потребуется добавить шаг установки системных пакетов (например, `sudo apt-get install -y librdf0-dev`), но это уже менее вероятный сценарий.

После замены файла сделайте коммит в `main` — workflow запустится автоматически. Если снова будет ошибка, пришлите текст из шага **Install Geoff** или **Build Site**, и я помогу разобраться.

### Ссылки

- Официальная инструкция по установке Geoff (crates.io / npm) — https://lib.rs/crates/geoff-reading-time
- Расписание релизов Node.js (GitHub) — https://github.com/nodejs/Release
- setup-node — https://github.com/actions/setup-node
- @chapeaux/geoff (npm) — https://www.npmjs.com/package/@chapeaux/geoff

## 3

Просмотр вашего workflow и анализ репозитория Geoff показывают, что проблема, скорее всего, вызвана особенностью npm-пакета `@chapeaux/geoff`. Он не содержит сам исполняемый файл, а лишь **скрипт-загрузчик**, который при установке пытается скачать готовый бинарник Geoff с GitHub Releases. В окружении GitHub Actions этот процесс может сбоить из-за сетевых ограничений, отсутствия подходящего бинарника для архитектуры runner'а или ошибок в самом скрипте загрузки.

### 🛠️ Что можно сделать

**Вариант 1. Использовать `npx` вместо глобальной установки**

Вместо `npm install -g @chapeaux/geoff` попробуйте вызывать инструмент через `npx`. Это исключает фазу глобальной установки и позволяет `npx` самому разрешить зависимости:

```yaml
- name: Build Site
  run: npx @chapeaux/geoff build --output-dir dist
```

Это не гарантирует решение, если проблема в самом скрипте `install.js`, но это более простой путь, который часто обходит ошибки, связанные с правами или кэшем npm.

**Вариант 2. Установить Geoff через Cargo (Rust)**

Если npm-пакет не работает, можно вернуться к установке через Rust. Ранее это тоже вызывало ошибку, но, возможно, она была связана с отсутствием nightly-тулчейна. Попробуйте такой шаг:

```yaml
- name: Setup Rust
  uses: dtolnay/rust-toolchain@stable

- name: Install Geoff
  run: cargo install chapeaux-geoff
```

Учтите, что компиляция может занять 10–15 минут.

**Вариант 3. Перейти на Rete**

Если Geoff продолжает сопротивляться, самым надёжным решением будет вернуться к **Rete** — инструменту, который мы уже разбирали. Rete:

- Не требует компиляции (работает через WASM в браузере).
- Устанавливается как обычный npm-пакет (`rete-graph`).
- Выполняет SPARQL-запросы **на клиенте**, как вы и хотели.
- Имеет рабочий пример, который мы уже разбирали ранее.

Для Rete вообще не нужен GitHub Actions — достаточно положить `.rete` файл и HTML-страницу в репозиторий.

**Вариант 4. Использовать Oxigraph WASM напрямую**

Если цель — просто клиентский SPARQL без сложной сборки, можно вообще отказаться от Geoff и использовать **Oxigraph WASM** через CDN. Вы пишете `.ttl` файл вручную, подключаете Oxigraph в HTML и выполняете запросы в браузере. Это работает без Node.js, без GitHub Actions и без CLI.

### 💎 Что я рекомендую

Самый быстрый путь к работающему результату — **Rete** или **Oxigraph WASM**. Geoff — мощный инструмент, но его npm-пакет, судя по всему, не предназначен для запуска в CI без дополнительной настройки. Если вы всё же хотите продолжить с Geoff, пришлите текст ошибки из шага **Install Geoff** или **Build Site** — я помогу разобраться точнее.

### 🔗 Ссылки

- Geoff (crates.io) — https://crates.io/crates/chapeaux-geoff
- Rete (GitHub) — https://github.com/caviri/rete
- rete-graph (npm) — https://www.npmjs.com/package/rete-graph
- Oxigraph (npm) — https://www.npmjs.com/package/oxigraph
- Oxigraph (GitHub) — https://github.com/oxigraph/oxigraph
