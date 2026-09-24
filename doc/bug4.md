## 1
Ошибка `Template 'blog-page.html' not found` означает, что в вашем проекте физически отсутствует файл шаблона с этим именем. Geoff — это статический генератор, который использует шаблонизатор **Tera** для превращения Markdown-файлов в HTML. Если в фронтматтере указан `template = "blog-page.html"`, Geoff ищет этот файл в папке шаблонов, но не находит его и останавливает сборку.

Поскольку вы создавали проект вручную через веб-интерфейс GitHub, а не командой `geoff init my-site --template blog`, папка с шаблонами просто не была сгенерирована. Самый простой путь — **создать минимальный шаблон вручную** и указать его имя в фронтматтере.

### 🛠️ Решение: создать файл `templates/page.html`

Создайте в корне репозитория папку `templates`, а внутри — файл `page.html` со следующим содержимым:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }} — Мой семантический Zettelkasten</title>
</head>
<body>
    <main>
        <h1>{{ title }}</h1>
        {{ content | safe }}
    </main>
</body>
</html>
```

Это минимальный валидный шаблон на Tera. Переменные `{{ title }}` и `{{ content | safe }}` автоматически подставляются Geoff: заголовок из фронтматтера и отрендеренный Markdown соответственно. Фильтр `| safe` отключает экранирование HTML, чтобы теги отображались корректно.

### 📝 Указать этот шаблон в фронтматтере

Теперь во **всех** Markdown-файлах в папке `content/` замените строку `template = "blog-page.html"` на `template = "page.html"`:

**`content/index.md`**
```toml
+++
title = "Главная"
type = "Note"
template = "page.html"
+++
```

**`content/hello.md`**
```toml
+++
title = "Hello Semantic World"
type = "Note"
template = "page.html"
date = 2026-04-10
author = "Alice"
tags = ["semantic", "hello-world"]
+++
```

**`content/bob.md`**
```toml
+++
title = "Заметка Боба"
type = "Note"
template = "page.html"
date = 2026-04-11
author = "Bob"
tags = ["sparql", "rdf"]
+++
```

### 📂 Итоговая структура проекта

После этих изменений репозиторий должен выглядеть так:

```
geoff_test/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── content/
│   ├── index.md
│   ├── hello.md
│   └── bob.md
├── templates/
│   └── page.html          ← новый файл
├── geoff.toml
├── ontology/
│   └── mappings.toml
└── README.md
```

Geoff автоматически ищет шаблоны в папке `templates/` в корне проекта. После создания `page.html` и замены ссылок в фронтматтере сборка должна пройти успешно.

### ⚠️ Если ошибка повторится с другим именем файла

Это будет означать, что вы пропустили какой-то Markdown-файл. Проверьте все `.md` в `content/` — в каждом из них должно быть `template = "page.html"`. Если добавите новые заметки в будущем, не забывайте указывать этот шаблон.

### 💎 Альтернатива: скопировать оригинальные шаблоны Geoff

Если вы хотите получить полноценный шаблон блога (с навигацией, стилями и т.д.), можно найти исходные шаблоны в репозитории Geoff на GitHub — https://github.com/chapeaux/geoff. Однако для «Hello world» достаточно минимального `page.html`, созданного вручную.

### 🔗 Ссылки

- Geoff (репозиторий) — https://github.com/chapeaux/geoff
- Geoff (документация) — https://lib.rs/crates/geoff-server
- Tera (шаблонизатор, используемый Geoff) — https://keats.github.io/tera/

## 2

Вы абсолютно правы — это более простое решение. Если создать `blog-page.html`, то **не нужно менять фронтматтер ни в одном файле**: все заметки уже ссылаются на `blog-page.html`, и Geoff просто найдёт этот шаблон. Это меньше правок и меньше шансов что-то пропустить.

Я предложил `page.html` только потому, что это имя используется Geoff как шаблон по умолчанию (когда `template` вообще не указан). Но раз в ваших файлах уже явно прописан `blog-page.html`, логичнее просто создать именно этот файл.

### 🛠️ Решение: создать файл `templates/blog-page.html`

Создайте в корне репозитория папку `templates`, а внутри — файл `blog-page.html`:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }} — Мой семантический Zettelkasten</title>
</head>
<body>
    <main>
        <h1>{{ title }}</h1>
        {{ content | safe }}
    </main>
</body>
</html>
```

После этого **фронтматтер менять не нужно**. Файлы `content/index.md`, `content/hello.md`, `content/bob.md` остаются как есть — с `template = "blog-page.html"`.

### 📂 Итоговая структура

```
geoff_test/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── content/
│   ├── index.md
│   ├── hello.md
│   └── bob.md
├── templates/
│   └── blog-page.html     ← новый файл
├── geoff.toml
├── ontology/
│   └── mappings.toml
└── README.md
```

### 💡 Почему это лучше

| Подход | Правок | Риск ошибки |
|---|---|---|
| Создать `blog-page.html` | 1 файл | Минимальный |
| Создать `page.html` + менять фронтматтер | 4 файла | Можно пропустить заметку |

Если в будущем вы захотите разные шаблоны для разных типов страниц (например, `note.html` для заметок и `index.html` для главной), вы просто создадите дополнительные файлы в `templates/` и укажете их в фронтматтере. Но для «Hello world» одного `blog-page.html` достаточно.

### 🔗 Ссылки

- Geoff (репозиторий) — https://github.com/chapeaux/geoff
- Tera (шаблонизатор, используемый Geoff) — https://keats.github.io/tera/
  
## 2

## 🔍 Почему висит «Загрузка...»

Geoff декларирует функцию «Client-side SPARQL search — Oxigraph WASM in the browser, querying the same graph that built the site». Однако эта функция **не включается автоматически в произвольный шаблон**. В оригинальных шаблонах Geoff (которые создаются командой `geoff init --template blog`) есть специальный JavaScript-код, который:

1. Загружает библиотеку **Oxigraph WASM**.
2. Загружает файл с RDF-графом (вероятно, `graph.nq` или `data.jsonld` в папке `dist`).
3. Выполняет SPARQL-запросы из блоков `sparql` в Tera-шаблонах.

Ваш минимальный `blog-page.html` содержит только `{{ content | safe }}` — то есть просто вставляет HTML, сгенерированный из Markdown. **Скрипт для клиентского SPARQL в нём отсутствует**. Поэтому браузер видит `<div id="sparql-results">Загрузка...</div>`, но никакой JavaScript его не обновляет.

## 🛠️ Что можно сделать

### Вариант 1. Использовать оригинальный шаблон Geoff

Самый надёжный путь — заменить ваш `blog-page.html` на оригинальный шаблон из стартового набора Geoff. Для этого:

1. Локально (или в GitHub Codespaces) выполните:
   ```bash
   geoff init my-site --template blog
   ```
2. Скопируйте файл `templates/blog-page.html` из сгенерированного проекта.
3. Замените им ваш текущий `blog-page.html` в репозитории `bpmbpm/geoff_test`.

Оригинальный шаблон уже содержит все необходимые подключения к Oxigraph WASM и обрабатывает блоки `sparql` в Tera.

### Вариант 2. Добавить клиентский скрипт вручную

Если вы хотите оставить минимальный шаблон, нужно добавить в `blog-page.html` скрипт, который загрузит Oxigraph и выполнит запрос. Примерная структура:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
    <main>
        <h1>{{ title }}</h1>
        {{ content | safe }}
    </main>

    <!-- Клиентский SPARQL через Oxigraph WASM -->
    <script type="module">
        import init, * as oxigraph from 'https://cdn.jsdelivr.net/npm/oxigraph@latest/web.js';

        await init();
        const store = new oxigraph.Store();

        // Загружаем RDF-граф, сгенерированный Geoff
        const response = await fetch('/geoff_test/graph.nq'); // путь может отличаться
        const data = await response.text();
        store.load(data, { format: 'application/n-quads' });

        // Выполняем запрос и вставляем результат
        const results = store.query(`
            PREFIX schema: <http://schema.org/>
            SELECT ?title ?author WHERE {
                ?note a schema:CreativeWork ;
                      schema:name ?title ;
                      schema:author ?author .
            }
        `);

        // ... отображение результатов
    </script>
</body>
</html>
```

Однако **точный путь к файлу графа** и **формат** (N-Quads, Turtle, JSON-LD) нужно уточнить. Geoff может генерировать граф в папку `dist` под именем, которое зависит от конфигурации. Проверьте содержимое артефакта сборки: в GitHub Actions, в последнем запуске, скачайте артефакт `github-pages` и посмотрите, какие файлы лежат внутри `dist/`. Ищите файлы с расширениями `.nq`, `.ttl`, `.jsonld` или `.trig`.

### Вариант 3. Использовать серверный SPARQL в шаблонах Tera

Geoff поддерживает **SPARQL в Tera-шаблонах** через функцию `sparql()`. Это значит, что вы можете выполнять запросы **во время сборки** и вставлять результаты прямо в HTML. Пример:

```html
{% set posts = sparql(query="
    PREFIX schema: <http://schema.org/>
    SELECT ?title ?author WHERE {
        ?note a schema:CreativeWork ;
              schema:name ?title ;
              schema:author ?author .
    }
") %}
<ul>
{% for post in posts %}
    <li>{{ post.title }} — {{ post.author }}</li>
{% endfor %}
</ul>
```

Этот вариант **не требует Oxigraph WASM** и работает полностью на этапе сборки. Если вам достаточно статического списка заметок (без интерактивных запросов), этот подход проще всего.

## 💎 Рекомендация

Для быстрого результата проще всего **Вариант 3** — использовать серверный SPARQL в Tera-шаблоне. Он не требует отладки клиентского JavaScript и сразу даст работающий список заметок на главной странице.

Если вам нужен именно **интерактивный клиентский SPARQL** (пользователь вводит запрос в браузере), то нужно либо взять оригинальный шаблон Geoff (Вариант 1), либо вручную добавить Oxigraph WASM (Вариант 2), но для этого сначала нужно узнать, где Geoff хранит сгенерированный RDF-граф.

### 🔗 Ссылки

- Geoff (lib.rs): описание клиентского SPARQL — https://lib.rs/crates/geoff-reading-time
- Oxigraph для JavaScript (CDN) — https://cdn.jsdelivr.net/npm/oxigraph
- Tera (шаблонизатор Geoff) — https://keats.github.io/tera/
- SPARQL 1.1 (W3C) — https://www.w3.org/TR/sparql11-query/

## 3  

Чтобы вручную добавить интерактивный клиентский SPARQL, нужно понимать, как Geoff экспортирует данные. Ключевой момент: **Geoff не создаёт готовый `.rete` файл**, как Rete, а экспортирует RDF-граф в формате **N-Triples** (`.nt`) во время сборки. Этот файл и нужно загружать в Oxigraph WASM.

Ниже — подробная инструкция и код для тестирования.

### 📍 Шаг 1. Включите экспорт RDF-графа в `geoff.toml`

Откройте `geoff.toml` и добавьте секцию `[search]`:

```toml
[search]
enabled = true
```

Согласно документации Geoff, при `enabled = true` во время сборки **экспортируется RDF-граф как N-Triples**. Без этого шага файла с графом в `dist/` не будет.

### 📂 Шаг 2. Найдите файл N-Triples после сборки

Сделайте коммит и дождитесь завершения GitHub Actions. Затем:

1. Откройте вкладку **Actions** в репозитории.
2. Выберите последний успешный запуск.
3. Внизу страницы, в разделе **Artifacts**, скачайте артефакт `github-pages`.
4. Распакуйте архив и посмотрите содержимое папки `dist/`.

Ищите файл с расширением **`.nt`** (N-Triples). Скорее всего, он называется `graph.nt` или `data.nt`. Запомните точное имя и путь — он понадобится для JavaScript.

### 🧪 Шаг 3. Создайте тестовую HTML-страницу с Oxigraph WASM

Создайте в корне репозитория файл `test-sparql.html` (рядом с `index.html` или в папке `static/`, если она есть). Этот файл будет загружать N-Triples и выполнять SPARQL-запросы прямо в браузере.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Тест клиентского SPARQL (Geoff + Oxigraph)</title>
    <style>
        body { font-family: system-ui, sans-serif; max-width: 900px; margin: 2rem auto; padding: 0 1rem; }
        textarea { width: 100%; font-family: monospace; font-size: 0.9rem; }
        button { padding: 0.5rem 1rem; cursor: pointer; margin-top: 0.5rem; }
        pre { background: #f4f4f4; padding: 1rem; overflow-x: auto; }
        #status { color: #888; font-style: italic; }
        #error { color: #c0392b; }
    </style>
</head>
<body>
    <h1>🧪 Тест клиентского SPARQL</h1>
    <p id="status">Загрузка RDF-графа…</p>
    <div id="error"></div>

    <h2>SPARQL-консоль</h2>
    <p>Введите запрос к графу, сгенерированному Geoff:</p>
    <textarea id="query" rows="10">PREFIX schema: <http://schema.org/>
SELECT ?title ?author WHERE {
  ?note a schema:CreativeWork ;
        schema:name ?title ;
        schema:author ?author .
}
ORDER BY ?title</textarea>
    <br>
    <button onclick="runQuery()">Выполнить</button>
    <pre id="results"></pre>

    <script type="module">
        // 1. Импортируем Oxigraph WASM через CDN
        import init, * as oxigraph from 'https://cdn.jsdelivr.net/npm/oxigraph@latest/web.js';

        const statusEl = document.getElementById('status');
        const errorEl = document.getElementById('error');
        const resultsEl = document.getElementById('results');

        // 2. Инициализируем Oxigraph WASM
        await init();
        const store = new oxigraph.Store();

        // 3. Загружаем N-Triples файл, сгенерированный Geoff
        //    ВАЖНО: проверьте точное имя файла в папке dist/ после сборки!
        //    Возможные варианты: graph.nt, data.nt, geoff.nt
        const ntFileUrl = './graph.nt';  // ← замените на реальное имя файла

        try {
            const response = await fetch(ntFileUrl);
            if (!response.ok) {
                throw new Error(`Файл ${ntFileUrl} не найден (HTTP ${response.status}). Проверьте имя файла в dist/.`);
            }
            const ntData = await response.text();
            store.load(ntData, { format: 'application/n-triples' });
            statusEl.textContent = `✅ Граф загружен. Триплетов: ${store.size}`;
        } catch (err) {
            statusEl.textContent = '❌ Ошибка загрузки графа';
            errorEl.textContent = err.message;
        }

        // 4. Функция для выполнения SPARQL-запросов
        window.runQuery = async () => {
            const query = document.getElementById('query').value;
            try {
                const results = store.query(query);
                const rows = [];
                for (const binding of results) {
                    const row = {};
                    for (const [key, value] of binding) {
                        row[key] = value.value;
                    }
                    rows.push(row);
                }
                resultsEl.textContent = JSON.stringify(rows, null, 2);
            } catch (err) {
                resultsEl.textContent = `Ошибка: ${err.message}`;
            }
        };
    </script>
</body>
</html>
```

### 🔍 Шаг 4. Проверьте формат и имя файла

Ключевая сложность: **имя файла N-Triples может отличаться** в разных версиях Geoff. В коде выше указан `./graph.nt` как предположение. Если файл не найдётся:

1. Скачайте артефакт сборки (`github-pages`).
2. Посмотрите, какой именно `.nt` файл создан в `dist/`.
3. Замените `const ntFileUrl = './graph.nt';` на реальное имя.

Также проверьте, что формат — именно **N-Triples**. Если Geoff экспортирует в Turtle (`.ttl`), замените формат в `store.load()` на `'text/turtle'`.

### 💡 Альтернатива: использовать готовый веб-компонент `<geoff-search>`

Если вы не хотите писать JavaScript вручную, Geoff предоставляет **готовый веб-компонент** для клиентского поиска. Для его использования:

1. Убедитесь, что в `geoff.toml` включено `[search] enabled = true`.
2. В шаблоне страницы (`templates/blog-page.html`) добавьте:

```html
<script type="module" src="/geoff-search.js"></script>
<geoff-search></geoff-search>
```

Согласно документации Geoff, этот компонент **лениво загружает Oxigraph WASM** и выполняет настоящие SPARQL-запросы к тому же графу, что построил сайт. Он поддерживает структурированные запросы вида `geoff:stage=develop` наряду с обычным текстовым поиском.

### ⚠️ Важное замечание

Если в вашем проекте `geoff-search.js` не создаётся (например, из-за минимальной конфигурации), ручной вариант с Oxigraph — единственный путь. Но тогда обязательно проверьте, что Geoff действительно экспортирует `.nt` файл при `[search] enabled = true`. Если файла нет, возможно, потребуется добавить в `geoff.toml` дополнительные настройки, которые не описаны в публичной документации.

### 🔗 Ссылки

- Oxigraph для JavaScript (npm) — https://www.npmjs.com/package/oxigraph
- Oxigraph WASM на jsDelivr — https://cdn.jsdelivr.net/npm/oxigraph@latest/web.js
- Geoff: раздел Client-Side Search — https://lib.rs/crates/geoff-reading-time
- N-Triples (W3C) — https://www.w3.org/TR/n-triples/

## 3a

Вы правы, в артефакте `github-pages` папки `dist` не будет. И это нормально, потому что `actions/upload-pages-artifact` **копирует содержимое указанной папки в корень артефакта**. То есть все файлы, которые Geoff сгенерировал в `dist/`, оказываются в корне архива. Именно поэтому на вашем сайте отображается контент — артефакт содержит готовые HTML-файлы, а не папку `dist`.

### 🔍 Что же тогда делать с RDF-графом?

Geoff не создаёт отдельный файл `.nt` или `.ttl` в папке `dist/`. Вместо этого он **встраивает JSON-LD прямо в HTML-страницы**. Это указано в документации: «Produces static HTML in `dist/` with embedded JSON-LD». То есть RDF-граф существует внутри `<script type="application/ld+json">` на каждой странице.

Если вы хотите выполнять **интерактивные клиентские SPARQL-запросы**, вам нужно:

1. **Извлечь JSON-LD** из HTML-страниц.
2. **Преобразовать его в RDF-квады** (например, с помощью библиотеки `jsonld`).
3. **Загрузить квады в Oxigraph WASM** и выполнять SPARQL.

Geoff заявляет поддержку клиентского SPARQL через Oxigraph WASM, но, судя по всему, эта функция доступна только в **оригинальных шаблонах**, которые создаются командой `geoff init --template blog`. В вашем минимальном шаблоне `blog-page.html` этого кода нет.

### 🛠️ Как проверить, что JSON-LD есть в HTML

Скачайте артефакт `github-pages` (это можно сделать на вкладке **Actions** в последнем запуске, внизу страницы). Распакуйте архив — вы увидите `index.html`, `hello.html`, `bob.html` и другие файлы. Откройте любой из них в текстовом редакторе и поищите строку `application/ld+json`. Если она есть — значит, RDF-данные встроены в страницу.

### 📋 Что можно сделать, чтобы получить клиентский SPARQL

**Вариант 1. Использовать оригинальный шаблон Geoff**

Самый надёжный путь — заменить ваш `blog-page.html` на оригинальный шаблон из стартового набора. Для этого:

1. Локально (или в GitHub Codespaces) выполните `geoff init my-site --template blog`.
2. Скопируйте файл `templates/blog-page.html` из сгенерированного проекта.
3. Замените им ваш текущий `blog-page.html` в репозитории.

Оригинальный шаблон уже содержит все необходимые подключения к Oxigraph WASM и обрабатывает блоки `sparql` в Tera.

**Вариант 2. Написать собственный JavaScript для извлечения JSON-LD и загрузки в Oxigraph**

Если вы хотите оставить минимальный шаблон, можно добавить скрипт, который:

1. Собирает все JSON-LD блоки со страницы.
2. Преобразует их в RDF-квады.
3. Загружает в Oxigraph и выполняет SPARQL.

Пример кода (требует библиотек `jsonld` и `oxigraph`):

```html
<script type="module">
  import jsonld from 'https://cdn.jsdelivr.net/npm/jsonld/+esm';
  import init, * as oxigraph from 'https://cdn.jsdelivr.net/npm/oxigraph@latest/web.js';

  await init();
  const store = new oxigraph.Store();

  // 1. Собираем все JSON-LD со страницы
  const scripts = document.querySelectorAll('script[type="application/ld+json"]');
  const jsonldData = [];
  for (const script of scripts) {
    try {
      jsonldData.push(JSON.parse(script.textContent));
    } catch (e) { console.warn('Ошибка парсинга JSON-LD', e); }
  }

  // 2. Преобразуем JSON-LD в N-Quads
  const nquads = await jsonld.toRDF(jsonldData, { format: 'application/n-quads' });

  // 3. Загружаем в Oxigraph
  store.load(nquads, { format: 'application/n-quads' });

  // 4. Выполняем SPARQL
  const results = store.query(`
    PREFIX schema: <http://schema.org/>
    SELECT ?title ?author WHERE {
      ?note a schema:CreativeWork ;
            schema:name ?title ;
            schema:author ?author .
    }
  `);

  console.log([...results]);
</script>
```

Этот подход работает, но требует, чтобы JSON-LD на странице был полным (содержал все нужные триплеты). Geoff генерирует JSON-LD для каждой страницы отдельно, поэтому на одной странице вы получите только триплеты этой страницы, а не всего графа.

### 💎 Итог

Папки `dist` в артефакте нет, потому что `upload-pages-artifact` копирует её содержимое в корень. RDF-граф в Geoff **не сериализуется в отдельный файл**, а встраивается в HTML как JSON-LD. Для клиентского SPARQL вам либо нужно взять оригинальный шаблон Geoff (Вариант 1), либо написать собственный скрипт, который извлекает JSON-LD и загружает его в Oxigraph (Вариант 2).

Если вы хотите **интерактивный SPARQL без всех этих сложностей**, рекомендую вернуться к **Rete** — он делает именно то, что вам нужно: упаковывает RDF-граф в один файл и позволяет выполнять SPARQL в браузере без возни с JSON-LD и шаблонами.

### 🔗 Ссылки

- Geoff: описание JSON-LD и сборки в dist/ — https://lib.rs/crates/geoff-server
- Oxigraph для JavaScript (CDN) — https://cdn.jsdelivr.net/npm/oxigraph
- Библиотека jsonld (CDN) — https://cdn.jsdelivr.net/npm/jsonld
