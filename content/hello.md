+++
title = "Hello Semantic World"
type = "Note"
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
