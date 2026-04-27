
# 🗂️ Лабораторная работа №31-32: Введение в SQLite и Entity Framework Core

> **Студент:** SDA
> **Группа:** Исп-232
> **Дата выполнения:** 28.04.2026


---

## 📌 Краткое описание работы

В данной лабораторной работе изучены основы работы с базами данных в веб-приложениях ASP.NET Core:

- ✅ Подключение SQLite — лёгкой встраиваемой БД
- ✅ Настройка Entity Framework Core через подход Code First
- ✅ Создание и применение миграций для управления схемой БД
- ✅ Выполнение CRUD-операций через LINQ вместо ручного SQL
- ✅ Реализация поиска, фильтрации, пагинации и статистики
- ✅ Логирование сгенерированных SQL-запросов для отладки

---

## 🛠️ Полезные команды dotnet ef

| Команда | Описание |
|---------|----------|
| `dotnet ef migrations add InitialCreate` | Создать первую миграцию |
| `dotnet ef migrations add AddDueDateToTask` | Создать миграцию для нового поля |
| `dotnet ef database update` | Применить все миграции к БД |
| `dotnet ef migrations list` | Показать список миграций |
| `dotnet ef migrations remove` | Удалить последнюю неприменённую миграцию |
| `dotnet ef database drop` | Удалить базу данных (для тестов) |

---

## 📁 Структура проекта

```
Lab31-32_EFCore/
├── TaskDb/
│   ├── Controllers/
│   │   └── TasksController.cs    # API-контроллер с CRUD + LINQ
│   ├── Data/
│   │   └── AppDbContext.cs       # Контекст БД (DbContext)
│   ├── Models/
│   │   ├── TaskItem.cs           # Модель задачи (таблица Tasks)
│   │   └── TaskDtos.cs           # DTO для запросов/ответов
│   ├── Migrations/               # Файлы миграций
│   │   ├── 20260428_InitialCreate.cs
│   │   └── 20260428_AddDueDateToTask.cs
│   ├── Properties/
│   ├── appsettings.json          # Строка подключения к БД
│   ├── appsettings.Development.json # Настройки логирования
│   ├── Program.cs                # Точка входа и настройка DI
│   └── TaskDb.csproj             # Зависимости проекта
├── img/                          # Скриншоты для отчёта
│   ├── gitPushLab31-32_Фамилия.png
│   ├── step4_migrationLab31-32_Фамилия.png
│   ├── step5_crudLab31-32_Фамилия.png
│   ├── step6_linqLab32_Фамилия.png
│   ├── step7_migrationLab32_Фамилия.png
│   └── step8_sqlLogsLab32_Фамилия.png
├── .editorconfig
├── .gitignore
└── README.md                     # Этот файл
```

---

## 🌐 Список реализованных маршрутов

| Метод | Маршрут | Описание | Параметры |
|-------|---------|----------|-----------|
| `GET` | `/api/tasks` | Получить все задачи | `?completed`, `?priority` |
| `GET` | `/api/tasks/{id}` | Получить задачу по ID | `id` в пути |
| `POST` | `/api/tasks` | Создать новую задачу | Body: `CreateTaskDto` |
| `PUT` | `/api/tasks/{id}` | Полностью обновить задачу | Body: `UpdateTaskDto` |
| `PATCH` | `/api/tasks/{id}/complete` | Переключить статус задачи | `id` в пути |
| `PATCH` | `/api/tasks/complete-all` | ✅ Пометить все невыполненные как выполненные | — |
| `DELETE` | `/api/tasks/{id}` | Удалить задачу по ID | `id` в пути |
| `DELETE` | `/api/tasks/completed` | ✅ Удалить все выполненные задачи | — |
| `GET` | `/api/tasks/search` | Поиск по тексту и фильтрам | `?query`, `?priority`, `?completed` |
| `GET` | `/api/tasks/stats` | Статистика по задачам | — |
| `GET` | `/api/tasks/paged` | Пагинация задач | `?page`, `?pageSize` |
| `GET` | `/api/tasks/overdue` | Просроченные задачи | — |


---

## 🔄 Таблица применённых миграций

| Миграция | Дата | Описание изменений |
|----------|------|-------------------|
| `InitialCreate` | 2026-04-28 | Создание таблицы `Tasks` с полями: Id, Title, Description, Priority, IsCompleted, CreatedAt + seed-данные |
| `AddDueDateToTask` | 2026-04-28 | Добавление колонки `DueDate` (nullable) для отслеживания дедлайнов |

---

## 🔀 Сравнительная таблица: LINQ vs SQL

| LINQ (C#) | SQL (генерирует EF Core) | Описание |
|-----------|---------------------------|----------|
| `.Where(t => t.IsCompleted == false)` | `WHERE is_completed = 0` | Фильтрация |
| `.Where(t => t.Title.Contains("EF"))` | `WHERE title LIKE '%EF%'` | Поиск по подстроке |
| `.OrderBy(t => t.CreatedAt)` | `ORDER BY created_at ASC` | Сортировка по возрастанию |
| `.OrderByDescending(t => t.CreatedAt)` | `ORDER BY created_at DESC` | Сортировка по убыванию |
| `.Take(10)` | `LIMIT 10` | Ограничение количества |
| `.Skip(20).Take(10)` | `OFFSET 20 LIMIT 10` | Пагинация |
| `.Count()` | `SELECT COUNT(*)` | Подсчёт записей |
| `.Count(t => t.IsCompleted)` | `SELECT COUNT(*) WHERE is_completed = 1` | Условный подсчёт |
| `.Any(t => t.Priority == "High")` | `SELECT EXISTS(...)` | Проверка наличия |
| `.Max(t => t.Id)` | `SELECT MAX(id)` | Максимальное значение |
| `.GroupBy(t => t.Priority)` | `GROUP BY priority` | Группировка |
| `.Select(t => t.Title)` | `SELECT title` | Выбор конкретных полей |

---

## 📊 Итоговая сравнительная таблица: Хранение в памяти vs EF Core + SQLite

| Концепция | Хранение в памяти (`static List<T>`) | EF Core + SQLite |
|-----------|-------------------------------------|------------------|
| **Хранение данных** | `static List<T>` в RAM | Файл `.db` на диске |
| **После перезапуска** | ❌ Данные пропадают | ✅ Данные сохраняются |
| **Поиск по условию** | LINQ to Objects (в памяти) | LINQ to Entities → SQL (в БД) |
| **Создание структуры** | Не нужно | Миграции (`dotnet ef`) |
| **Начальные данные** | Хардкод в коде | `HasData()` в миграции |
| **Получение данных** | `list.FirstOrDefault(...)` | `await db.Table.FindAsync(id)` |
| **Добавление** | `list.Add(item)` | `db.Table.Add(item) + SaveChangesAsync()` |
| **Удаление** | `list.Remove(item)` | `db.Table.Remove(item) + SaveChangesAsync()` |
| **Масштабируемость** | ❌ Ограничена оперативной памятью | ✅ Гигабайты данных на диске |
| **Транзакции** | ❌ Нет | ✅ Встроены в EF Core |

---

## 💡 Главные выводы

1. **EF Core — это «переводчик» между C# и SQL**. Вы пишете типобезопасный LINQ-код, а фреймворк автоматически генерирует оптимальные SQL-запросы. Это снижает количество ошибок и ускоряет разработку.

2. **Миграции — это система контроля версий для структуры БД**. Как Git отслеживает изменения кода, так миграции фиксируют изменения схемы базы данных. Это позволяет команде разработчиков синхронно обновлять БД и легко откатывать изменения.

3. **Подход Code First удобнее ручного SQL**. Вы описываете данные через классы C#, а EF Core сам создаёт и обновляет таблицы. Изменили модель → создали миграцию → применили к БД. Никаких ручных `ALTER TABLE`.

4. **`SaveChangesAsync()` — ключевой момент**. До вызова этого метода все изменения (добавление, обновление, удаление) существуют только в памяти контекста. Только `SaveChangesAsync()` фиксирует их в базе данных через транзакцию.

5. **`async/await` при работе с БД — обязательный стандарт**. Блокировать поток сервера на время ожидания ответа от базы данных — плохая практика. Асинхронные методы позволяют серверу обрабатывать другие запросы, повышая производительность приложения.

---

## 📚 Дополнительно: Что изучили в лабораторных

| Команда / Концепция | Описание |
|---------------------|----------|
| `dotnet add package` | Установить NuGet-пакет (аналог `npm install`) |
| `dotnet tool install --global dotnet-ef` | Установить CLI-инструмент для миграций |
| `dotnet ef migrations add <Name>` | Создать миграцию по изменениям в моделях |
| `dotnet ef database update` | Применить миграции к реальной БД |
| `dotnet ef migrations list` | Показать список миграций и их статус |
| `AppDbContext` | Главный класс EF Core — управление подключением к БД |
| `DbSet<T>` | Коллекция-таблица в коде C# |
| `HasData(...)` | Seed-данные, добавляемые при создании БД |
| `AsQueryable()` | Начать построение отложенного LINQ-запроса |
| `ToListAsync()` | Выполнить запрос и получить список результатов |
| `FindAsync(id)` | Быстрый поиск по первичному ключу (с кэшем) |
| `SaveChangesAsync()` | Зафиксировать все изменения в БД |
| `.Where(t => ...)` | Фильтрация → `WHERE` в SQL |
| `.OrderBy() / .OrderByDescending()` | Сортировка → `ORDER BY` в SQL |
| `.Skip(n).Take(m)` | Пагинация → `OFFSET n LIMIT m` в SQL |
| `.CountAsync(...)` | Подсчёт записей → `SELECT COUNT(*)` в SQL |
| `.GroupBy(...)` | Группировка → `GROUP BY` в SQL |
| `[Required]`, `[MaxLength]` | Валидация + ограничения в схеме БД |
| `DateTime?` (nullable) | Необязательное поле → `NULL` допустим в БД |
| **Code First** | Подход: сначала код C#, потом БД создаётся по нему |

---

