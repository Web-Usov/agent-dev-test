---
name: dev-review
description: Review an implementation or pull request against its specification, plan, constitution, tests and risks. Use for independent code review, pre-merge verification, scope checking and defect finding. Read-only by default.
---

<!-- version: 1.0.0 -->

# dev-review

Независимый режим проверки готового или почти готового изменения.

## Read-only по умолчанию

Не изменяй код, spec, plan, tasks или PR в рамках review, если пользователь явно не попросил исправить findings.

После команды на исправление выбери подходящий режим:

- `dev-build` — если исправление очевидно и соответствует существующему plan;
- `dev-debug` — если finding требует расследования/root cause;
- `dev-core` — если finding показывает проблему требований, scope или архитектуры.

## Перед началом

1. Прочитай `AGENTS.md` и `docs/constitution.md`.
2. Найди связанный Issue, spec, plan, tasks и ADR.
3. Получи diff/PR и результаты CI.
4. Определи области риска и необходимые предметные компетенции.
5. Просмотри metadata доступных auxiliary skills (`name` + `description`) и выбери минимальный набор для проверки сложных областей.

Не проси пользователя вручную перечислять security/frontend/database skills, если среда уже предоставляет их metadata.

## Что проверять

### Соответствие требованиям

- Реализация покрывает acceptance criteria.
- Не потеряны сценарии из spec.
- Нет незапланированного пользовательского поведения.
- Out-of-scope не был скрыто реализован.

### Соответствие plan/ADR

- Архитектура и контракты соответствуют plan.
- Отклонения документированы.
- Изменения данных/API имеют ожидаемую совместимость и migration path.

### Корректность реализации

Ищи прежде всего конкретные дефекты:

- неправильная логика и edge cases;
- race/concurrency/state issues;
- ошибки error handling;
- security/privacy regressions;
- несовместимые изменения контрактов;
- потерю данных;
- resource/performance regressions;
- недостаточную observability там, где она нужна.

Не превращай review в субъективный style audit, если стиль не влияет на correctness, maintainability или правила проекта.

### Проверки и тесты

- Новое поведение реально проверяется тестами.
- Тесты способны упасть при регрессии.
- Нет отключённых/ослабленных checks ради зелёного CI.
- CI соответствует фактическому diff.

### Документация и tasks

- `tasks.md` отражает фактическое состояние.
- Docs/ADR обновлены при изменении контрактов/решений.
- PR объясняет что изменено и как проверить.

## Формат findings

Сначала findings, от более серьёзных к менее серьёзным.

Для каждого finding укажи:

1. severity: `critical`, `high`, `medium` или `low`;
2. конкретный файл/участок/контракт;
3. наблюдаемую проблему;
4. почему это проблема;
5. минимальное направление исправления.

Не выдавай гипотезу как подтверждённый дефект. Если evidence недостаточно, пометь finding как вопрос или риск.

## Завершение review

После findings дай краткий verdict:

- `APPROVE` — блокирующих проблем не найдено;
- `APPROVE WITH NOTES` — только неблокирующие замечания;
- `CHANGES REQUIRED` — есть findings, которые нужно исправить до merge;
- `BLOCKED` — review нельзя завершить из-за отсутствующих данных/CI/spec.

Даже при `APPROVE` явно назови оставшиеся непроверенные риски, если они есть.