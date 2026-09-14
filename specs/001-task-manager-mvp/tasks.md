# Задачи: Task Manager MVP

## Правила

- Каждая задача должна давать проверяемый результат.
- Задачи идут в порядке выполнения.
- Любая задача должна быть объяснима через `spec.md` и `plan.md`.
- При появлении нового scope сначала обновить spec/plan.
- Реализацию вести небольшими логическими коммитами/PR; не смешивать незапланированный рефакторинг.

## Подготовка

- [ ] T001 Согласовать рабочие допущения MVP: single-user без auth, React + Vite, Prisma, REST.
- [ ] T002 Принять или скорректировать `docs/adr/0001-task-manager-mvp-architecture.md`.
- [ ] T003 Адаптировать `docs/constitution.md` под подтверждённый стек проекта, не дублируя детали текущей feature-spec.
- [ ] T004 Зафиксировать project-specific команды разработки и CI в `docs/commands.md`.

## Основа проекта

- [ ] T010 Инициализировать pnpm workspace с `apps/api` и `apps/web`.
- [ ] T011 Добавить корневые scripts для dev/lint/typecheck/test/build.
- [ ] T012 Добавить `.env.example` и проверить `.gitignore`.
- [ ] T013 Добавить `compose.yaml` с PostgreSQL для локальной разработки.

## Данные

- [ ] T020 Подключить Prisma к `apps/api`.
- [ ] T021 Описать `TaskStatus`, `TaskPriority` и модель `Task` согласно plan.
- [ ] T022 Создать первую Prisma migration.
- [ ] T023 Проверить воспроизводимость: чистая PostgreSQL → migrations → рабочая schema.

## Backend

- [ ] T030 Инициализировать NestJS приложение и `/api/health`.
- [ ] T031 Добавить Prisma module/service с корректным lifecycle.
- [ ] T032 Реализовать `tasks` module, DTO и validation.
- [ ] T033 Реализовать `POST /api/tasks`.
- [ ] T034 Реализовать `GET /api/tasks` с `q/status/priority/sort/order`.
- [ ] T035 Реализовать `GET /api/tasks/:id`.
- [ ] T036 Реализовать `PATCH /api/tasks/:id`, включая инвариант `completedAt`.
- [ ] T037 Реализовать `DELETE /api/tasks/:id`.
- [ ] T038 Подключить Swagger/OpenAPI и описать DTO/query params.
- [ ] T039 Нормализовать 400/404/500 поведение и не отдавать raw Prisma errors.

## Backend tests

- [ ] T040 Добавить unit tests на status transitions и `completedAt`.
- [ ] T041 Добавить unit tests на not-found и фильтрацию/search mapping.
- [ ] T042 Добавить integration/e2e tests с PostgreSQL: create/list/get/update/delete.
- [ ] T043 Добавить integration/e2e tests на combined filters/search и ошибки 400/404.

## Frontend — основа

- [ ] T050 Инициализировать React + Vite + TypeScript приложение.
- [ ] T051 Подключить TanStack Query и типизированный API слой.
- [ ] T052 Реализовать application shell, responsive layout и базовые UI primitives.
- [ ] T053 Реализовать loading/empty/error states.

## Frontend — задачи

- [ ] T060 Реализовать список task cards.
- [ ] T061 Реализовать поиск по `q`.
- [ ] T062 Реализовать фильтры `status` и `priority`, включая совместное применение.
- [ ] T063 Реализовать create task form/modal.
- [ ] T064 Реализовать edit task form/modal.
- [ ] T065 Реализовать быстрое изменение статуса.
- [ ] T066 Реализовать удаление с подтверждением.
- [ ] T067 Реализовать отображение due date и признак просроченной задачи.
- [ ] T068 Проверить cache invalidation/refetch после всех mutations.

## Frontend tests

- [ ] T070 Добавить component tests на loading/empty/error states.
- [ ] T071 Добавить tests на create/edit forms и validation errors.
- [ ] T072 Добавить tests на filters/search.
- [ ] T073 Добавить test на delete confirmation и status change.

## CI и tooling

- [ ] T080 Реализовать `scripts/project-ci`: lint, typecheck, unit tests, build.
- [ ] T081 Реализовать `scripts/project-ci-full`: PostgreSQL, migrations, integration/e2e tests.
- [ ] T082 Проверить GitHub Actions на чистом runner и сохранить существующие минимальные permissions.
- [ ] T083 Проверить, что failures дают достаточную диагностику и не содержат secrets.

## Документация

- [ ] T090 Обновить README: назначение приложения, стек, quick start и ограничения MVP.
- [ ] T091 Документировать local development, env и DB migrations.
- [ ] T092 Зафиксировать Swagger URL и основные API endpoints.
- [ ] T093 Явно указать, что без auth MVP не предназначен для публичного production deployment.

## Converge

- [ ] T100 Запустить `bash scripts/ci`.
- [ ] T101 Запустить `bash scripts/ci-full`.
- [ ] T102 Сверить реализацию со всеми AC-1…AC-12 из `spec.md`.
- [ ] T103 Проверить отсутствие незапланированного scope: auth, Kanban DnD, tags, projects, notifications.
- [ ] T104 Зафиксировать дальнейшие улучшения отдельными Issue, не расширяя MVP перед merge.
