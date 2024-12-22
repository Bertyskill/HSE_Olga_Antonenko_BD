# Отчёт по итоговому заданию по дисциплине "Базы данных"

## 1. Структура базы данных

### Таблицы

1. **Students (Студенты)**:
   - `student_id` (INT, PRIMARY KEY) — уникальный идентификатор студента.
   - `first_name` (VARCHAR) — имя студента.
   - `last_name` (VARCHAR) — фамилия студента.
   - `date_of_birth` (DATE) — дата рождения студента.
   - `contact_info` (VARCHAR) — контактная информация.
   - `group_id` (INT, FOREIGN KEY) — идентификатор группы.

2. **Teachers (Преподаватели)**:
   - `teacher_id` (INT, PRIMARY KEY) — уникальный идентификатор преподавателя.
   - `first_name` (VARCHAR) — имя преподавателя.
   - `last_name` (VARCHAR) — фамилия преподавателя.
   - `date_of_birth` (DATE) — дата рождения преподавателя.
   - `contact_info` (VARCHAR) — контактная информация.

3. **Subjects (Предметы)**:
   - `subject_id` (INT, PRIMARY KEY) — уникальный идентификатор предмета.
   - `subject_name` (VARCHAR) — название предмета.
   - `teacher_id` (INT, FOREIGN KEY) — идентификатор преподавателя.

4. **Grades (Оценки)**:
   - `grade_id` (INT, PRIMARY KEY) — уникальный идентификатор оценки.
   - `student_id` (INT, FOREIGN KEY) — идентификатор студента.
   - `subject_id` (INT, FOREIGN KEY) — идентификатор предмета.
   - `grade` (INT) — оценка студента.
   - `date` (DATE) — дата выставления оценки.

5. **Groups (Группы)**:
   - `group_id` (INT, PRIMARY KEY) — уникальный идентификатор группы.
   - `group_name` (VARCHAR) — название группы.

---

## 2. Описание связей между таблицами

- **Связь между `Students` и `Groups`**:
  - Студент принадлежит определённой группе.
  - Связь реализована через внешний ключ `group_id` в таблице `Students`.

- **Связь между `Subjects` и `Teachers`**:
  - Каждый предмет ведётся определённым преподавателем.
  - Связь реализована через внешний ключ `teacher_id` в таблице `Subjects`.

- **Связь между `Grades`, `Students` и `Subjects`**:
  - Оценка выставляется студенту по определённому предмету.
  - Связь реализована через внешние ключи `student_id` и `subject_id` в таблице `Grades`.

---

## 3. Описание ограничений и правил целостности

- **Ограничения на атрибуты**:
  - В таблице `Grades` оценка (`grade`) должна быть в диапазоне от 1 до 5:
    ```sql
    ALTER TABLE Grades
    ADD CONSTRAINT chk_grade CHECK (grade BETWEEN 1 AND 5);
    ```

- **Уникальные ключи**:
  - В таблице `Students` поле `student_id` является уникальным.
  - В таблице `Teachers` поле `teacher_id` является уникальным.

- **Внешние ключи**:
  - В таблице `Grades` используются внешние ключи для связи с таблицами `Students` и `Subjects`.
  - В таблице `Subjects` используется внешний ключ для связи с таблицей `Teachers`.

---


## 4. Запросы к базе данных

1. Вывод списка студентов по определённому предмету:
```sql
SELECT s.first_name, s.last_name
FROM Students s
JOIN Grades g ON s.student_id = g.student_id
JOIN Subjects sub ON g.subject_id = sub.subject_id
WHERE sub.subject_name = 'Математика';
```
2. Возможность выводить список предметов, которые преподаёт конкретный преподаватель:
```sql
SELECT sub.subject_name
FROM Subjects sub
JOIN Teachers t ON sub.teacher_id = t.teacher_id
WHERE t.first_name = 'Иван' AND t.last_name = 'Иванов';
```
3. Возможность выводить средний балл студента по всем предметам: 
```sql
SELECT s.first_name, s.last_name, AVG(g.grade) AS average_grade
FROM Students s
JOIN Grades g ON s.student_id = g.student_id
GROUP BY s.student_id;
```
4. Возможность выводить рейтинг преподавателей по средней оценке студентов:
```sql
SELECT t.first_name, t.last_name, AVG(g.grade) AS average_grade
FROM Teachers t
JOIN Subjects sub ON t.teacher_id = sub.teacher_id
JOIN Grades g ON sub.subject_id = g.subject_id
GROUP BY t.teacher_id
ORDER BY average_grade DESC;
```
5. Возможность выводить список преподавателей, которые преподавали более 3 предметов 
за последний год :
```sql
SELECT t.first_name, t.last_name
FROM Teachers t
JOIN Subjects sub ON t.teacher_id = sub.teacher_id
JOIN Grades g ON sub.subject_id = g.subject_id
WHERE g.date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY t.teacher_id
HAVING COUNT(DISTINCT sub.subject_id) > 3;
```
6. Возможность выводить список студентов, которые имеют средний балл выше 4 по
математическим предметам, но ниже 3 по гуманитарным:
```sql
WITH MathGrades AS (
SELECT s.student_id, AVG(g.grade) AS math_avg
FROM Students s
JOIN Grades g ON s.student_id = g.student_id
JOIN Subjects sub ON g.subject_id = sub.subject_id
WHERE sub.subject_name LIKE '%Математика%'
GROUP BY s.student_id
),
HumanitiesGrades AS (
SELECT s.student_id, AVG(g.grade) AS hum_avg
FROM Students s
JOIN Grades g ON s.student_id = g.student_id
JOIN Subjects sub ON g.subject_id = sub.subject_id
WHERE sub.subject_name LIKE '%Гуманитарный%'
GROUP BY s.student_id
)
SELECT s.first_name, s.last_name
FROM Students s
JOIN MathGrades mg ON s.student_id = mg.student_id
JOIN HumanitiesGrades hg ON s.student_id = hg.student_id
WHERE mg.math_avg > 4 AND hg.hum_avg < 3;
```
7. Возможность определить предметы, по которым больше всего двоек в текущем семестре:
```sql
SELECT sub.subject_name, COUNT(*) AS num_of_twos
FROM Grades g
JOIN Subjects sub ON g.subject_id = sub.subject_id
WHERE g.grade = 2
GROUP BY sub.subject_id
ORDER BY num_of_twos DESC
LIMIT 1;
```
8. Возможность выводить студентов, которые получили высший балл по всем своим
экзаменам, и преподавателей, которые вели эти предметы:
```sql
SELECT s.first_name, s.last_name, t.first_name AS teacher_first_name, t.last_name AS teacher_last_name
FROM Students s
JOIN Grades g ON s.student_id = g.student_id
JOIN Subjects sub ON g.subject_id = sub.subject_id
JOIN Teachers t ON sub.teacher_id = t.teacher_id
WHERE g.grade = 5;
```
9. Возможность просматривать изменение среднего балла студента по годам обучения:
```sql
SELECT YEAR(g.date) AS year, AVG(g.grade) AS average_grade
FROM Grades g
WHERE g.student_id = 1
GROUP BY YEAR(g.date);
```
10. Возможность определить группы, в которых средний балл выше, чем в других, по
аналогичным предметам, чтобы выявить лучшие методики преподавания или особенности
состава группы:
```sql
SELECT g.group_name, AVG(gr.grade) AS average_grade
FROM Groups g
JOIN Students s ON g.group_id = s.group_id
JOIN Grades gr ON s.student_id = gr.student_id
JOIN Subjects sub ON gr.subject_id = sub.subject_id
WHERE sub.subject_name = 'Математика'
GROUP BY g.group_id
ORDER BY average_grade DESC;
```
11. Вставка записи о новом студенте с его личной информацией, такой как ФИО, дата
рождения, контактные данные и др.:
```sql
INSERT INTO Students (first_name, last_name, date_of_birth, contact_info, group_id)
VALUES ('Петр', 'Петров', '2000-01-01', 'petr@example.com', 1);
```
12. Обновление контактной информации преподавателя, например, электронной почты или
номера телефона, на основе его идентификационного номера или ФИО:
```sql
UPDATE Teachers
SET contact_info = 'new_email@example.com'
WHERE teacher_id = 1;
```
13. Удаление записи о предмете, который больше не преподают в учебном заведении.
Требуется также учесть возможные зависимости, такие как оценки студентов по этому
предмету:
```sql
ALTER TABLE Grades
ADD CONSTRAINT fk_subject_id
FOREIGN KEY (subject_id) REFERENCES Subjects(subject_id)
ON DELETE CASCADE;
```
14. Вставка новой записи об оценке, выставленной студенту по определённому предмету, с
указанием даты, преподавателя и полученной оценки:
```sql
INSERT INTO Grades (student_id, subject_id, grade, date)
VALUES (1, 2, 5, '2023-10-01');
```
