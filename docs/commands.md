# Контракт исполняемых команд

GitHub Actions и AI-агенты не должны знать детали конкретного стека. Они вызывают стабильные точки входа из `scripts/`.

## `bash scripts/ci`

Быстрые проверки, которые должны завершаться за разумное время и запускаться на каждый PR.

Типичный состав после адаптации под проект:

- форматирование/линт;
- typecheck/compile;
- unit tests;
- быстрая сборка;
- проверка схем/миграций без внешних production-ресурсов.

В чистом шаблоне команда проверяет обязательную структуру, базовый формат workflow skills и затем, если существует `scripts/project-ci`, вызывает её.

## `bash scripts/ci-full`

Полная проверка проекта.

Типичный состав:

- всё из fast CI;
- integration tests;
- e2e tests;
- тестовые миграции;
- контейнерная сборка;
- более тяжёлые security/quality checks.

В чистом шаблоне сначала выполняется `scripts/ci`, затем опциональный `scripts/project-ci-full`.

## `bash scripts/diagnostics [директория]`

Создаёт безопасный набор текстовой диагностики для анализа ошибки CI.

Требования:

- не выводить secrets и environment целиком;
- не архивировать `.env`, credentials, ключи и токены;
- хранить только технический контекст, нужный для воспроизведения ошибки;
- конкретный проект может расширить сбор через `scripts/project-diagnostics`.

## `bash scripts/package-skills [директория]`

Упаковывает базовые workflow skills из `.agents/skills/` в отдельные ZIP bundles для установки или обновления native Skills в ChatGPT Web.

По умолчанию результат создаётся в `dist/skills/`:

```text
dist/skills/
  dev-core.zip
  dev-build.zip
  dev-debug.zip
  dev-review.zip
  manifest.tsv
```

В каждом ZIP `SKILL.md` находится в корне архива. `manifest.tsv` содержит имя skill, имя архива и SHA-256 checksum.

Команда упаковывает только содержимое каталогов, в которых есть `SKILL.md`, и не собирает `.git`, repository secrets или произвольные файлы за пределами конкретного skill bundle.

Для выполнения нужна утилита `zip`; workflow `Упаковать skills` запускает эту же команду на GitHub-hosted runner и сохраняет результат как artifact.

## Project-specific hooks

Конкретный стек может подключаться через:

- `scripts/project-ci`;
- `scripts/project-ci-full`;
- `scripts/project-diagnostics`.

Workflow skills и GitHub Actions должны по возможности вызывать стабильные точки входа, а не дублировать команды package manager/framework в нескольких местах.

## Почему не `run: ${{ inputs.command }}`

Произвольный shell из workflow input, Issue или PR создаёт ненужный remote-code-execution интерфейс. Вместо этого добавляйте явные scripts с понятным контрактом и review в Git.

## Рекомендация для конкретного проекта

Не переписывайте workflow под каждый package manager. Сохраните стабильные `scripts/ci` и `scripts/ci-full`, а стековые команды держите в `scripts/project-*` или вызывайте из этих файлов напрямую.

Так ChatGPT/GitHub workflow остаётся неизменным даже при смене внутренней структуры приложения.
