# geoff_test
geoff_test — семантический Zettelkasten на GitHub Pages  
https://bpmbpm.github.io/geoff_test/

Демонстрационный проект, показывающий, как развернуть семантическую вики на GitHub Pages с использованием [Geoff](https://github.com/chapeaux/geoff) — статического генератора сайтов, построенного на linked data.

## Что это?

Geoff превращает Markdown-файлы с TOML-frontmatter в статический HTML со встроенным JSON-LD, backed by запрашиваемый RDF-граф. Каждая заметка становится набором RDF-триплетов, а в браузере можно выполнять SPARQL-запросы через Oxigraph WASM.

## Как это работает

1. **Markdown + TOML frontmatter** — вы пишете заметки в Markdown, а метаданные описываете в TOML.
2. **Маппинг онтологии** — поля frontmatter автоматически превращаются в RDF-свойства согласно `ontology/mappings.toml`.
3. **Сборка** — GitHub Actions запускает `geoff build`, который генерирует HTML и RDF-граф.
4. **Клиентский SPARQL** — в браузере пользователя работает Oxigraph WASM, который выполняет SPARQL-запросы к графу.

## Структура

- `geoff.toml` — основная конфигурация.
- `ontology/mappings.toml` — маппинг frontmatter → RDF.
- `content/` — Markdown-заметки.
- `.github/workflows/deploy.yml` — сборка и деплой.

## Тестовые данные

В `content/` лежат три заметки: `index.md`, `hello.md` и `bob.md`. Они связаны через `related` и содержат метаданные `author`, `date`, `tags`.

## Как воспроизвести

1. Форкните или клонируйте этот репозиторий.
2. Включите GitHub Pages: **Settings → Pages → Source: GitHub Actions**.
3. Сделайте пуш в `main` — Actions соберёт и опубликует сайт.
4. Откройте `https://bpmbpm.github.io/geoff_test/`.

## Ссылки

- [Geoff (GitHub)](https://github.com/chapeaux/geoff)
- [chapeaux-geoff (crates.io)](https://crates.io/crates/chapeaux-geoff)
- [@chapeaux/geoff (npm)](https://www.npmjs.com/package/@chapeaux/geoff)
- [Oxigraph (RDF store с WASM)](https://github.com/oxigraph/oxigraph)
- [SPARQL 1.1 Query Language (W3C)](https://www.w3.org/TR/sparql11-query/)
