# Join

Несколько различных форм:
* естественное соединение 
* декартово произведение
* внешнее соединение
* тета-соединение, самомоединение, полусоединение

Две синтаксические формы оператора соединения
* явный синтаксис (SQL92)
* неявный (старого стиля)

При явном используются ключевые слова
* `CROSS JOIN` - декартово соединение
* `[INNER] JOIN` - естественное соединение
* `LEFT [OUTER] JOIN`
* `RIGHT [OUTER] JOIN`
* `FULL [OUTER] JOIN` - соединение правого и левого внешнего соединений

Естественное соединение
* естественное соединение (natural join) - всегда имеет одну или несколько пар столбцов с идентичными значениями в каждой строке
* соединение по эквивалентности (equi-join) - устраняет такие столбцы из результатов операции соединения 

```sql
--script
USE sample;
SELECT employee.*, department.*
    FROM employee INNER JOIN department
    ON employee.dept_no = department.dept_no;
```

## Соединение более 2х таблиц
```sql
/* Выборка наименований проектов (с удалением избыточных дубликатов), в которых участвуют сотрудники бухгалтерии
*/
--script
USE sample;
SELECT DISTINCT project_name
    FROM project JOIN works_on
    ON project.project_no = works_on.project_no
        JOIN employee ON works_on.emp_no = employee.emp_no
        JOIN department ON employee.dept_no = department.dept_no
    WHERE dept_name = 'Accounting';
```

## Декартово произведение
```sql
--script
USE sample;
SELECT employee.*, department.*
    FROM employee CROSS JOIN department;
```

## Внешнее соединение

Иногда кроме совпадающих строк бывает необходимым извлечь из одной или обеих таблиц строки без совпадений. 

```sql
--template
-- сравни
USE sample;
SELECT employee_enh.*, department.location
    FROM employee_enh JOIN department
        ON domicile = location;

SELECT employee_enh.*, department.location
    FROM employee_enh LEFT OUTER JOIN department
        ON domicile = location;
```
Эмуляция левого внешнего соединения с помощью UNION и NOT EXIST. (стр 200)

## Тета-соединение

## Самосоединение
```sql
--script
USE sample;
SELECT t1.dept_no, t1.dept_name, t1.location
    FROM department t1 JOIN department t2
    ON t1.location = t2.location
    WHERE t1.dept_no <> t2.dept_no;
```

## Полусоединение
Похоже на естественное соединение, но взвращает только набор всех строк из одной таблицы, для которой в другой таблице есть одно или несколько совпадений. Еще одна особенность: список выбора SELECT содержит толлько столбцы из таблицы employee.
```sql
--script
USE sample;
SELECT emp_no, emp_lname, e.dept_no
    FROM employee e JOIN department d
    ON e.dept_no = d.dept_no
    WHERE location = 'NY';
```

## Подзапросы, функция `EXIST`
* \+'s: подзапросов
* \+'s: соединений

## Табличные выражения
\- подзапросы, которые используются там, где ожидается наличие таблицы. 
* производные таблицы
* обобщенные табличные выражения

## Производные таблицы
\- derived table, табличное выражение, входящее в предложение FROM запроса. 

```sql
--script
USE sample;
SELECT MONTH(enter_data) as enter_month
    FROM works_on
    GROUP BY enter_month;
/* ошибка выполнения: предложение GROUP BY обрабатывается до обработки соответствующего списка инструкции SELECT, и при обработке этой группы псевдоним столбца enter_month неизвестен. */
```

```sql
--script
USE sample;
SELECT enter_month
    FROM (SELECT MONTH(enter_date) as enter_month
          FROM works_on) AS m
    GROUP BY enter_month;
```

## Обощенные табличные выражения
\- common table expression (CTE) - именованное табличное выражение, поддерживаемое языком T-SQL. Используются в следующих типах запросов:
* нерекурсивных
* рекурсивных

### ОТВ и нерекурсивные запросы
```sql
--template
WITH cte_name (column_list) AS
(inner_query)
outer_query
```

```sql
--script
USE AdventureWOrks;
WITH price_calc (year_2002) AS 
    (SELECT AVG(TotalDue)
        FROM Sales.SalesOrderHeader
        WHERE YEAR(OrderDate) = '2002')
SELECT SalesOrderID
    FROM Sales.SalesOrderHeader
    WHERE TotalDue < (SELECT year_2002 FROM price_cals)
    AND Freight > (SELECT year_2002 FROM price_calc)/2.5;
```

### ОТВ и рекурсивные запросы

```sql
--template
WITH cte_name (column_list) AS 
    (anchor_member
     UNION ALL
     recursive_member)
outer_query
```

```sql
--script
USE sample;
WITH list_of_parts (assembly, quantity, cost) AS
    (SELECT containing_assembly, quantity_contained, unit_cost
     FROM airplane
     WHERE contained_assembly IS NULL)
UNION ALL
SELECT a.containing_assembly, a.quantity_contained, CAST (l.quantity*l.cost AS DECIMAL(6,2)
    FROM list_of_parts l, airplane a
    WHERE l.assembly = a.contained_assembly)
SELECT assembly, SUM(quantity) parts, SUM(cost) sum_cost
    FROM list_of_parts
    GROUP BY assembly;
```




```sql
--template

```

```sql
--script

```