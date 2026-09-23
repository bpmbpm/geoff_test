Ниже — полные тексты файлов для замены и инструкция, как остановить автоматический запуск Actions.

## 📄 Файлы для замены

### `content/index.md`

```markdown
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
```

### `content/hello.md`

```markdown
+++
title = "Hello Semantic World"
type = "Note"
template = "blog-page.html"
date = 2026-04-10
author = "Alice"
tags = ["semantic", "hello-world"]
related = "content/index.md"
+++

# Hello Semantic World

Это моя первая семантическая заметка.

Она автоматически превращается в RDF-триплеты:

- `title` → `schema:name`
- `author` → `schema:author`
- `date` → `schema:datePublished`
- `tags` → `schema:keywords`

Все эти данные доступны для SPARQL-запросов в браузере.
```

### `content/bob.md`

```markdown
+++
title = "Заметка Боба"
type = "Note"
template = "blog-page.html"
date = 2026-04-11
author = "Bob"
tags = ["sparql", "rdf"]
related = "content/hello.md"
+++

# Заметка Боба

Боб тоже пишет заметки. Его заметка связана с заметкой Алисы через `related`.
```

Ключевое изменение — строка `template = "blog-page.html"` в каждом файле. Если в вашем проекте папка с шаблонами называется иначе или файл `blog-page.html` отсутствует, используйте имя того шаблона, который реально существует в репозитории. Проверить это можно, открыв папку с шаблонами в репозитории `bpmbpm/geoff_test`.

## 🛑 Как остановить выполнение Action

Есть несколько способов — от временной приостановки до полного удаления триггера.

### Способ 1. Временное отключение workflow (рекомендуемый)

1. Откройте вкладку **Actions** в репозитории.
2. В левом меню выберите workflow **Deploy Geoff Semantic Wiki**.
3. Нажмите кнопку **⋯** (три точки) справа вверху.
4. Выберите **Disable workflow**.

Workflow перестанет запускаться при пушах, но останется в репозитории. Чтобы снова включить — нажмите **Enable workflow**.

### Способ 2. Ручной запуск (убрать автоматический триггер)

Откройте `.github/workflows/deploy.yml` и замените блок `on:`:

```yaml
on:
  workflow_dispatch:
```

Вместо:

```yaml
on:
  push:
    branches: ["main"]
  workflow_dispatch:
```

Теперь workflow **не будет запускаться автоматически** при пушах. Он будет запускаться только вручную: **Actions → Deploy Geoff Semantic Wiki → Run workflow**.

### Способ 3. Временно отключить через сообщение коммита

Если вы хотите пропустить запуск для одного конкретного коммита, добавьте в сообщение коммита строку:

```
[skip ci]
```

или

```
[no ci]
```

GitHub Actions увидит это и не запустит workflow для данного пуша. Это работает, если в workflow нет дополнительных условий. Подробнее — в документации GitHub по skip ci.

### Способ 4. Удалить файл workflow

Самый радикальный способ: удалить `.github/workflows/deploy.yml` из репозитория. GitHub Actions больше не будет запускаться, но и сайт не будет обновляться. Если вы захотите вернуть автоматизацию — просто создайте файл заново.

### Что делать, если запуск уже идёт

Если workflow уже выполняется и вы хотите его остановить:

1. Откройте вкладку **Actions**.
2. Выберите текущий запуск.
3. Нажмите кнопку **Cancel workflow** справа вверху.

Это прервёт выполнение, но не отменит уже задеплоенные изменения, если шаг деплоя успел завершиться.

## 💎 Рекомендация

Для вашего сценария (эксперименты с Geoff, частая правка файлов) я рекомендую **Способ 2**: оставить только `workflow_dispatch`. Тогда вы сможете вносить изменения в файлы сколько угодно, а сборку запускать вручную, когда захотите. Это избавит от лишних запусков и сэкономит минуты GitHub Actions.

## 🔗 Ссылки

- GitHub Actions: отключение workflow — https://docs.github.com/en/actions/managing-workflow-runs/disabling-and-enabling-a-workflow
- GitHub Actions: пропуск запусков через skip ci — https://docs.github.com/en/actions/managing-workflow-runs/skipping-workflow-runs
- GitHub Actions: отмена запуска — https://docs.github.com/en/actions/managing-workflow-runs/canceling-a-workflow

## 1.1

Да, `pages build and deployment` — это **отдельный автоматический workflow**, который GitHub создаёт сам, когда в настройках Pages выбран источник **Deploy from a branch**. Он не имеет отношения к вашему `deploy.yml` и запускается независимо.

## 🔍 Что это за workflow

Когда вы включаете GitHub Pages с источником **Deploy from a branch** (например, `main / (root)`), GitHub автоматически создаёт внутренний workflow `pages-build-deployment`. Он:

1. Запускается при **каждом пуше** в выбранную ветку.
2. Берёт содержимое указанной папки (корень репозитория или `/docs`).
3. Прогоняет его через Jekyll (если в репозитории нет файла `.nojekyll`).
4. Публикует результат на `https://<логин>.github.io/<репозиторий>/`.

Именно поэтому вы видите `README.md` на своём сайте: этот workflow публикует **содержимое ветки**, а не артефакт, который собирает ваш `deploy.yml`.

## ⚠️ Проблема: два workflow конфликтуют

У вас сейчас **два параллельных механизма публикации**:

| Workflow | Источник | Что публикует |
|---|---|---|
| `Deploy Geoff Semantic Wiki` (ваш) | GitHub Actions | Артефакт `dist/` от Geoff |
| `pages build and deployment` (автоматический) | Ветка `main` | Содержимое репозитория (README.md и т.д.) |

Они борются за один и тот же URL. Побеждает тот, кто запустился последним. Пока ваш `deploy.yml` падал с ошибкой, автоматический workflow публиковал ветку — поэтому вы видели README.md.

## 🛠️ Как убрать `pages build and deployment`

**Способ 1. Переключить источник Pages на GitHub Actions (рекомендуемый)**

Это самый правильный путь, если вы хотите публиковать именно артефакт Geoff:

1. Откройте **Settings → Pages**.
2. В разделе **Build and deployment → Source** выберите **GitHub Actions**.
3. Автоматический workflow `pages-build-deployment` **исчезнет** (или перестанет запускаться).
4. Публикацией будет заниматься только ваш `deploy.yml`.

Кнопки Save в этом разделе нет — выбор применяется сразу. Если источник уже стоит на GitHub Actions, но `pages build and deployment` всё ещё появляется в списке, это остатки старых запусков; новые запуски появляться не должны.

**Способ 2. Отключить workflow вручную**

1. Откройте вкладку **Actions**.
2. В левом меню найдите `pages build and deployment`.
3. Нажмите **⋯ → Disable workflow**.

Это остановит автоматические запуски, но не удалит workflow из системы. Если позже вы снова переключите источник на ветку, он может вернуться.

**Способ 3. Добавить `.nojekyll` (не рекомендуется)**

Если по какой-то причине вы хотите оставить источник «Deploy from a branch», можно добавить в корень репозитория пустой файл `.nojekyll`. Это отключит обработку Jekyll, но workflow всё равно продолжит публиковать содержимое ветки. Проблема с README.md не решится.

## 💎 Рекомендация

Для вашего проекта (`bpmbpm/geoff_test`) правильная конфигурация такая:

1. **Settings → Pages → Source: GitHub Actions.**
2. **Actions → `pages build and deployment` → Disable workflow** (если он всё ещё виден).
3. Оставить только ваш `deploy.yml` — и запускать его вручную (`workflow_dispatch`), как мы обсуждали в прошлом ответе.

Тогда при правке файлов ничего не будет запускаться само, а по команде **Run workflow** будет собираться и публиковаться именно Geoff-сайт.

## 🔗 Ссылки

- GitHub Pages: настройка источника публикации — https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- GitHub Actions: отключение workflow — https://docs.github.com/en/actions/managing-workflow-runs/disabling-and-enabling-a-workflow
- О `.nojekyll` — https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#static-site-generators
