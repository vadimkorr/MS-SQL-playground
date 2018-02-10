# DML

## Table of contents

DML
* `SELECT`
* `INSERT`
* `UPDATE`
* `DELETE`
Эти инструкции оперируют либо твблицами, либо представлениями.

## **`INSERT`**
Вставляет строки (или части строк) в таблицу. Существует две разные формы.

* 1я форма - вставить в таблицу одну строку (или ее часть)
```sql
INSERT INTO employee VALUES (25348, 'Matthew', 'Smith', 'd3');
-- вставка в часть столбцов
INSERT INTO employee (emp_no, emp_fname, emp_lname)
    VALUES (15201, 'Dave', 'Davis');

-- вставка постредством конструктора значений
INSERT INTO department VALUES
    ('d4', 'Human Resources', 'Chicago'),
    ('d5', 'Distribution', 'New Orleans'),
    ('d6', 'Sales', 'Chicago');
```
* 2я форма -  вставить в таблицу результирующий набор иструкции `SELECT` или хранимой процедуры.
```sql
CREATE TABLE dallas_dept
    (dept_no CHAR(4) NOT NULL,
     dept_name CHAR(20) NOT NULL);
    
INSERT INTO dallas_dept (dept_no, dept_name)
    SELECT dept_no, dept_name
    FROM department
    WHERE location='Dallas';
```

## **`UPDATE`**
```sql
UPDATE works_on
    SET job='Manager'
    WHERE emo_no=18316
    AND project_no='p2';

-- wih case
UPDATE project 
    SET budget = CASE
        WHEN budget > 0 AND budget < 100000 THEN budget*1.2
        WHEN budget >= 100000 AND budget < 200000 THEN budget*1.1
        ELSE budget*1.05
    END
```

## **`DELETE`**
Две формы
```sql
DELETE FROM table_name
    [WHERE predicate];
--
DELETE table_name
    FROM table_name [,...n]
    [WHERE condition];
```

```sql
DELETE FROM works_on
    WHERE emp_no IN
    (SELECT emp_no
        FROM employee
        WHERE emp_lname='Moser');
    
DELETE FROM employee
    WHERE emp_lname='Moser';

-- or 

DELETE works_on
    FROM works_on, employee
    WHERE works_on.emp_no = emplyee.emp_no
    AND emp_lname = 'Moser';

DELETE FROM employee
    WHERE emp_lname='Moser';
```

## **`TRUNCATE TABLE`**
Более быстрая версия иструкции `DELETE` без предложения `WHERE`. 

```sql
TRUNCATE TABLE table_name
```

## **`MERGE`**
Объединяет последовательность иструкций `INSERT`, `UPDATE`, `DELETE` в одну элементарную иструкцию, в зависимости от существования строки.

Основная область применения - среда хранилищ данных, где таблицы необходимо периодически обновлять, чтобы отражать новые данные. 

```sql
CREATE TABLE bonus
    (pr_no CHAR(4), bonus SMALLINT DEFAULT 100);
INSERT INTO bonus (pr_no) VALUES ('p1');

MERGE INTO bonus B -- target
    USING (SELECT project_no, budget FROM project) E -- source
    ON (B.pr_no = E.project_no)
    WHEN MATCHED THEN 
        UPDATE SET B.bonus = E.budget * 0.1
    WHEN NOT MATCHED THEN
        INSERT (pr_no, bonus) 
        VALUES (E.project_no, E.budget * 0.05); 
```

## **`OUTPUT`**
По умолчанию единым видимым результатом выполнения иструкции `INSERT`, `UPDATE`, `DELETE` является только сообщение о количестве модифицированных строк. `OUTPUT` выводит модифицированные строки. 

```sql
DECLARE @del_table TABLE (emp_no INT, emp_lname CHAR(20));

DELETE employee 
OUTPUT DELETED.emp_no, DELETED.epm_lname INTO @del_table
WHERE emp_no > 15000;

SELECT * FROM @del_table
```

```sql
DECLARE @upd_table TABLE (
    emp_no INT, project_no CHAR(20), old_job CHAR(20), new_job CHAR(20)
);

UPDATE works_on 
SET job=NULL
OUTPUT DELETED.emp_no, DELETED.project_no, DELETED.job, INSERTED.job INTO @upd_table
WHERE job='Clerk';

SELECT * FROM @upd_table
```



