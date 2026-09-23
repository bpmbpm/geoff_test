## 1

Вы правы: ошибка `404 Not Found` при обращении к `https://registry.npmjs.org/@chapeaux%2fgeoff` означает, что **npm-пакета с таким именем не существует в реестре**. Это не сбой сети и не проблема прав доступа — пакет просто не опубликован. Установка через `npm install -g @chapeaux/geoff` не сработает, сколько бы раз вы её ни повторяли.

### Почему так произошло

В документации Geoff указаны два способа установки: `cargo install chapeaux-geoff` и `npm install -g @chapeaux/geoff`. Первый способ корректен — пакет `chapeaux-geoff` действительно опубликован на crates.io. Второй способ, судя по всему, является ошибкой в документации: либо пакет планировался к публикации, но так и не был опубликован, либо имя указано неверно. Поиск по npm-реестру не находит ни `@chapeaux/geoff`, ни `chapeaux/geoff`.

### Что делать

**Единственный рабочий способ установки Geoff — через Rust/Cargo.** Возвращаемся к этому варианту, но с учётом предыдущих ошибок. Ранее `cargo install chapeaux-geoff` падал с `exit code 101` — это могло быть связано с отсутствием nightly-тулчейна или системных зависимостей. Попробуем более надёжную конфигурацию.

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

      - name: Setup Rust
        uses: dtolnay/rust-toolchain@nightly

      - name: Install Geoff
        run: cargo install chapeaux-geoff --locked

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

### Что изменилось и почему

| Элемент | Значение | Пояснение |
|---|---|---|
| `dtolnay/rust-toolchain@nightly` | nightly Rust | В одном из результатов поиска упоминается `rustc 1.96.0-nightly` — Geoff может требовать nightly-функции |
| `cargo install chapeaux-geoff --locked` | флаг `--locked` | Использует зафиксированные версии зависимостей из `Cargo.lock`, что снижает риск конфликтов |
| `geoff --version` | проверка установки | Покажет в логах, что CLI доступен. Если шаг упадёт — сразу увидите, что установка не удалась |
| `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24` | `true` | Убирает предупреждение о Node.js 20, которое не влияет на сборку, но засоряет лог |

### Важное предупреждение о времени сборки

`cargo install` **компилирует Geoff из исходников**. Это может занять **10–20 минут** при первом запуске. GitHub Actions имеет лимит в 6 часов на job, так что это не проблема, но будьте готовы к долгому ожиданию. Если компиляция снова упадёт с `exit code 101`, проверьте в логах сообщение об ошибке — скорее всего, это будет отсутствие какой-либо системной библиотеки (например, `librdf` или `libssl`). В таком случае потребуется добавить шаг `sudo apt-get install -y <пакет>` перед `cargo install`.

### Если и это не сработает

Если `cargo install chapeaux-geoff` снова упадёт, я рекомендую **вернуться к Rete** или **Oxigraph WASM**. Эти инструменты решают ту же задачу (клиентский SPARQL на GitHub Pages), но не требуют компиляции Rust:

- **Rete** — работает через WASM в браузере, устанавливается как npm-пакет `rete-graph`, имеет готовый пример.
- **Oxigraph WASM** — подключается через CDN, загружает `.ttl` файл и выполняет SPARQL-запросы в браузере. Никакой сборки, никакого CLI.

Если вы хотите, я могу подготовить полную инструкцию для одного из этих вариантов — без Rust, без Geoff и без ошибок 404.

### Ссылки

- chapeaux-geoff на crates.io — https://crates.io/crates/chapeaux-geoff
- Geoff на lib.rs (с ошибочной npm-инструкцией) — https://lib.rs/crates/geoff-reading-time
- Rust toolchain action — https://github.com/dtolnay/rust-toolchain
- Rete (GitHub) — https://github.com/caviri/rete
- Oxigraph (npm) — https://www.npmjs.com/package/oxigraph
