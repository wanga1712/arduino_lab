# Архитектура Arduino Lab

## Физическая структура

```text
apps/
  web/src/
    app/                 композиция приложения и URL
    features/lab/        пользовательский сценарий лаборатории
    ui/                  общие визуальные примитивы
    infrastructure/
      api/               HTTP-клиент
      storage/           localStorage-кэш
  api/src/
    app/                 запуск и композиция Fastify
    routes/              HTTP-входы и ответы
    services/            прикладные сценарии
    repositories/        интерфейсы и PostgreSQL-реализации хранения
    infrastructure/
      database/          pool, транзакции и запуск миграций
packages/
  shared/src/
    circuit/             граф, netlist, физика и валидация
    execution/           контракт и BlinkExecutionEngine
    lesson/              конфигурация урока и состояние
    contracts/           API DTO и runtime-проверки
database/
  migrations/            последовательные SQL-миграции
deploy/                  конфигурация сервера №13 без секретов
docs/
  decisions/             согласованные архитектурные исключения
scripts/                 проверки архитектуры и служебные команды
```

Каталоги созданы заранее. Наполнение выполняется только на соответствующем шаге плана.

## Направление зависимостей

```text
apps/web ──┐
           ├──> packages/shared
apps/api ──┘

route -> service -> repository -> PostgreSQL driver
```

`packages/shared` не знает о React, Fastify и PostgreSQL. Web и API не импортируют друг друга. Инфраструктурные реализации зависят от внутренних интерфейсов, а не наоборот.

## Канонические владельцы

| Сущность | Единственное место |
| --- | --- |
| Circuit, компоненты и netlist | `packages/shared/src/circuit` |
| Диагностика электрической схемы | `packages/shared/src/circuit` |
| Blink-парсер и план исполнения | `packages/shared/src/execution` |
| LessonDefinition и состояние | `packages/shared/src/lesson` |
| API DTO и проверка payload | `packages/shared/src/contracts` |
| HTTP-семантика | `apps/api/src/routes` |
| Сохранение сессии | `apps/api/src/services` |
| SQL и отображение строк БД | `apps/api/src/repositories` |
| Вызовы API браузером | `apps/web/src/infrastructure/api` |
| Локальный кэш | `apps/web/src/infrastructure/storage` |
| Визуальное поведение урока | `apps/web/src/features/lab` |

Если новая функция не имеет однозначного владельца, сначала уточняется граница. Вторую реализацию в другом слое создавать нельзя.

## Публичные контракты

Каждая область экспортирует минимальный API через свой `index.ts`. Импорт приватных файлов другой области запрещён. Сериализуемые структуры версионируются и проверяются во время исполнения на обеих границах сети.

## Проверка архитектуры

На шаге 0 кодер добавляет кроссплатформенную команду `npm run check:architecture`. Она должна:

- отклонять рукописные модули длиннее 150 строк;
- проверять запрещённые импорты и циклы;
- запускать поиск дублирования с разумным порогом, исключив generated и lock-файлы.

Автоматическая проверка дополняет, но не заменяет ручной поиск смысловых дублей ревьюером.
