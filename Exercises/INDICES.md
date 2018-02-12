# Indices (Tasks and solutions)

* 1 Создайте некластеризованный индекс для столбца `enter_date` таблицы `works_on`, с заполнением пространства каждой страницы листьев индекса на 70%.


* 2 Создайте однозначный составной индекс для столбцов `emp_lname` и `emp_fname` таблицы employee. Будет ли какая-либо разница, если изменить порядок столбцов в составном индексе?

* 3 Как удалить индекс, который был неявно создан для первичного ключа таблицы?

* 4  Изложите достоинства и недостатки индексов.

В следующих четырех упражнениях создайте индексы, которые повысят производительность запросов. Предполагается, что все таблицы базы данных sample, используемые в этих упражнениях, имеют большое количество строк.

* 5
```sql
SELECT emp_no, emp_fname, emp_lname
FROM employee
WHERE emp_lname = 'Smith'
```

* 6
```sql
SELECT emp_no, emp_fname, emp_lname
FROM employee
WHERE emp_lname = 'Hansel'
AND emp_fname = 'Elke'
```

* 7
```sql
SELECT job
FROM works_on, employee
WHERE employee.emp_no = works_on.emp_no
```

* 8
```sql
SELECT emp_lname, emp_fname
FROM employee, department
WHERE employee.dept_no = department.dept_no
AND dept_name = 'Research'
```