## 1

Я проверил содержимое репозитория. Все необходимые файлы на месте: `geoff.toml`, папка `content/` с тремя заметками, `ontology/mappings.toml` и workflow в `.github/workflows/deploy.yml`. Структура полностью соответствует инструкции, которую я давал ранее. Проблема, скорее всего, не в файлах, а в **настройках GitHub Pages** или в **сбое workflow**.

### 🔍 Что я увидел в репозитории

- **`geoff.toml`** — конфигурация корректна: `base_url = "/geoff_test"`, `content_dir = "content"`, `output_dir = "dist"`.
- **`content/`** — содержит `index.md`, `hello.md`, `bob.md`.
- **`ontology/mappings.toml`** — на месте.
- **`.github/workflows/deploy.yml`** — workflow запускается на `push` в `main`, устанавливает Node.js, Geoff, собирает сайт и деплоит на GitHub Pages.

**Почему вы видите только README.md:** GitHub Pages, вероятно, всё ещё настроен на **«Deploy from a branch»** (публикацию содержимого ветки `main`), а не на **«GitHub Actions»**. Когда источник — ветка, GitHub Pages отдаёт корень репозитория, где лежит только `README.md`. Собранный Geoff сайт находится в папке `dist`, которая создаётся workflow-ом и публикуется как артефакт — но только если в настройках Pages выбран источник **GitHub Actions**.

### 🛠️ Что нужно проверить и исправить

**1. Переключите источник GitHub Pages на GitHub Actions**

- Откройте настройки репозитория: **Settings → Pages**.
- В разделе **Build and deployment → Source** выберите **GitHub Actions** (не «Deploy from a branch»).
- Нажмите **Save**.

После этого GitHub Pages начнёт публиковать артефакт, который создаёт workflow, а не содержимое ветки.

**2. Проверьте статус последнего запуска workflow**

- Откройте вкладку **Actions** в репозитории.
- Найдите последний запуск (он будет помечен вашим сообщением коммита).
- Если запуск **красный** — откройте его и посмотрите, на каком шаге произошла ошибка.
- Наиболее вероятные причины сбоя:
  - **Geoff не установился** через `npm install -g @chapeaux/geoff` (возможно, пакет переименован или недоступен).
  - **`geoff build` упал** из-за синтаксической ошибки в TOML-frontmatter или в `geoff.toml`.
  - **Шаблон `blog-page.html` не найден** — Geoff может использовать шаблоны по умолчанию, но если `default_template` указан, а файла нет, сборка может упасть.

**3. Если workflow падает — упростите конфигурацию**

Попробуйте временно убрать из `geoff.toml` строку `default_template = "blog-page.html"` и параметр `[search]`. Если после этого сборка пройдёт успешно, значит, проблема была в отсутствующем шаблоне или в настройках поиска.

**4. Проверьте, что Geoff вообще собирает сайт**

В логах workflow после шага **Build Site** должно быть что-то вроде `Generated 3 pages` или `Build completed`. Если там пусто или ошибка — Geoff не может найти контент. Убедитесь, что файлы в `content/` имеют корректный frontmatter с закрывающими `+++`.

**5. Альтернатива: посмотрите артефакт вручную**

В последнем запуске workflow на вкладке **Actions** внизу страницы есть раздел **Artifacts**. Если там есть артефакт `github-pages`, скачайте его и проверьте, что внутри есть `index.html` и другие страницы. Если артефакт пустой или содержит только `README.md` — значит, `geoff build` не сгенерировал сайт.

### 💎 Итог

Файлы в репозитории корректны. Проблема с отображением README.md почти наверняка связана с тем, что **источник GitHub Pages не переключён на GitHub Actions**. После переключения сайт должен начать отдавать собранный Geoff-ом `dist/`. Если после этого появится ошибка 404 или пустая страница — проверьте логи workflow: скорее всего, `geoff build` падает из-за отсутствующего шаблона или синтаксической ошибки в frontmatter.

Если после переключения источника сайт всё ещё не работает, пришлите скриншот или текст ошибки из последнего запуска workflow — я помогу разобраться.

### 🔗 Ссылки

- GitHub Pages: настройка источника публикации — https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- Geoff (GitHub) — https://github.com/chapeaux/geoff
- @chapeaux/geoff (npm) — https://www.npmjs.com/package/@chapeaux/geoff

## 1a
Проблема в самом workflow — он падает с ошибкой `exit code 1`.

Предупреждение о Node.js 20 — это не причина падения, а лишь уведомление о том, что GitHub постепенно переводит экшены на Node.js 24. Ошибка `exit code 1` означает, что один из шагов workflow завершился с ошибкой. Скорее всего, это шаг **Install Geoff** или **Build Site**.

### 🔍 Наиболее вероятная причина: Geoff — это Rust-инструмент, а не npm-пакет

Geoff — это статический генератор на Rust, который распространяется через **crates.io** под именем `chapeaux-geoff`. Хотя в документации упоминается установка через `npm install -g @chapeaux/geoff`, этот npm-пакет может быть неполным, устаревшим или вовсе отсутствовать в реестре. Если `npm install` не может найти пакет, шаг падает с `exit code 1`.

**Проверить это можно так:** откройте последний запуск workflow на вкладке **Actions**, раскройте шаг **Install Geoff** и посмотрите на текст ошибки. Если там `404 Not Found` или `npm ERR! 404`, значит, пакет не найден.

### 🛠️ Решение: установить Geoff через Cargo (Rust)

Вместо `npm install -g @chapeaux/geoff` нужно использовать `cargo install chapeaux-geoff`. Это потребует настройки Rust в workflow. Вот исправленный `.github/workflows/deploy.yml`:

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
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true   # убирает предупреждение о Node.js 20

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Rust
        uses: dtolnay/rust-toolchain@stable

      - name: Install Geoff
        run: cargo install chapeaux-geoff

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

### 📋 Что изменилось и почему

| Было | Стало | Причина |
|---|---|---|
| `actions/setup-node@v4` | `dtolnay/rust-toolchain@stable` | Geoff написан на Rust, ему нужен Rust-тулчейн, а не Node.js |
| `npm install -g @chapeaux/geoff` | `cargo install chapeaux-geoff` | Установка из crates.io — официальный способ для Rust-инструментов |
| `node-version: '20'` | — (убрано) | Node.js больше не нужен для сборки Geoff |
| — | `env: FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true` | Убирает предупреждение о Node.js 20 |

### ⚠️ Важное замечание о времени сборки

`cargo install chapeaux-geoff` **компилирует Geoff из исходников**, что может занять **5–15 минут** при первом запуске. GitHub Actions имеет лимит в 6 часов на job, так что это не проблема, но первый деплой будет долгим. Последующие запуски могут быть быстрее, если кэшировать `~/.cargo` и `target/`, но для тестового проекта это не обязательно.

### 🔄 Что делать дальше

1. Откройте `.github/workflows/deploy.yml` в репозитории.
2. Замените содержимое на исправленный вариант выше.
3. Нажмите **Commit changes**.
4. Перейдите на вкладку **Actions** и дождитесь завершения нового запуска (он может идти 5–15 минут).
5. Если сборка пройдёт успешно — сайт появится по адресу `https://bpmbpm.github.io/geoff_test/`.

Если после этого workflow снова упадёт, пришлите текст ошибки из шага **Install Geoff** или **Build Site** — я помогу разобраться.

### 🔗 Ссылки

- chapeaux-geoff на crates.io — https://crates.io/crates/chapeaux-geoff
- Geoff на lib.rs — https://lib.rs/crates/geoff-content
- Rust toolchain action — https://github.com/dtolnay/rust-toolchain
- GitHub Actions: Node.js 20 deprecation — https://github.blog/changelog/2025-09-19-deprecation-of-node-20-on-github-actions-runners/


