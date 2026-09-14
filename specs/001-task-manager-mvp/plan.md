# Технический план: Task Manager MVP

## Связанные документы

- Spec: `specs/001-task-manager-mvp/spec.md`
- Issue: #1
- ADR: `docs/adr/0001-task-manager-mvp-architecture.md`

## Краткое решение

Собрать небольшой TypeScript monorepo на pnpm workspaces: `apps/api` с NestJS и Prisma, `apps/web` с React + Vite, PostgreSQL как единственное постоянное хранилище. Backend предоставляет REST API `/api/tasks`, frontend работает с ним через HTTP и использует server-state подход без отдельного глобального state manager. Локальная БД запускается через Docker Compose. Swagger/OpenAPI используется как исполняемая документация API.

## Затрагиваемые части системы

- backend: `apps/api`;
- frontend: `apps/web`;
- данные: PostgreSQL + Prisma migrations;
- инфраструктура разработки: pnpm workspace, Docker Compose, `.env.example`;
- CI: project-specific hooks для lint/test/build;
- документация: README/commands/ADR/spec.

## Предлагаемая структура репозитория

```text
apps/
  api/
    src/
      app.module.ts
      main.ts
      prisma/
      tasks/
        dto/
        tasks.controller.ts
        tasks.service.ts
        tasks.module.ts
    prisma/
      schema.prisma
      migrations/
    test/
  web/
    src/
      api/
      components/
      features/tasks/
      pages/
      App.tsx
      main.tsx
    tests/
packages/
  # пока пусто; shared package не создаём без реальной необходимости
compose.yaml
pnpm-workspace.yaml
package.json
.env.example
```

## Архитектурные решения

### Monorepo

Использовать pnpm workspaces. Один репозиторий содержит frontend, backend, спецификацию и CI. На MVP это упрощает синхронное изменение API/UI и единый pipeline.

### Backend

- NestJS;
- REST API;
- Prisma как ORM и migration tool;
- глобальный `ValidationPipe` с whitelist/transform;
- Swagger/OpenAPI;
- стандартный Nest logger;
- CORS настраивается через env.

Модуль `tasks` делится на controller/service/data access через Prisma. Отдельный repository abstraction на MVP не вводится: он не даёт достаточной пользы при одном хранилище и простой модели.

### Frontend

- React + Vite + TypeScript;
- TanStack Query для загрузки, мутаций и cache invalidation;
- локальное состояние React для формы и UI-фильтров;
- без Redux/Zustand на MVP;
- адаптивный single-page интерфейс;
- task form используется и для create, и для edit.

### API contract

Базовый prefix: `/api`.

Endpoints:

- `GET /api/health` — health check приложения;
- `GET /api/tasks` — список, поиск и фильтрация;
- `GET /api/tasks/:id` — одна задача;
- `POST /api/tasks` — создание;
- `PATCH /api/tasks/:id` — частичное изменение, включая статус;
- `DELETE /api/tasks/:id` — удаление.

`GET /api/tasks` query params:

- `q?: string`;
- `status?: TODO | IN_PROGRESS | DONE`;
- `priority?: LOW | MEDIUM | HIGH`;
- `sort?: createdAt | updatedAt | dueAt` (default `createdAt`);
- `order?: asc | desc` (default `desc`).

Ответ списка на MVP — массив задач без pagination.

### Ошибки

Использовать предсказуемые HTTP-коды:

- 400 — validation/query errors;
- 404 — задача не найдена;
- 500 — непредвиденная серверная ошибка.

На первом этапе можно использовать стандартную Nest error shape, не вводя собственный error envelope без необходимости.

## Изменения данных

### Enum `TaskStatus`

- `TODO`
- `IN_PROGRESS`
- `DONE`

### Enum `TaskPriority`

- `LOW`
- `MEDIUM`
- `HIGH`

### Таблица `Task`

```text
id           UUID PK
 title        VARCHAR(200) NOT NULL
 description  TEXT NULL
 status       TaskStatus NOT NULL DEFAULT TODO
 priority     TaskPriority NOT NULL DEFAULT MEDIUM
 dueAt        TIMESTAMPTZ NULL
 completedAt  TIMESTAMPTZ NULL
 createdAt    TIMESTAMPTZ NOT NULL DEFAULT now()
 updatedAt    TIMESTAMPTZ NOT NULL
```

Индексы:

- `status`;
- `priority`;
- `dueAt`;
- `createdAt`.

Правило `completedAt` реализуется на уровне service: при переходе в `DONE` записывается текущее время, при переходе из `DONE` — `null`.

## DTO

### CreateTaskDto

- `title: string` — trim, 1–200;
- `description?: string | null` — до 5000;
- `priority?: TaskPriority`;
- `dueAt?: ISO datetime | null`.

`status` при создании не обязателен и по умолчанию `TODO`. Для уменьшения вариантов первого сценария frontend создаёт новые задачи именно в `TODO`.

### UpdateTaskDto

Partial от изменяемых полей:

- `title`;
- `description`;
- `status`;
- `priority`;
- `dueAt`.

Системные поля (`id`, timestamps) клиент менять не может.

## UI первой версии

Одна основная страница:

1. Header с названием приложения и кнопкой «Новая задача».
2. Поиск.
3. Фильтры `status` и `priority`.
4. Список task cards.
5. Каждая карточка показывает title, status, priority, due date и признак просрочки.
6. Действия: изменить, быстро сменить статус, удалить.
7. Create/edit form открывается в modal/drawer.
8. Empty state при отсутствии задач.
9. Error state с возможностью повторить запрос.

Kanban и drag-and-drop сознательно откладываются.

## Стратегия тестирования

### Backend unit

- правила `completedAt`;
- обработка not found;
- фильтры/search mapping;
- create/update validation-related service behaviour.

### Backend integration/e2e

С реальным PostgreSQL test service:

- create → get/list;
- update/status transition;
- filtering/search;
- delete;
- 400/404 cases.

### Frontend unit/component

Vitest + Testing Library:

- empty/loading/error states;
- создание/редактирование через форму;
- применение фильтров;
- подтверждение удаления;
- визуальное отображение статуса/приоритета.

### End-to-end browser

Не делать обязательным для самого первого PR. Добавить Playwright отдельным небольшим шагом после появления стабильного UI, если это оправдано.

## План наблюдаемости и диагностики

- `GET /api/health` для smoke check;
- Nest logger для startup и server errors;
- Prisma ошибки не должны утекать клиенту в сыром виде;
- CI при падении должен сохранять диагностический artifact по существующим правилам шаблона;
- frontend показывает пользователю понятное общее сообщение, а техническая ошибка остаётся в console/devtools в development.

## Безопасность

- credentials только через env;
- `.env` в `.gitignore`, `.env.example` без секретов;
- DTO whitelist не позволяет mass assignment неизвестных полей;
- параметризованные запросы Prisma;
- ограничить длину текстовых полей;
- CORS через конфигурацию, не wildcard для production-профиля;
- поскольку auth отсутствует, MVP предназначен только для локального/dev использования до отдельного решения о deployment.

## CI

Адаптировать существующий workflow через project hooks:

`scripts/project-ci`:

- install/check lockfile;
- lint;
- typecheck;
- unit tests;
- production build обоих apps.

`scripts/project-ci-full` дополнительно:

- поднять PostgreSQL service;
- применить Prisma migrations;
- запустить backend integration/e2e tests.

## Порядок реализации

1. Зафиксировать ADR и project-specific команды.
2. Инициализировать pnpm monorepo.
3. Поднять PostgreSQL через Docker Compose и Prisma schema/migration.
4. Реализовать NestJS `tasks` module + Swagger + health.
5. Добавить backend tests.
6. Реализовать React application shell и API client.
7. Реализовать list/search/filter.
8. Реализовать create/edit/status/delete flows.
9. Добавить frontend tests.
10. Подключить project CI hooks.
11. Обновить README и проверить acceptance criteria.

## Риски

- риск: API и frontend DTO разойдутся.
  - снижение риска: Swagger как контракт и API integration tests; shared/generated client можно добавить позже.
- риск: отсутствие auth случайно воспримут как production-ready.
  - снижение риска: явно отметить local/dev назначение MVP в README.
- риск: слишком раннее усложнение архитектуры.
  - снижение риска: без repository layer, event bus, CQRS и global state manager на первой версии.
- риск: отсутствие pagination станет проблемой при большом числе задач.
  - снижение риска: контракт списка можно расширить позже отдельной spec.

## Альтернативы

### Next.js вместо React + Vite

Не выбран: SSR/server actions не дают заметной пользы для локального SPA с отдельным NestJS API, а архитектура становится тяжелее.

### TypeORM вместо Prisma

Не выбран: Prisma даёт компактную schema, миграции и типизированный client, что удобно для небольшого greenfield проекта.

### GraphQL вместо REST

Не выбран: модель CRUD простая, REST уменьшает количество инфраструктуры и проще проверяется Swagger/e2e тестами.

### Общий `packages/contracts` с первого дня

Не выбран: пока API маленькое, общий пакет создаст дополнительную связанность. Добавить при появлении фактического дублирования или генерации клиента из OpenAPI.

## Открытые технические вопросы

Нет блокирующих вопросов для MVP. Решения из ADR имеют статус proposed до согласования/начала реализации.
