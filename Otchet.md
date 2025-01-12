# Отчёт по итоговому заданию по дисциплине "Базы данных"

## 1. Структура базы данных

### Таблицы

**1. StudentGroups (Группы студентов)**

- group_id (INT, PRIMARY KEY, AUTO_INCREMENT) — уникальный идентификатор группы.
- group_name (VARCHAR(50), NOT NULL) — название группы.

**2. Students (Студенты)**

- student_id (INT, PRIMARY KEY, AUTO_INCREMENT) — уникальный идентификатор студента.
- first_name (VARCHAR(50), NOT NULL) — имя студента.
- last_name (VARCHAR(50), NOT NULL) — фамилия студента.
- date_of_birth (DATE) — дата рождения студента.
- contact_info (VARCHAR(100)) — контактная информация.
- group_id (INT, FOREIGN KEY) — идентификатор группы, ссылается на StudentGroups(group_id).

**3. Teachers (Преподаватели)**

- teacher_id (INT, PRIMARY KEY, AUTO_INCREMENT) — уникальный идентификатор преподавателя.
- first_name (VARCHAR(50), NOT NULL) — имя преподавателя.
- last_name (VARCHAR(50), NOT NULL) — фамилия преподавателя.
- date_of_birth (DATE) — дата рождения преподавателя.
- contact_info (VARCHAR(100)) — контактная информация.

**4. Subjects (Предметы)**

- subject_id (INT, PRIMARY KEY, AUTO_INCREMENT) — уникальный идентификатор предмета.
- subject_name (VARCHAR(100), NOT NULL) — название предмета.
- teacher_id (INT, FOREIGN KEY) — идентификатор преподавателя, ссылается на Teachers(teacher_id).

**5. Grades (Оценки)**

- grade_id (INT, PRIMARY KEY, AUTO_INCREMENT) — уникальный идентификатор оценки.
- student_id (INT, FOREIGN KEY) — идентификатор студента, ссылается на Students(student_id).
- subject_id (INT, FOREIGN KEY) — идентификатор предмета, ссылается на Subjects(subject_id).
- grade (INT, CHECK (grade BETWEEN 1 AND 5)) — оценка студента в диапазоне от 1 до 5.
- date (DATE) — дата выставления оценки.

---

## 2. Описание связей между таблицами

-Связь между Students и StudentGroups:

Студент принадлежит определённой группе.
Связь реализована через внешний ключ group_id в таблице Students.

-Связь между Subjects и Teachers:

Каждый предмет ведётся определённым преподавателем.
Связь реализована через внешний ключ teacher_id в таблице Subjects.

-Связь между Grades, Students и Subjects:

Оценка выставляется студенту по определённому предмету.
Связь реализована через внешние ключи student_id и subject_id в таблице Grades.

---

## 3. Описание ограничений и правил целостности

**Ограничения на атрибуты (целостность данных):**

- В таблице Grades, поле grade имеет ограничение CHECK (grade BETWEEN 1 AND 5), что гарантирует, что оценка всегда будет находиться в пределах от 1 до 5.
- Поля, такие как group_name в таблице StudentGroups и subject_name в таблице Subjects, имеют атрибут NOT NULL, чтобы предотвратить создание записей без названия группы или предмета.
- Поля first_name и last_name в таблицах Students и Teachers также являются NOT NULL, что исключает возможность создания записей без указания имени и фамилии.
- Для полей date_of_birth введён тип данных DATE, что позволяет вводить только корректные даты.

**Механизмы предотвращения дублирования данных:**

Уникальность записей обеспечивается использованием первичных ключей (PRIMARY KEY) в каждой таблице:
group_id в StudentGroups.
student_id в Students.
teacher_id в Teachers.
subject_id в Subjects.
grade_id в Grades.

Для предотвращения дублирования связей между студентами и группами или между предметами и преподавателями используются внешние ключи (FOREIGN KEY), которые контролируют соответствие значений в связанных таблицах.

---

## 4. Запросы к базе данных

Перед запросами необходимо наполнить базу данных тестовыми данными. 


1. Вывод списка студентов по определённому предмету:
```sql
SELECT 
    Students.first_name, Students.last_name
FROM 
    Students
JOIN 
    Grades ON Students.student_id = Grades.student_id
JOIN 
    Subjects ON Grades.subject_id = Subjects.subject_id
WHERE 
    Subjects.subject_name = 'Русский язык';

```
2. Возможность выводить список предметов, которые преподаёт конкретный преподаватель:
```sql
SELECT 
    Subjects.subject_name
FROM 
    Subjects
JOIN 
    Teachers ON Subjects.teacher_id = Teachers.teacher_id
WHERE 
    Teachers.first_name = 'Василий' AND Teachers.last_name = 'Сергеев';
```
3. Возможность выводить средний балл студента по всем предметам: 
```sql

SELECT 
    Students.first_name, Students.last_name, AVG(Grades.grade) AS average_grade
FROM 
    Students
JOIN 
    Grades ON Students.student_id = Grades.student_id
GROUP BY 
    Students.student_id;

```
4. Возможность выводить рейтинг преподавателей по средней оценке студентов:
```sql

SELECT 
    Teachers.first_name, Teachers.last_name, AVG(Grades.grade) AS average_grade
FROM 
    Teachers
JOIN 
    Subjects ON Teachers.teacher_id = Subjects.teacher_id
JOIN 
    Grades ON Subjects.subject_id = Grades.subject_id
GROUP BY 
    Teachers.teacher_id
ORDER BY 
    average_grade DESC;
```
5. Возможность выводить список преподавателей, которые преподавали более 3 предметов 
за последний год :
```sql
SELECT 
    Teachers.first_name, Teachers.last_name, COUNT(DISTINCT Subjects.subject_id) AS subject_count
FROM 
    Teachers
JOIN 
    Subjects ON Teachers.teacher_id = Subjects.teacher_id
JOIN 
    Grades ON Subjects.subject_id = Grades.subject_id
WHERE 
    Grades.date >= '2024-01-01' AND Grades.date <= '2024-12-31'
GROUP BY 
    Teachers.teacher_id
HAVING 
    subject_count > 3;

```
6. Возможность выводить список студентов, которые имеют средний балл выше 4 по
математическим предметам, но ниже 3 по гуманитарным:
```sql
WITH SubjectCategories AS (
    SELECT 
        Subjects.subject_id, 
        Subjects.subject_name, 
        CASE 
            WHEN Subjects.subject_name IN ('Математика', 'Физика', 'Информатика') THEN 'Математика'
            ELSE 'Гуманитарные'
        END AS category
    FROM 
        Subjects
), StudentAverages AS (
    SELECT 
        Students.student_id,
        Students.first_name,
        Students.last_name,
        AVG(CASE WHEN SubjectCategories.category = 'Математика' THEN Grades.grade END) AS math_avg,
        AVG(CASE WHEN SubjectCategories.category = 'Гуманитарные' THEN Grades.grade END) AS hum_avg
    FROM 
        Students
    JOIN 
        Grades ON Students.student_id = Grades.student_id
    JOIN 
        SubjectCategories ON Grades.subject_id = SubjectCategories.subject_id
    GROUP BY 
        Students.student_id
)
SELECT 
    first_name, last_name, math_avg, hum_avg
FROM 
    StudentAverages
WHERE 
    math_avg > 4 AND hum_avg < 3;

```
7. Возможность определить предметы, по которым больше всего двоек в текущем семестре:
```sql
SELECT 
    Subjects.subject_name, COUNT(*) AS double_count
FROM 
    Grades
JOIN 
    Subjects ON Grades.subject_id = Subjects.subject_id
WHERE 
    Grades.grade = 2 AND Grades.date >= '2024-01-01' AND Grades.date <= '2024-06-30'
GROUP BY 
    Subjects.subject_id
ORDER BY 
    double_count DESC;

```
8. Возможность выводить студентов, которые получили высший балл по всем своим
экзаменам, и преподавателей, которые вели эти предметы:
```sql
SELECT
    Students.first_name AS student_first_name,
    Students.last_name AS student_last_name,
    Teachers.first_name AS teacher_first_name,
    Teachers.last_name AS teacher_last_name,
    Subjects.subject_name
FROM
    Students
JOIN
    Grades ON Students.student_id = Grades.student_id
JOIN
    Subjects ON Grades.subject_id = Subjects.subject_id
JOIN
    Teachers ON Subjects.teacher_id = Teachers.teacher_id
WHERE
    Grades.grade = 5 -
    AND NOT EXISTS ( 
        SELECT 1
        FROM Grades AS InnerGrades
        WHERE InnerGrades.student_id = Students.student_id
        AND InnerGrades.grade < 5
    )
GROUP BY
    Students.student_id, Teachers.teacher_id, Subjects.subject_id
ORDER BY
    Students.last_name, Students.first_name, Subjects.subject_name;

```
9. Возможность просматривать изменение среднего балла студента по годам обучения:
```sql
-- студентов
SELECT 
    Students.first_name, Students.last_name, YEAR(Grades.date) AS year, AVG(Grades.grade) AS average_grade
FROM 
    Students
JOIN 
    Grades ON Students.student_id = Grades.student_id
GROUP BY 
    Students.student_id, YEAR(Grades.date)
ORDER BY 
    Students.student_id, year;

-- студента 
SELECT
    Students.first_name AS student_first_name,
    Students.last_name AS student_last_name,
    YEAR(Grades.date) AS year,
    AVG(Grades.grade) AS average_grade
FROM
    Students
JOIN
    Grades ON Students.student_id = Grades.student_id
WHERE
    Students.student_id = 1  -- Замените 1 на нужный ID студента
GROUP BY
    Students.student_id, YEAR(Grades.date)
ORDER BY
    year;

```
10. Возможность определить группы, в которых средний балл выше, чем в других, по
аналогичным предметам, чтобы выявить лучшие методики преподавания или особенности
состава группы:
```sql
-- по отдельному предмету
SELECT
    StudentGroups.group_name,
    AVG(Grades.grade) AS average_grade
FROM
    StudentGroups
JOIN
    Students ON StudentGroups.group_id = Students.group_id
JOIN
    Grades ON Students.student_id = Grades.student_id
JOIN
    Subjects ON Grades.subject_id = Subjects.subject_id
WHERE
    Subjects.subject_name = 'Русский язык'
GROUP BY
    StudentGroups.group_id
ORDER BY
    average_grade DESC;

-- или все предметы смотрим сразу
SELECT 
    StudentGroups.group_name, Subjects.subject_name, AVG(Grades.grade) AS average_grade
FROM 
    Students
JOIN 
    StudentGroups ON Students.group_id = StudentGroups.group_id
JOIN 
    Grades ON Students.student_id = Grades.student_id
JOIN 
    Subjects ON Grades.subject_id = Subjects.subject_id
GROUP BY 
    StudentGroups.group_id, Subjects.subject_id
ORDER BY 
    average_grade DESC;

```
11. Вставка записи о новом студенте с его личной информацией, такой как ФИО, дата
рождения, контактные данные и др.:
```sql

INSERT INTO Students (first_name, last_name, date_of_birth, contact_info, group_id) 
VALUES ('Иван', 'Петров', '2005-07-14', 'petrov@example.com', 2);

```
12. Обновление контактной информации преподавателя, например, электронной почты или
номера телефона, на основе его идентификационного номера или ФИО:
```sql
--для сравнения просим прошлуб инф-ю

SELECT
    Teachers.first_name,
    Teachers.last_name,
    Teachers.contact_info
FROM
    Teachers
WHERE
    Teachers.teacher_id = 1;

-- и обновляем ее

UPDATE 
    Teachers
SET 
    contact_info = 'vasya666@example.com' 
WHERE 
    teacher_id = 1; 

-- можно перепроверить, что все корректно обновилось
```
13. Удаление записи о предмете, который больше не преподают в учебном заведении.
Требуется также учесть возможные зависимости, такие как оценки студентов по этому
предмету:
```sql

ALTER TABLE Grades
ADD CONSTRAINT fk_subject
FOREIGN KEY (subject_id)
REFERENCES Subjects(subject_id)
ON DELETE CASCADE;

DELETE FROM Subjects WHERE subject_id = 9;

```
14. Вставка новой записи об оценке, выставленной студенту по определённому предмету, с
указанием даты, преподавателя и полученной оценки:
```sql
INSERT INTO Grades (student_id, subject_id, grade, date) 
VALUES (2, 1, 5, '2024-06-15');


```
