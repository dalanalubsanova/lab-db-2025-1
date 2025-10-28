# Лабораторная работа 1 (Проектирование логической и физической модели БД)

Перечень [лабораторных работ](https://edu.irnok.net/doku.php?
id=db:main#%D0%BB%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D0%B0%D1%8F_%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0_5_%D1%82%D1%80%D0%B8%D0%B3%D0%B3%D0%B5%D1%80%D1%8B)

Telegram: [at]dalannu

## Постановка задачи (вариант 38)

**Фитнес-трекер и планировщик тренировок**

*Сущности:* Упражнения (наименование, группа мышц, тип), тренировки (дата, продолжительность), пользователи (ID, ФИО, цель)

*Процессы:* Пользователи выполняют тренировки, состоящие из нескольких упражнений с определенным количеством подходов и повторений.

*Выходные документы:*
- Для заданного пользователя выдать прогресс по рабочему весу в определенном упражнении за последний месяц, отсортированный по дате.
- Выдать статистику по наиболее часто прорабатываемым группам мышц за указанный период, отсортированную по убыванию частоты.

## Промпт к DeepSeek

```text
Лаба по проектированию информационной модели для реляционных баз данных.
Предполагаем PostgreSQL.

Есть ошибки, замечания, неточности?

# Фитнес-трекер и планировщик тренировок

## Постановка задачи

*Сущности:*
- Упражнения (наименование, группа_мышц, тип)
- Тренировки (дата, продолжительность)
- Пользователи (ID, ФИО, цель)

*Процессы:*
Пользователи выполняют тренировки, состоящие из нескольких упражнений с определенным количеством подходов и повторений.

*Выходные документы:*
- Для заданного пользователя выдать прогресс по рабочему весу в определенном упражнении за последний месяц
- Статистика по наиболее часто прорабатываемым группам мышц за период

## ER-Модель

### Базовые сущности
- Пользователь(ФИО, цель)
- Упражнение(наименование, группа_мышц, тип)  
- Тренировка(дата, продолжительность)

### Отношения
[Пользователь]---N---выполняет---N---[Тренировка]

[Тренировка]---N---содержит---N---[Упражнение]
               | подходы, повторения, вес |

## Логическая модель

Получаем четыре таблицы:

- User(user_id, full_name, goal)
- Exercise(exercise_id, name, muscle_group, type)  
- Workout(workout_id, user_id, date, duration)
- WorkoutExercise(workout_id, exercise_id, sets, reps, weight)

## Физическая модель

Типы данных:

- user_id: SERIAL
- full_name: VARCHAR(100)
- goal: TEXT
- exercise_id: SERIAL  
- name: VARCHAR(100)
- muscle_group: VARCHAR(50)
- type: VARCHAR(50)
- workout_id: SERIAL
- date: DATE
- duration: INTEGER (в минутах)
- sets: INTEGER
- reps: INTEGER  
- weight: DECIMAL(5,2)

## Проверка нормальных форм

### Первая нормальная форма
Соблюдена - все значения атомарные

### Вторая нормальная форма  
Соблюдена - нет зависимостей от части ключа

### Третья нормальная форма
Соблюдена - нет транзитивных зависимостей
Ответ DeepSeek
Отличная работа! Модель в целом хорошо продумана, но есть несколько важных улучшений.

Критические замечания и рекомендации
1. Первичный ключ для WorkoutExercise
Проблема: В текущей модели нет явного первичного ключа для таблицы WorkoutExercise. Составной ключ (workout_id, exercise_id) может не подойти, если одно упражнение выполняется несколько раз в одной тренировке.

Исправление:

sql
WorkoutExercise(
    workout_exercise_id SERIAL PRIMARY KEY,
    workout_id INTEGER NOT NULL REFERENCES Workout(workout_id),
    exercise_id INTEGER NOT NULL REFERENCES Exercise(exercise_id), 
    sets INTEGER NOT NULL,
    reps INTEGER NOT NULL,
    weight DECIMAL(5,2)
);
2. Отслеживание прогресса весов
Проблема: Для выдачи прогресса по рабочему весу нужно хранить историю выполнения каждого упражнения с датами.

Решение: Текущая структура WorkoutExercise уже решает эту проблему, так как связана с тренировкой, у которой есть дата.

3. Типы данных
duration - лучше использовать INTERVAL или хранить в секундах (INTEGER)

date - в Workout лучше использовать TIMESTAMP для точного времени тренировки

weight - DECIMAL(5,2) подходит для большинства упражнений

Исправленная и улучшенная модель
Логическая модель
User (user_id, full_name, goal)

Exercise (exercise_id, name, muscle_group, type)

Workout (workout_id, user_id, workout_date, duration)

WorkoutExercise (workout_exercise_id, workout_id, exercise_id, sets, reps, weight)

Физическая модель (DDL для PostgreSQL)
sql
CREATE TABLE Users (
    user_id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    goal TEXT
);

CREATE TABLE Exercise (
    exercise_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    muscle_group VARCHAR(50) NOT NULL,
    type VARCHAR(50)
);

CREATE TABLE Workout (
    workout_id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES Users(user_id) ON DELETE CASCADE,
    workout_date TIMESTAMP NOT NULL,
    duration_minutes INTEGER
);

CREATE TABLE WorkoutExercise (
    workout_exercise_id SERIAL PRIMARY KEY,
    workout_id INTEGER NOT NULL REFERENCES Workout(workout_id) ON DELETE CASCADE,
    exercise_id INTEGER NOT NULL REFERENCES Exercise(exercise_id),
    sets INTEGER NOT NULL CHECK (sets > 0),
    reps INTEGER NOT NULL CHECK (reps > 0),
    weight DECIMAL(5,2) CHECK (weight >= 0)
);

-- Индексы для ускорения запросов
CREATE INDEX idx_workout_date ON Workout(workout_date);
CREATE INDEX idx_workout_user ON Workout(user_id);
CREATE INDEX idx_workoutexercise_workout ON WorkoutExercise(workout_id);
CREATE INDEX idx_exercise_muscle ON Exercise(muscle_group);
Примеры запросов для "Выходных документов"
1. Прогресс по рабочему весу для пользователя в упражнении за последний месяц:

sql
SELECT 
    w.workout_date,
    we.weight,
    we.sets,
    we.reps
FROM WorkoutExercise we
JOIN Workout w ON we.workout_id = w.workout_id
JOIN Exercise e ON we.exercise_id = e.exercise_id
WHERE w.user_id = 1  -- ID пользователя
  AND e.exercise_id = 5  -- ID упражнения
  AND w.workout_date >= CURRENT_DATE - INTERVAL '1 month'
ORDER BY w.workout_date;
2. Статистика по группам мышц за период:

sql
SELECT 
    e.muscle_group,
    COUNT(we.workout_exercise_id) as workout_count,
    COUNT(DISTINCT w.workout_id) as unique_workouts
FROM WorkoutExercise we
JOIN Workout w ON we.workout_id = w.workout_id  
JOIN Exercise e ON we.exercise_id = e.exercise_id
WHERE w.workout_date BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY e.muscle_group
ORDER BY workout_count DESC;
Промпт 2
text
В текущей модели для статистики по группам мышц нужно делать сложные JOIN через несколько таблиц.
Не проще ли добавить мышечную группу прямо в WorkoutExercise для денормализации и ускорения запросов?
Отличное предложение! Это разумный компромисс между нормализацией и производительностью.

Упрощенная модель с денормализацией
sql
CREATE TABLE WorkoutExercise (
    workout_exercise_id SERIAL PRIMARY KEY,
    workout_id INTEGER NOT NULL REFERENCES Workout(workout_id) ON DELETE CASCADE,
    exercise_id INTEGER NOT NULL REFERENCES Exercise(exercise_id),
    muscle_group VARCHAR(50) NOT NULL, -- Денормализованное поле
    sets INTEGER NOT NULL CHECK (sets > 0),
    reps INTEGER NOT NULL CHECK (reps > 0), 
    weight DECIMAL(5,2) CHECK (weight >= 0)
);
Преимущества денормализации:
Быстрее запросы - не нужны JOIN для статистики по группам мышц

Проще агрегации - прямое группирование по muscle_group

Историческая сохранность - если изменится группа мышц упражнения, старая статистика останется корректной

Упрощенный запрос для статистики:
sql
SELECT 
    muscle_group,
    COUNT(*) as exercise_count
FROM WorkoutExercise we
JOIN Workout w ON we.workout_id = w.workout_id
WHERE w.workout_date BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY muscle_group
ORDER BY exercise_count DESC;
Недостатки:
Избыточность данных - muscle_group хранится в двух местах

Возможная несогласованность - нужно следить за обновлением

Вывод: Для аналитических запросов по статистике денормализация оправдана.

Полученные диаграммы:
ER-диаграмма

erDiagram
    USERS ||--o{ WORKOUT : performs
    WORKOUT ||--o{ WORKOUT_EXERCISE : contains
    EXERCISE ||--o{ WORKOUT_EXERCISE : used_in
    
    USERS {
        integer user_id PK "SERIAL"
        varchar full_name
        text goal
    }
    
    EXERCISE {
        integer exercise_id PK "SERIAL" 
        varchar name
        varchar muscle_group
        varchar type
    }
    
    WORKOUT {
        integer workout_id PK "SERIAL"
        integer user_id FK
        timestamp workout_date
        integer duration_minutes
    }
    
    WORKOUT_EXERCISE {
        integer workout_exercise_id PK "SERIAL"
        integer workout_id FK
        integer exercise_id FK
        varchar muscle_group
        integer sets
        integer reps
        decimal weight
    }

Логическая модель в виде Диаграммы классов UML

classDiagram
    class Users {
        +user_id: Integer (PK)
        +full_name: String
        +goal: String
        +getWorkouts() List~Workout~
    }
    
    class Exercise {
        +exercise_id: Integer (PK)
        +name: String
        +muscle_group: String
        +type: String
        +getWorkoutExercises() List~WorkoutExercise~
    }
    
    class Workout {
        +workout_id: Integer (PK)
        +user_id: Integer (FK)
        +workout_date: DateTime
        +duration_minutes: Integer
        +getUser() Users
        +getExercises() List~WorkoutExercise~
    }
    
    class WorkoutExercise {
        +workout_exercise_id: Integer (PK)
        +workout_id: Integer (FK)
        +exercise_id: Integer (FK)
        +muscle_group: String
        +sets: Integer
        +reps: Integer
        +weight: Decimal
        +getWorkout() Workout
        +getExercise() Exercise
    }
    
    Users "1" -- "*" Workout : performs
    Workout "1" -- "*" WorkoutExercise : contains
    Exercise "1" -- "*" WorkoutExercise : used_in

Физическая модель БД

erDiagram
    users {
        integer user_id PK "SERIAL"
        varchar full_name "NOT NULL"
        text goal
    }
    
    exercise {
        integer exercise_id PK "SERIAL"
        varchar name "NOT NULL"
        varchar muscle_group "NOT NULL"
        varchar type
    }
    
    workout {
        integer workout_id PK "SERIAL"
        integer user_id FK "NOT NULL"
        timestamp workout_date "NOT NULL"
        integer duration_minutes
    }
    
    workout_exercise {
        integer workout_exercise_id PK "SERIAL"
        integer workout_id FK "NOT NULL"
        integer exercise_id FK "NOT NULL"
        varchar muscle_group "NOT NULL"
        integer sets "NOT NULL"
        integer reps "NOT NULL"
        decimal weight
    }
    
    users ||--o{ workout : "FOREIGN KEY (user_id) REFERENCES users(user_id)"
    workout ||--o{ workout_exercise : "FOREIGN KEY (workout_id) REFERENCES workout(workout_id)"
    exercise ||--o{ workout_exercise : "FOREIGN KEY (exercise_id) REFERENCES exercise(exercise_id)"
    
Заключение
В первой лабораторной работе при помощи теоретического инструментария проектирования БД и языковой модели DeepSeek произведено проектирование ER-, логической и физической модели базы данных варианта 38. Основные решения и улучшения:

Реализовано две итерации консультации с ИИ-ассистентом

Введен суррогатный ключ для связи многие-ко-многим (WorkoutExercise)

Применена денормализация для ускорения аналитических запросов (muscle_group в WorkoutExercise)

Добавлены проверочные ограничения (CHECK) для целостности данных

Созданы индексы для оптимизации частых запросов

Учтена временная составляющая для анализа прогресса

Модель удовлетворяет требованиям нормальных форм и эффективно решает поставленные задачи по трекингу прогресса и статистике тренировок.
