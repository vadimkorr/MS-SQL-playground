# Queries

## Table of contents
* [`SELECT`](#select)
* [`WHERE`](#where)
* [Операторы `IN` и `BETWEEN`](#in-between)
* [Оператор `LIKE`](#like-оператор)
* [`GROUP BY`](#group-by)
* [Агрегатные функции](#agregatnye-funkzii)
* [`HAVING`](#having)
* [`ORDER BY`](#order-by)
* [Extra](#extra)
* [Операторы работы с наборами](#operatori-raboti-s-naborami)
* [Выражение `CASE`](#virazhenie-case)
* [Подзапросы](#podzaprosi)
* [Временные таблицы](#vremennie-tablizi)


## [**`SELECT`**](#select)
```sql
--template
SELECT [ALL|DISTINCT] column_list
    [INTO new_table]
    FROM {table1 [tab_alias1]},...
    [WHERE search_condition]
    [GROUP BY group_by_expression]
    [HAVING search_condition]
    [ORDER BY order_expression [ASC | DESC]];
```
`DISTINCT` можно использовать лишь раз, должен предшествовать всем столбцам

## [**`WHERE`**](#where)
```sql
--script
USE sample;
SELECT dept_name, dept_no
    FROM department
    WHERE location='NY';
```

```sql
--script
USE sample;
SELECT emp_no, project_no
    FROM works_on
    WHERE project_no='p2'
    ~~WHERE job <> NULL~~ -- неправильная проверка на NULL
    AND job [NOT] IS NULL;
```

```sql
--script
USE sample;
SELECT emp_no, ISNULL(job, 'Job unknown') AS task
    FROM works_on
    WHERE project_no='p1';
```
Результат:
emp_no | task
------ | ----
10102 | Analyst
9031 | Manager
28559 |  Job unknown


```sql
--template
<> (!=)
<
>
>=
<=
!> -- не больше чем
!< -- не меньше чем
```

```sql
--template
AND
OR
NOT
```

## [**Операторы `IN` и `BETWEEN`**](#in-between)
```sql
--script
USE sample;
SELECT emp_no, emp_fname, emp_lname
    FROM employee
    WHERE emp_no [NOT] IN (293, 28559, 2534);
```

```sql
--script
USE sample;
SELECT project_name, budget
    FROM project
    WHERE budget BETWEEN 9000 AND 15000;
```
`BETWEEN` включает границы диапазона

## [**Оператор `LIKE`**](#like-оператор)

```sql
--template
% - последовательность любых символов любой длины
_ - любой один символ
```

```sql
--script
USE sample;
SELECT emp_fname, emp_lname, emp_no
    FROM employee
    WHERE emp_fname LIKE '_a%';
```

```sql
--script
USE sample;
SELECT dept_nt, dept_name, location
    FROM department
    WHERE location LIKE '[C-F]%'; -- начинается с символа от C до F
    -- WHERE location LIKE '[^C-F]%'; -- отрицание начала с символа от C до F
```

Любой подстановочный символ, заключенный в квадратные скобки, остается обычным символом и представляет сам себя, такаую же возможность предоставляет оператор `ESCAPE`.
```sql
WHERE project_name LIKE '%!_%' ESCAPE '!';
WHERE project_name LIKE '%[_]%';
```

## [**`GROUP BY`**](#group-by)
Каждый столбец в списке выборки запроса также должен присутствовать в предложениии `GROUP BY`.

```sql
--script
USE sample;
SELECT project_no, job
    FROM works_on
    GROUP BY project_no, job;
```

## [**Агрегатные функции**](#agregatnye-funkzii)
* обычные (`MIN`, `MAX`, `SUM`, `AVG`, `COUNT`, `COUNT_BIG`)
* статистичнские (`VAR`, `VARP`, `STDEV`, `STDEVP`)
* определяемые пользователем
* аналитические

```sql
--script
USE sample;
SELECT emp_no, emp_lname
    FROM employee
    WHERE emp_no = 
        (SELECT MIN(emp_no)
         FROM employee);
```

## [**`HAVING`**](#having)
Поределяется условие, которое применяется у группе строк
```sql
--script
USE sample;
SELECT project_no
    FROM works_on
    GROUP BY project_no
    HAVING COUNT(*) < 4;
```

```sql
--script
USE sample;
SELECT job
    FROM works_on
    GROUP BY job
    HAVING job LIKE 'M%';
```

## [**`ORDER BY`**](#order-by)
```sql
--script
USE sample;
SELECT emp_fname, emp_lname, dept_no
    FROM emplyee
    WHERE emp_no < 20000
    ORDER BY emp_lname, emp_fname;
```

```sql
--script
USE sample;
SELECT project_no, COUNT(*), emp_quantity
    FROM works_on
    GROUP BY project_no
    ORDER BY 2 DESC;
```
Разбиение на страницы:
```sql
--script
USE AdventureWorks2012;
SELECT BusinessEntityID, JobTitle, BirthDate
    FROM HumanResources.Employee
    WHERE Gender='F'
    ORDER BY JobTitle
    OFFSET 20 ROWS
    FETCH NEXT 10 ROWS ONLY;
```

## [**Extra**](#extra)
`IDENTITY`, `$identity`, `IDENT_SEED()`, `IDENT_INCR()`

`SEQUENCE`

## [**Операторы работы с наборами**](#operatori-raboti-s-naborami)

* `UNION` - объединяет результаты двух или более запросов в один результирующий набор, в который входят все строки. Можно объединять только совместимые таблицы (одинаковое число столбцов, совместимые типы данных). Результат объединения можно уплрядочить только с `ORDER BY`.
```sql
select_1 UNION [ALL] select_2 {[UNION [ALL] select_3]}...
```
* `INTERSECT` - пересенчение
```sql
--script
USE sample;
SELECT emp_no
    FROM employee
    WHERE dept_no='d1'
INTERSECT
SELECT emp_no
    FROM works_on
    WHERE enter_date < '01.01.2008';
```
* `EXCEPT` - разность

## [**Выражение `CASE`**](#virazhenie-case)
```sql
--script
USE sample;
SELECT project_name,
    CASE 
        WHEN budget > 0 AND budget < 10000 THEN 1
        WHEN budget >= 10000 AND budget < 20000 THEN 2
        WHEN budget >= 20000 AND budget < 30000 THEN 3
        ELSE 4
    END budget_weight
FROM project;
```

## [**Подзапросы**](#podzaprosi)
* независимые
* связанные


* операторами сравнения
* оператором `IN`
* операторами `ANY` и `ALL`

## [**Временные таблицы**](#vremennie-tablizi)
Объект БД, хранится и управляется системой БД на временной основе. Имена начинаются с префикса `#`. Временная таблица принадлежит создавшему ее сеансу, и видима только ему. Глобальные временные таблицы начинаются с префикса `##`.
```sql
--script
USE sample;
CREATE TABLE #project_temp
    (project_no CHAR(4) NOT NULL,
     project_name CHAR(25) NOT NULL);
```

```sql
--script
USE sample;
SELECT project_no, project_name
    INTO #project_temp1
    FROM project;
```






```sql
--template

```
```sql
--script

```
