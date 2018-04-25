# DML (Tasks and solutions)

* 7.1 Вставьте данные для новой сотрудницы по имени Julia Long, табельный номер 11111. Она еще не назначена в какой-либо отдел. 
```sql
INSERT INTO employee values(11111,'Julia','Long',NULL);
```

* 7.2 Создайте новую таблицу `emp_d1_d2` и загрузите в нее из таблицы employee всех сотрудников, работающих в отделах d1 и d2. Создайте два разных, но равнозначных решения. 

```sql
SELECT emp_no, emp_fname, emp_lname, dept_no
    INTO emp_d1_d2
    FROM employee
    WHERE dept_no IN ('d1', 'd2');
```

* 7.3 
Создайте новую таблицу для сотрудников, которые приступили к работе над своими проектами в 2007 г., и загрузите в нее соответствующие строки из таблицы `employee`. 

```sql
CREATE TABLE employee_three (emp_no INTEGER NOT NULL,
                    emp_fname CHAR(20) NOT NULL,
                    emp_lname CHAR(20) NOT NULL,
                    dept_no CHAR(4) NULL);
INSERT INTO employee_three(emp_no, emp_fname, emp_lname, dept_no)
               SELECT emp_no, emp_fname, emp_lname, dept_no
                       FROM employee
                       WHERE emp_no IN
                       (SELECT emp_no FROM works_on
                                 WHERE enter_date BETWEEN '01.01.2007' AND '12.31.2007');

```

* 7.4 
Измените должности всех менеджеров (Manager) в проекте p1 на клерков (Clerk).

```sql
UPDATE works_on
    SET job = 'Clerk'
    WHERE job = 'Manager' AND project_no = 'p1';

```

* 7.5 
Измените значение бюджетов всех проектов на значение `NULL`. 

```sql
UPDATE project SET budget = NULL;
```

* 7.6
Измените должность сотрудника с табельным номером 28559 на менеджера (Manager) для всех его проектов. 

```sql
UPDATE works_on
    SET job = 'Manager'
    WHERE emp_no = 28559;

```

* 7.7 
Повысьте на 10% бюджет проекта, менеджер которого имеет табельный номер 10102. 

```sql
UPDATE project
SET budget = budget/10+budget
WHERE project_no IN(
    SELECT project_no FROM works_on
    WHERE job='Manager' AND emp_no=10102);

```

* 7.8 
Измените наименование отдела, в котором работает сотрудник по фамилии James, на название Sales. 

```sql
UPDATE department
SET dept_name='Sales'
WHERE dept_no =
(SELECT dept_no FROM employee WHERE emp_lname='James');

```

* 7.9 
Измените дату начала работы над проектом для сотрудников, которые работают над проектом p1 и числятся в отделе Sales на 12 декабря 2007 г. 

```sql
UPDATE works_on
        SET enter_date='12.12.2007'
        WHERE project_no='p1' AND emp_no IN (
                    SELECT emp_no FROM employee
                    JOIN department ON employee.dept_no = department.dept_no
                    WHERE dept_name='Sales');

```

* 7.10 
Удалите все отделы, расположенные в Сиэтле (Seattle). 

```sql
DELETE FROM department
    WHERE location = 'Seattle';
```

* 7.11 
Проект p3 выполнен. Удалите всю информацию об этом проекте из базы данных sample. 

```sql
DELETE FROM works_on
    WHERE project_no = 'p3';
DELETE FROM project
    WHERE project_no = 'p3';

```

* 7.12 
Удалите всю информацию из таблицы works_on для всех сотрудников, которые работают в отделах, расположенных в Далласе (Dallas).

```sql
DELETE FROM works_on
WHERE emp_no IN (SELECT emp_no FROM employee
                WHERE dept_no IN (SELECT dept_no 
                FROM department WHERE location ='Dallas')); 

```
