# ADR-0001: Базовая архитектура Task Manager MVP

- Статус: proposed
- Дата: 2026-09-15
- Связанные spec/Issue/PR: `specs/001-task-manager-mvp/`, #1

## Контекст

Проект создаётся с нуля как небольшое web-приложение для управления задачами. Пользователь явно задал NestJS для backend и PostgreSQL для хранения данных. Нужно выбрать минимальную архитектуру frontend/backend, структуру репозитория и способ доступа к БД так, чтобы первая версия была простой, проверяемой и не требовала преждевременной инфраструктуры.

## Решение

Для MVP принять следующую базовую архитектуру:

- один monorepo на pnpm workspaces;
- `apps/api` — NestJS + TypeScript;
- `apps/web` — React + Vite + TypeScript;
- PostgreSQL — единственное постоянное хранилище;
- Prisma — ORM и migration tool;
- REST API с prefix `/api`;
- Swagger/OpenAPI — документация публичного API;
- TanStack Query — управление server state на frontend;
- Docker Compose — локальный PostgreSQL;
- без authentication/authorization в первой версии;
- без CQRS, event bus, repository abstraction и global frontend state manager до появления реальной необходимости.

## Последствия

### Положительные

- один TypeScript stack на frontend/backend;
- быстрый greenfield setup;
- простая локальная разработка;
- Prisma schema и migrations дают явную модель данных;
- REST + Swagger легко тестировать и документировать;
- monorepo упрощает согласованные изменения API и UI;
- минимальное количество архитектурных слоёв для маленького CRUD-приложения.

### Отрицательные

- frontend и backend остаются связанными по темпу изменений внутри одного репозитория;
- Prisma становится значимой инфраструктурной зависимостью;
- отсутствие auth ограничивает MVP локальным/dev использованием;
- DTO/types могут начать дублироваться между frontend и backend до появления generated/shared contracts.

### Риски

- по мере роста приложения monorepo scripts и CI могут потребовать отдельного build orchestrator;
- при появлении сложной доменной логики прямого service→Prisma доступа может стать недостаточно;
- при большом списке задач потребуется pagination;
- публичный deployment нельзя делать безопасно без отдельной спецификации authentication/authorization.

## Рассмотренные альтернативы

### Next.js вместо React + Vite

Не выбран для MVP, потому что приложение является SPA поверх отдельного NestJS API, а SSR/server actions не дают обязательной пользы для текущего scope.

### TypeORM вместо Prisma

Не выбран: Prisma даёт компактную декларативную schema, удобные migrations и типизированный client при меньшем количестве boilerplate для данного проекта.

### GraphQL вместо REST

Не выбран: CRUD-модель маленькая, REST проще для первой версии и естественно документируется OpenAPI.

### Polyrepo

Не выбран: отдельные репозитории для frontend/backend увеличили бы количество CI/configuration work без заметной пользы на текущем масштабе.

### CQRS/DDD с отдельным repository layer с первого дня

Не выбран: для одной сущности `Task` это добавит слои без достаточной доменной сложности. Архитектуру можно усилить позже при появлении бизнес-правил и дополнительных bounded contexts.

## Условия пересмотра

Решение стоит пересмотреть, если:

- появляется multi-user/authentication;
- frontend требует SSR/SEO;
- появляются сложные доменные процессы, события или несколько хранилищ;
- API начинают использовать внешние клиенты;
- DTO начинают существенно дублироваться между frontend/backend;
- объём данных требует pagination/caching;
- monorepo CI становится медленным и требует Turborepo/Nx или другого orchestrator.
