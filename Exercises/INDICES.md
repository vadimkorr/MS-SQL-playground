# Indices (Tasks and solutions)

* 1 Создайте некластеризованный индекс для столбца `enter_date` таблицы `works_on`, с заполнением пространства каждой страницы листьев индекса на 70%.

```sql
CREATE INDEX i_enterdate
     ON works_on(enter_date)
     WITH FILLFACTOR = 70;
```

* 2 Создайте однозначный составной индекс для столбцов `emp_lname` и `emp_fname` таблицы employee. Будет ли какая-либо разница, если изменить порядок столбцов в составном индексе?

```sql
CREATE UNIQUE INDEX i_lfname
    ON employee (emp_lname, emp_fname);
```
A composite index can be used for index access for the leading part of the index. Therefore, there is a significant difference if you change the order of the columns in a composite index.


* 3 Как удалить индекс, который был неявно создан для первичного ключа таблицы?

It can be dropped only if you drop the constraint (using the ALTER TABLE statement with the DROP CONSTRAINT clause).

* 4  Изложите достоинства и недостатки индексов.

During index access, only the rows that satisfy the search criteria of the query are accessed. This is in most cases an obvious advantage in relation to a table scan, where the system does not use an index. But besides this significant benefit, index scan can have two disadvantages: In contrast to a table scan, the Database Engine uses smaller I/O units to read rows for index access; therefore, a number of read operations will be comparatively higher. The second disadvantage of the index access method (using a nonclustered index) is that data pages must be read repeatedly, because the rows to be selected are scattered on data pages.

В следующих четырех упражнениях создайте индексы, которые повысят производительность запросов. Предполагается, что все таблицы базы данных sample, используемые в этих упражнениях, имеют большое количество строк.

* 5
```sql
SELECT emp_no, emp_fname, emp_lname
FROM employee
WHERE emp_lname = 'Smith'
```
```sql
CREATE INDEX i_employee_lname ON employee (emp_lname);
```

* 6
```sql
SELECT emp_no, emp_fname, emp_lname
FROM employee
WHERE emp_lname = 'Hansel'
AND emp_fname = 'Elke'
```
```sql
CREATE INDEX i_emp_name ON employee (emp_lname, emp_fname);
```

* 7
```sql
SELECT job
FROM works_on, employee
WHERE employee.emp_no = works_on.emp_no
```
```sql
CREATE INDEX i_workson_empno ON works_on (emp_no);
CREATE INDEX i_employee_empno ON employee (emp_no);
```

* 8
```sql
SELECT emp_lname, emp_fname
FROM employee, department
WHERE employee.dept_no = department.dept_no
AND dept_name = 'Research'
```
```sql
CREATE INDEX i_department_deptno ON department (dept_no);
CREATE INDEX i_employee_deptno ON employee (dept_no);
CREATE INDEX i_department_deptname ON department (dept_name);
```