Нет, npm-пакет `@chapeaux/geoff` **нельзя** напрямую подключить через CDN как клиентскую библиотеку. Это **CLI-инструмент для сборки**, а не браузерная библиотека. Однако это не значит, что клиентский SPARQL невозможен — просто он работает не так, как вы предположили.

### Почему `@chapeaux/geoff` не подходит для CDN

`@chapeaux/geoff` — это пакет, который при установке предоставляет **команду `geoff`** для терминала. Внутри он содержит Rust-бинарник (скомпилированный или обёрнутый), который запускается в Node.js-среде и **генерирует статические файлы**. Браузер не может выполнить этот бинарник, потому что:

- Браузер не имеет доступа к файловой системе.
- Браузер не может запустить Rust-код напрямую.
- Пакет не экспортирует браузерную JS-библиотеку с функциями типа `query()`.

### Как Geoff на самом деле делает клиентский SPARQL

В документации Geoff указано: **«Client-side SPARQL search — Oxigraph WASM in the browser, querying the same graph that built the site»**. Это означает, что **во время сборки** Geoff:

1. Строит RDF-граф из ваших Markdown-файлов.
2. Сериализует этот граф в файл (например, `.ttl` или `.nq`).
3. Генерирует HTML-страницы, которые **включают JavaScript-код**, загружающий **Oxigraph WASM** и выполняющий запросы к этому файлу.

То есть клиентский SPARQL — это не сам Geoff, а **Oxigraph WASM**, который Geoff подключает в сгенерированный HTML. Oxigraph — это отдельная библиотека, которая **действительно доступна через CDN** и может работать в браузере.

### Как это сделать вручную (без Geoff)

Если вы хотите клиентский SPARQL без CLI, вы можете:

1. **Сгенерировать RDF-граф** любым способом (даже вручную написать `.ttl` файл).
2. **Подключить Oxigraph WASM через CDN** в вашем HTML.
3. **Загрузить `.ttl` файл** в Oxigraph и выполнять SPARQL-запросы.

Пример HTML-страницы, которая делает это **без Geoff**:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>SPARQL на клиенте через Oxigraph WASM</title>
</head>
<body>
  <h1>Мои заметки</h1>
  <div id="results">Загрузка...</div>

  <script type="module">
    // Подключаем Oxigraph WASM через CDN
    import oxigraph from 'https://cdn.jsdelivr.net/npm/oxigraph@latest/web.js';

    // Инициализируем store
    const store = new oxigraph.Store();

    // Загружаем RDF-граф (ваш .ttl файл)
    const response = await fetch('./graph.ttl');
    const ttl = await response.text();
    store.load(ttl, { format: 'text/turtle' });

    // Выполняем SPARQL-запрос
    const results = store.query(`
      PREFIX schema: <http://schema.org/>
      SELECT ?title ?author WHERE {
        ?note a schema:CreativeWork ;
              schema:name ?title ;
              schema:author ?author .
      }
    `);

    // Отображаем результаты
    document.getElementById('results').innerHTML = [...results]
      .map(r => `<div><strong>${r.get('title').value}</strong> — ${r.get('author').value}</div>`)
      .join('');
  </script>
</body>
</html>
```

**Что здесь происходит:**
- `oxigraph` загружается через jsDelivr как ES-модуль (это **браузерная сборка**, а не Node.js-пакет).
- `store.load()` загружает ваш `.ttl` файл в память браузера.
- `store.query()` выполняет SPARQL **полностью на клиенте**, без сервера.

### Итог: два разных подхода

| Подход | Что делает | Где выполняется | Нужен ли CLI |
|---|---|---|---|
| **Geoff** | Собирает сайт из Markdown, генерирует HTML + RDF-граф | GitHub Actions (Node.js/Rust) | Да, `geoff build` |
| **Oxigraph WASM + ваш HTML** | Выполняет SPARQL к готовому `.ttl` файлу | Браузер пользователя | Нет, только HTML + CDN |

Если ваша цель — **вообще избежать CLI** и при этом получить клиентский SPARQL, вам не нужен Geoff. Достаточно:

1. Написать `.ttl` файл с RDF-данными (вручную или любым другим способом).
2. Создать HTML-страницу, которая подключает Oxigraph WASM через CDN и выполняет запросы.
3. Положить оба файла на GitHub Pages.

Это будет работать **без GitHub Actions**, без Node.js, без Rust — только статические файлы.

### Ссылки

- Oxigraph (JavaScript bindings, npm) — https://www.npmjs.com/package/oxigraph
- Oxigraph (GitHub) — https://github.com/oxigraph/oxigraph
- Oxigraph WASM на jsDelivr — https://cdn.jsdelivr.net/npm/oxigraph
- Geoff (описание клиентского SPARQL) — https://lib.rs/crates/geoff-reading-time
