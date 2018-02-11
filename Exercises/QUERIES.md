# Queries  (Tasks and solutions)

* 6.1 Выполните выборку всех строк из таблицы works_on.

```sql
SELECT emp_no, project_no, job, enter_date 
FROM works_on
```

* 6.2 Выполните выборку табельных номеров всех сотрудников с должностью клерк (`clerk`).

```sql
SELECT emp_no
FROM works_on
WHERE job = 'clerk';
```

* 6.3 Выполните выборку табельных номеров всех сотрудников, которые работают над проектом `p2` и чей табельный номер меньше, чем `10000`. Решите эту задачу, используя два различных, но эквивалентных запроса с инструкцией `SELECT`. 

```sql
SELECT DISTINCT emp_no
FROM works_on
WHERE project_no = 'p2' AND emp_no < 10000;

-- or

SELECT DISTINCT emp_no
FROM works_on
WHERE project_no = 'p2' AND emp_no BETWEEN 1 AND 9999;
```

* 6.4 Выполните выборку табельных номеров всех сотрудников, которые не приступили к работе над проектом в 2007 г. 

```sql
SELECT DISTINCT emp_no
FROM works_on
WHERE YEAR(enter_date) <> '2007';

-- or

SELECT DISTINCT emp_no
FROM works_on
WHERE enter_date NOT BETWEEN '01.01.2007' AND '12.31.2007';
```

* 6.5 Выполните выборку табельных номеров всех сотрудников проекта `p1` с ведущими должностями (т. е. аналитик — `analyst` и менеджер - `manager`).

```sql
SELECT emp_no
FROM works_on
WHERE project_no = 'p1' AND (job = 'analyst' OR job = 'manager')
```

* 6.6 Выполните выборку всех сотрудников проекта `p2`, чья должность еще не определена. 

```sql
SELECT emp_no
FROM works_on
WHERE job IS NULL AND project_no = 'p2'
```

* 6.7 Выполните выборку табельных номеров и фамилий сотрудников, чьи имена содержат две буквы "t". 

```sql
SELECT emp_no, emp_lname
FROM employee
WHERE emp_fname LIKE '%t%t%'
```

* 6.8 Выполните выборку табельных номеров и имен всех сотрудников, у которых вторая буква фамилии "o" или "a" (буквы английские) и последние буквы фамилии "es". 

```sql
SELECT emp_no, emp_lname
FROM employee
WHERE emp_fname LIKE '_[oa]%es'
```

* 6.9 Выполните выборку табельных номеров сотрудников, чьи отделы расположены в Сиэтле (Seattle). 

```sql
SELECT e.emp_no
FROM employee e, department d
WHERE e.dept_no = d.dept_no AND d.location = 'Seattle';

SELECT emp_no
FROM employee
WHERE dept_no = (
	SELECT dept_no
	FROM department
	WHERE location = 'Seattle');
```

* 6.10 Выполните выборку фамилий и имен сотрудников, которые приступили к работе над проектами 4 января 2007 г.

```sql
SELECT emp_fname, emp_lname
FROM employee e, works_on w
WHERE w.emp_no = e.emp_no AND w.enter_date = '01.04.2007';

-- or

SELECT emp_fname, emp_lname
FROM employee
WHERE emp_no IN (
	SELECT emp_no
	FROM works_on
	WHERE enter_date = '01.04.2007');
```

* 6.11 Сгруппируйте все отделы по их местонахождению. 

```sql
SELECT location
	FROM department
	GROUP BY location;
```

* 6.12 Объясните разницу между предложениями `DISTINCT` и `GROUP BY`. 

GROUP BY без дополнений (агрегаты, `HAVING`) - то же самое что и `DISTINCT`

* 6.13 Как предложение `GROUP BY` обрабатывает значения `NULL`? Подобна ли эта обработка обычной обработке этих значений?

Все `NULL` входят в одну группу

* 6.14 Объясните разницу между агрегатными функциями `COUNT(*)` и `COUNT(column)`.

`COUNT(expression)` - ожидает аргумент (столбец/выражеие) и показывает все не `NULL` вхождения аргумента
`COUNT(*)` - сичтает все строки (не важно `NULL` или нет).

* 6.15 Выполните выборку наибольшего табельного номера сотрудника. 

```sql
SELECT MAX(emp_no)
FROM employee
```

* 6.16 Выполните выборку должностей, занимаемых больше, чем двумя сотрудниками. 

```sql
SELECT job
FROM works_on
WHERE job -- IS NOT NULL
GROUP BY job
HAVING COUNT(*) > 2
```

* 6.17 Выполните выборку табельных номеров сотрудников, которые или имеют должность клерк `clerk`, или работают в отделе `d3`.

```sql
SELECT emp_no
FROM employee 
WHERE emp_no IN (SELECT emp_no
	FROM works_on
	WHERE job = 'clerk')
OR dept_no = 'd3'

-- or

SELECT DISTINCT emp_no
FROM works_on
WHERE Job = 'Clerk'
OR  emp_no IN (SELECT emp_no
	FROM employee
	WHERE dept_no='d3');
```

* 6.18 Объясните, почему следующий запрос неправильный.

```sql
SELECT project_name
FROM project
WHERE project_no = (SELECT project_no
	FROM works_on
	WHERE Job = 'Clerk');
```
Исправьте синтаксис запроса.

```sql
SELECT project_name
FROM project
WHERE project_no IN (SELECT project_no
	FROM works_on
	WHERE Job = 'Clerk');
```

* 6.19 Каково практическое применение временных таблиц? 

Могут быть использованы для хранения промежуточных результатов.

* 6.20 Объясните разницу между глобальными и локальными временными таблицами. 

Локальные таблицы - удаляются в конце текущей сессии.
Глобальные - удаляются в конце сессии, создавшей таблицу. 

* 6.21 Создайте следующие соединения таблиц `project` и `works_on`, выполнив:
	естественное соединение;
	декартово произведение.

```sql
SELECT *
FROM project p JOIN works_on w
ON p.project_no = w.project_no
```

```sql
SELECT *
FROM project CROSS JOIN works_on
``` 

* 6.22 Сколько условий соединения необходимо для соединения в запросе `n` таблиц? 

at least n-1

* 6.23 Выполните выборку табельного номера сотрудника и должности для всех сотрудников, работающих над проектом Gemini. 

```sql
SELECT emp_no, job
FROM works_on w, project p
WHERE w.project_no = p.project_no 
AND project_name = 'Gemini'

-- or

SELECT emp_no, job
FROM works_on w JOIN project p
ON w.project_no = p.project_no 
WHERE project_name = 'Gemini'
```

* 6.24 Выполните выборку имен и фамилий всех сотрудников, работающих в отделе `Research` или `Accounting`. 

```sql
SELECT emp_fname, emp_lname
FROM department d JOIN employee e
ON d.dept_no = e.dept_no
WHERE dept_name IN ('Research', 'Accounting') 
```

* 6.25 Выполните выборку всех дат начала работы для всех клерков (`clerk`), работающих в отделе `d1`. 

```sql
SELECT enter_date 
FROM employee e JOIN works_on w 
ON e.emp_no = w.emp_no
WHERE job = 'clerk' AND dept_no = 'd1'
```

* 6.26 Выполните выборку всех проектов, над которыми работают двое или больше сотрудников с должностью клерк (`clerk`).

```sql
SELECT p.project_no, COUNT(*) as 'clerks'
FROM project p JOIN works_on w
ON p.project_no = w.project_no
WHERE job = 'clerk'
GROUP BY w.job, p.project_no
HAVING COUNT(*) >= 2

-- or

SELECT project_name
FROM project
WHERE project_no IN (
	SELECT project_no
	FROM works_on
	WHERE job = 'clerk'
	GROUP BY project_no
	HAVING COUNT(*) >=2
)

```

* 6.27 Выполните выборку имен и фамилий сотрудников, которые имеют должность менеджер (`manager`) и работают над проектом `Mercury`.

```sql
SELECT emp_fname, emp_lname
FROM employee
WHERE emp_no IN (
	SELECT emp_no
	FROM works_on w JOIN project p
	ON w.project_no = p.project_no
	WHERE job = 'manager'
	AND p.project_name = 'Mercury');

-- or

SELECT emp_fname, emp_lname
FROM 
	employee e JOIN works_on w ON e.emp_no = w.emp_no
			   JOIN project p ON w.project_no = p.project_no
	WHERE job = 'manager'
	AND p.project_name = 'Mercury';
```

* 6.28 Выполните выборку имен и фамилий всех сотрудников, которые начали работать над проектом одновременно, по крайней мере, еще с одним другим сотрудником.

```sql
SELECT emp_fname, emp_lname
FROM 
	works_on w JOIN employee e
	ON w.emp_no = e.emp_no
GROUP BY enter_date, emp_fname, emp_lname
HAVING COUNT(*) > 1

-- or

SELECT emp_fname, emp_lname
	FROM employee
	WHERE emp_no IN
         (SELECT a.emp_no
		  FROM works_on a, works_on b
		  WHERE  b.enter_date=a.enter_date
		  AND a.emp_no != b.emp_no);
```

* 6.29 Выполните выборку табельных номеров сотрудников, которые живут в том же городе, где находится их отдел. (Используйте расширенную таблицу enployee_enh базы данных sample.)

* 6.30 Выполните выборку табельных номеров всех сотрудников, работающих в отделе маркетинга marketing. Создайте два равнозначных запроса, используя:
• оператор соединения;
• связанный подзапрос. 

```sql
-- JOIN
SELECT emp_no
FROM department d JOIN employee e
ON d.dept_no = e.dept_no
WHERE d.dept_name = 'marketing';

-- Subquery
SELECT emp_no
FROM employee e
WHERE dept_no IN (
	SELECT dept_no 
	FROM department
	WHERE dept_name = 'marketing');
```