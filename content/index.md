+++
title = "Главная"
type = "Note"
template = "blog-page.html"
+++

# Мой семантический Zettelkasten

Это тестовая семантическая вики, собранная с помощью Geoff.

## Все заметки

<div id="sparql-results">Загрузка...</div>

<script type="module">
  import { init } from '@chapeaux/geoff-client';

  const engine = await init();

  const results = await engine.query(`
    PREFIX schema: <http://schema.org/>
    SELECT ?title ?author WHERE {
      ?note a schema:CreativeWork ;
            schema:name ?title ;
            schema:author ?author .
    }
    ORDER BY ?title
  `);

  document.getElementById('sparql-results').innerHTML = results
    .map(r => `<div><strong>${r.title.value}</strong> — ${r.author.value}</div>`)
    .join('');
</script>
