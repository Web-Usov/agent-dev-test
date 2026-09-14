# Agent Dev Template

Шаблон репозитория для облачной разработки через ChatGPT.com + GitHub без обязательного локального coding agent.

Основная идея: **GitHub хранит состояние проекта, спецификации, skills и историю решений, GitHub Actions исполняет проверки, а AI-агент работает через Issue → spec/plan/tasks → ветку → PR → CI → review → merge.**

## Для чего этот шаблон

Этот репозиторий не привязан к Node.js, Python, Go или другому стеку. Он задаёт процесс и контракт, который конкретный проект дополняет своими командами сборки, тестирования и специализированными capability skills.

## Skills-based workflow

Базовый процесс разбит на 4 небольших workflow skill:

- `dev-core` — clarification, scope, spec, plan, tasks;
- `dev-build` — реализация согласованной задачи, тесты, CI и PR;
- `dev-debug` — reproduction, evidence, root cause, минимальный fix и regression test;
- `dev-review` — независимое review; read-only по умолчанию.

Канонические версии лежат в `.agents/skills/`.

Если пользователь явно указал workflow skill, он имеет приоритет. Если нет — агент выбирает подходящий режим сам. После выбора workflow агент может автоматически подключить минимальный набор доступных специализированных skills по их `name`/`description`, например frontend, Playwright или PostgreSQL. Пользователю не требуется вручную перечислять их для каждой задачи.

Подробности: [.agents/skills/README.md](.agents/skills/README.md).

## Цикл разработки

```text
Идея / Issue
    ↓
dev-core
    ↓
spec → plan → tasks
    ↓
dev-build
    ↓
Pull Request + CI
    ↓
dev-debug при проблемах
    ↓
dev-review
    ↓
Merge
```

## Быстрый старт нового проекта

1. Создайте новый репозиторий через **Use this template**.
2. Заполните `docs/constitution.md` правилами конкретного проекта.
3. Адаптируйте `scripts/ci` и `scripts/ci-full` под стек приложения либо добавьте хуки `scripts/project-ci` и `scripts/project-ci-full`.
4. Для первой функции создайте Issue и запустите процесс через `dev-core` либо просто опишите задачу — агент должен выбрать режим сам.
5. Создайте каталог функции на основе `specs/_template/`, например `specs/001-auth/`, и доведите `spec.md` → `plan.md` → `tasks.md` до состояния ready for implementation.
6. Реализуйте задачу через `dev-build`, PR и CI.
7. Перед merge выполните `dev-review`.

Подробный процесс: [docs/development-process.md](docs/development-process.md).

## Codex и ChatGPT Web

`.agents/skills/` — source of truth для базовых skills.

**Codex** может использовать project skills из этого каталога и отдельно обнаруживать локальные/персональные skills.

**ChatGPT Web** использует native installed skills. Файлы в GitHub сами по себе не устанавливают skill в ChatGPT. Для установки/обновления:

```bash
bash scripts/package-skills
```

Команда создаёт отдельные ZIP в `dist/skills/`. То же самое делает workflow **Упаковать skills**, который сохраняет bundles как GitHub Actions artifact. Нужный ZIP устанавливается через ChatGPT → Skills → Upload from computer.

Установленная в ChatGPT копия считается производной; GitHub остаётся каноническим источником и автоматическая синхронизация между ними в шаблоне не предполагается.

## Структура

```text
.agents/
  skills/                workflow skills и их registry
.github/
  ISSUE_TEMPLATE/        формы Issue на русском языке
  workflows/             CI, packaging и ручные проверки
  pull_request_template.md
specs/
  _template/             шаблоны spec / plan / tasks
docs/
  constitution.md        постоянные правила проекта
  development-process.md
  commands.md            контракт исполняемых команд
  adr/                    архитектурные решения
scripts/
  ci                      быстрые проверки
  ci-full                 полные проверки
  diagnostics             диагностический пакет для CI
  package-skills          упаковка ChatGPT skill bundles
AGENTS.md                 bootstrap, invariants и skills routing
```

## Совместимость со Spec Kit

Структура сохраняет смысловую цепочку GitHub Spec Kit: **constitution → specify → plan → tasks → implement → converge**, но не копирует и не форкает сам Spec Kit. Его можно подключить поверх шаблона, если проекту нужен официальный CLI или дополнительные skills.

Наши `dev-*` skills крупнее отдельных фаз Spec Kit: они задают рабочий режим, чтобы пользователю не приходилось вручную переключаться между множеством микрокоманд.

## GitHub Actions как execution layer

Шаблон предоставляет:

- `CI` — быстрые проверки на `push` и `pull_request`;
- `Полный CI` — тяжёлые проверки, запускаемые вручную или через reusable workflow;
- `Упаковать skills` — создание ZIP bundles для установки/обновления native ChatGPT Skills.

При ошибке CI создаётся диагностический artifact, удобный для `dev-debug`.

GitHub Actions здесь **не является универсальным удалённым shell**. Произвольные команды из Issue, PR или комментариев не выполняются.

## Что сознательно не входит

- staging/production hosting;
- Railway, Render, Vercel, Fly.io, VPS и другие runtime providers;
- production secrets и production DB migrations;
- выбор конкретного backend/frontend framework;
- автоматическая публикация или обновление skills в аккаунте ChatGPT;
- глобальная библиотека персональных capability skills.

GitHub Pages можно использовать отдельно для публикации документации.

## Язык проекта

Документация, Issue и Pull Request в проектах на основе этого шаблона ведутся **на русском языке**, если конкретный проект явно не изменил это правило в `docs/constitution.md`.
