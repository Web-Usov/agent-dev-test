# План: skills-based workflow

Spec: `specs/003-agent-skills/spec.md`

Issue: #3

## Решение

Хранить 4 базовых workflow skill в `.agents/skills/<name>/SKILL.md`. Они задают режим работы агента, а не предметную специализацию. Подробный процесс переносится из `AGENTS.md` в skills; `AGENTS.md` остаётся bootstrap-файлом с invariants, source of truth и routing.

## Структура

```text
.agents/skills/
  README.md
  dev-core/SKILL.md
  dev-build/SKILL.md
  dev-debug/SKILL.md
  dev-review/SKILL.md
```

Каждый `SKILL.md` использует минимальный переносимый YAML frontmatter:

```yaml
---
name: dev-core
description: ...
---
```

Версия хранится обычным HTML-комментарием `<!-- version: 1.0.0 -->`, чтобы не расширять обязательный frontmatter неизвестными полями.

## Routing

Если пользователь явно указал workflow skill, он имеет приоритет. Иначе агент выбирает один из четырёх режимов по намерению задачи.

Перед существенной работой workflow skill:

1. определяет необходимые компетенции;
2. просматривает metadata доступных auxiliary skills, если среда их предоставляет;
3. выбирает минимальный релевантный набор;
4. загружает/применяет только выбранные skills;
5. продолжает работу даже без auxiliary skill, если он недоступен и не обязателен.

## ChatGPT Web

`.agents/skills/` остаётся source of truth. Для native ChatGPT Skills пользователь устанавливает ZIP каждого skill через Skills UI. Установленная копия считается производной и не синхронизируется автоматически с GitHub.

Добавляется `scripts/package-skills`, создающий отдельные ZIP с `SKILL.md` в корне архива. GitHub Actions публикует их как artifact после изменений skills в `main` и по ручному запуску.

## Codex

Project skills лежат в стандартном `.agents/skills/`. Локальные/персональные capability skills могут обнаруживаться самим Codex. Базовые workflow skills не содержат жёсткий список frontend/backend навыков.

## CI

`scripts/ci` дополнительно проверяет:

- наличие четырёх `SKILL.md`;
- открывающий и закрывающий YAML frontmatter;
- наличие `name` и `description`;
- совпадение `name` с именем каталога;
- shell syntax `scripts/package-skills`.

## Безопасность

- skills не получают права выполнять shell из текста Issue/PR/comment;
- упаковка берёт только содержимое конкретного каталога skill;
- workflow имеет `contents: read`;
- secrets в artifact не включаются.

## Изменяемые файлы

- `.agents/skills/**`;
- `AGENTS.md`;
- `README.md`;
- `docs/development-process.md`;
- `scripts/ci`;
- `scripts/package-skills`;
- `.github/workflows/package-skills.yml`;
- `specs/003-agent-skills/*`.

## Проверки

- быстрый CI на PR;
- ручная проверка ZIP layout через `unzip -l` при необходимости;
- после открытия PR проверить GitHub Actions run и artifact workflow отдельно после merge или manual dispatch.