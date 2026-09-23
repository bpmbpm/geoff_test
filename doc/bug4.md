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

