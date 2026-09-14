# Skills registry

`.agents/skills/` — канонический, version-controlled источник базовых workflow skills этого шаблона.

## Базовые workflow skills

| Skill | Версия | Назначение |
|---|---:|---|
| `dev-core` | 1.0.0 | Clarification, scope, spec, plan, tasks и выбор следующего режима |
| `dev-build` | 1.0.0 | Реализация согласованной задачи, тесты, CI и PR |
| `dev-debug` | 1.0.0 | Reproduction, evidence, root cause, минимальный fix и regression test |
| `dev-review` | 1.0.0 | Независимое review против spec/plan/constitution; read-only по умолчанию |

Версия также записана внутри каждого `SKILL.md` как HTML-комментарий. Она намеренно не добавлена в YAML frontmatter: обязательный переносимый frontmatter оставлен минимальным — только `name` и `description`.

## Workflow skills и capability skills

Эти 4 skill отвечают на вопрос **как работать**, а не **какой технологией владеть**.

Capability skills могут быть project-local, персональными или установленными средой, например:

- frontend design / React;
- Playwright;
- PostgreSQL;
- NestJS;
- security review;
- accessibility.

Workflow skill должен автоматически выбрать минимальный релевантный набор доступных capability skills по их `name` и `description`. Жёсткий список capability skills внутри `dev-*` запрещён.

## Приоритет выбора

1. Явно указанный пользователем workflow skill имеет наивысший приоритет.
2. Если workflow skill не указан, агент выбирает один базовый режим по намерению задачи.
3. Затем агент оценивает доступные auxiliary/capability skills.
4. Используется минимальный набор, который существенно помогает задаче.
5. Если подходящего вспомогательного skill нет, работа продолжается без него, если это безопасно и возможно.

## Codex

Codex может использовать project skills из `.agents/skills/` и отдельно обнаруживать локальные/персональные skills. Пользователю не нужно вручную перечислять capability skills для каждой задачи.

## ChatGPT Web

Файлы в `.agents/skills/` сами по себе не означают, что skill установлен в native ChatGPT Skills UI.

Для ChatGPT Web:

1. GitHub-версия остаётся source of truth.
2. Выполните `bash scripts/package-skills` или используйте GitHub Actions workflow `Упаковать skills`.
3. Возьмите ZIP нужного skill.
4. Установите/обновите его через ChatGPT → Skills → Upload from computer.
5. После установки ChatGPT может выбирать релевантный skill автоматически; пользователь также может вызвать его явно.

Установленная в ChatGPT копия является производной. Автоматическая синхронизация GitHub → ChatGPT в этом шаблоне не предполагается.

## Обновление версии

При изменении поведения skill:

- обновить `<!-- version: X.Y.Z -->` в его `SKILL.md`;
- обновить таблицу в этом файле;
- заново упаковать и переустановить skill в ChatGPT, если нужна новая native-копия;
- проверить CI.

Рекомендуемая схема:

- patch — уточнение инструкций без изменения назначения;
- minor — новое совместимое поведение;
- major — изменение назначения, routing или существенных контрактов.

## Формат bundle

`scripts/package-skills` создаёт отдельный ZIP на каждый каталог skill так, чтобы `SKILL.md` лежал в корне ZIP. Это позволяет не устанавливать все workflow skills одним монолитным пакетом.