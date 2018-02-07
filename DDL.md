# DDL

## Содержание
* [Creation of a DB](#creation-of-a-db)
* [Creation of a DB snapshot](#creation-of-a-db-snapshot)
* [Attaching and Detaching Databases](#attaching-and-detaching-databases)
* [`CREATE TABLE` a basic form](#create-table-a-basic-form)
* [`CREATE TABLE` and declarative integrity constraints](#create-table-and-declarative-integrity-constraints)
* [`UNIQUE` clause](#unique-clause)
* [`CHECK` clause](#check-clause)
* [`FOREIGN KEY` clause](#foreign-key-clause)
* [Referential Integrity](#referential-integrity)
* [`ON DELETE` and `ON UPDATE` options](#on-delete-and-on-update-options)
* [Creating other DB objects](#creating-other-db-objects)
* [Integrity constraints and domains](#integrity-constraints-and-domains)
* [Modifying DB objects](#modifying-db-objects)
* [Altering a table](#altering-a-table)
* [Renaming tables and other DB objects](#renaming-tables-and-other-db-objects)
* [Removing DB objects](#removing-db-objects)

## **Creation of a DB**
```sql
--template
CREATE DATABASE db_name
    [ON [PRIMARY] {file_spec1},. ]
    [LOG ON {file_spec2},. ]
    [COLLATE collation_name]
    [FOR {ATTACH ATTACH_REBUILD_LOG}]

-- [] необязательный
-- {} повторяемый
```
* `db_name` - имя БД
* `ON` - все файлы БД указываются явно
* `file_spec1` - представляет спецификацию файла и сам может содержать дополнительные опции (логическое имя файла, физическое имя и размер)
* `PRIMARY` - указывает первый и наиболее важный файл, который содержит системные таблицы
* `LOG ON` - определяет один или более файлов в качестве физического хранилища журнала транзакций
* `COLLATE` - порядок сортировки по умолчанию для БД
* `FOR ATTACH` - указывается, что БД создается за счет подключения существующего набора файлов
* `FOR ATTACH_REBUILD_LOG` - указывается, что БД создается методом присоединения существующего набора файлов ОС

```sql
--script
USE master;
CREATE DATABASE projects
    ON (NAME=projects_dat,
        FILENAME='C:\projects.mdf',
        SIZE=10,
        MAXSIZE=100,
        FILEGROWTH=5)
LOG ON
    (NAME=projects_log,
    FILENAME='C:\projects.ldf',
    SIZE=40,
    MAXSIZE=100,
    FILEGROWTH=10);
```
Созданная БД называется `projects`. Т.к. опция `PRIMARY` не указана, но первичнвм файлом предполагается первый файл. Этот файл имеет логическое имя `projects_dat` и он сохраняется в дисковом файле `projects.mdf`. Исходный размер этого файла 10Мб с приращением при необходимости в 5Мб. Если не указать опцию `MAXSIZE` или этой если присвоить значение `UNLIMITED`, то максимальный размер файла может увеличиваться и будет ограничивать ся только размером дискового пространства (`MB`, `KB`, `TB`). Также создается файл журнала транзакций.

## **Creation of a DB snapshot**
\- согласованная с точки зрения завершенных транзакций копия исходной БД на момент создания снимка доступная только для чтения (необходим NTFS)

```sql
--template
CREATE DATABASE database_snapshot_name
    ON (NAME=logical_file_name,
        FILENAME='os_file_name',) [,..n]
    AS SNAPSHOT OF source_database_name
```

```sql
--script
USE master;
CREATE DATABASE AdventureWorks_snapshot
    ON (NAME='AdventureWorks_Data',
        FILENAME='C:\temp\snapshot_db.mdf')
    AS SNAPSHOT OF AdventureWorks;
```
## **Attaching and Detaching Databases**
Используется при перемещении БД. 

*Отсоединение:* системная функция `sp_detach_db` (ДБ должна находиться в однопользовательском режиме)

*Присоединение:* `CREATE DATABASE` с `FOR ATTACH`

## **`CREATE TABLE` a basic form**
```sql
--template
CREATE TABLE table_name
    (col_name1 type1 [NOT NULL | NULL]
    [{, col_name2 type2 [NOT NULL | NULL]} ... ])
```

```sql
--script
USE sample;
CREATE TABLE employee (emp_no INTEGER NOT NULL,
                       emp_fname CHAR(20) NOT NULL,
                       emp_lname CHAR(20) NOT NULL,
                       dept_no CHAR(4) NULL);

CREATE TABLE department(dept_no CHAR(4) NOT NULL,
                       dept_name CHAR(25) NOT NULL,
                       location CHAR(30) NULL);

CREATE TABLE project  (project_no CHAR(4) NOT NULL,
                       project_name CHAR(15) NOT NULL,
                       budget FLOAT NULL);

CREATE TABLE works_on (emp_no INTEGER NOT NULL,
                       project_no CHAR(4) NOT NULL,
                       job CHAR(15) NULL,
                       enter_date DATE NULL);
```

Спецификация столбца также может иметь следующие параметры:
* `DEFAULT` - указывает значение столбца по умолчанию (также можно указать одну из системных функций: `USER`, `CURRENT_USER`, `SESSION_USER`, `SYSTEM_USER`, `CURRENT_TIMESTAMP`, `NULL`)
* `IDENTITY` - столбец иднтификаторов может иметь только целочисленные значения, начальное значение и шаг инкремента

## **`CREATE TABLE` and declarative integrity constraints**
Ограничения, которые используются для проверки данных при модификации или вставке (integrity constraints). Декларативные (`CREATE TABLE`, `ALTER TABLE`) и при помощи триггеров. Ограничения уровня столбцов, уровня таблицы. 

Декларативные ограничения можно сгруппировать:
* `DEFAULT`
* `UNIQUE`
* `PRIMARY KEY`
* `CHECK`
* `FOREIGN KEY` ссылочная целостность

## **`UNIQUE` clause**
Иногда несколько столбцов таблицы имеют уникальные значения, что позваоляет их использовать в качестве первичного ключа (candidate key). `CREATE TABLE` или `ALTER TABLE`.

```sql
--template
[CONSTRAINT c_name]
    UNIQUE [CLUSTERED | NONCLUSTERED] ({col_name1},...)
```

```sql
--script
USE sample;
CREATE TABLE projects (project_no CHAR(4) DEFAULT 'p1',
                       project_name CHAR(15) NOT NULL,
                       budget FLOAT NULL
                       CONSTRAINT unique_no UNIQUE (project_no));
```

## **`PRIMARY KEY` clause**
Первичным ключом является столбец или группа столбцов, значения которого разные в каждой строке. `CREATE TABLE` или `ALTER TABLE`.
```sql
--template
[CONSTRAINT c_name]
    PRIMARY KEY [CLUSTERED | NONCLUSTERED] ({col_name1},...)
```

```sql
--script
USE sample;
CREATE TABLE employee (emp_no INTEGER NOT NULL,
                       emp_fname CHAR(20) NOT NULL,
                       emp_lname CHAR(20) NOT NULL,
                       dept_no CHAR(4) NULL
                       CONSTRAINT prim_empl PRIMARY KEY (emp_no));
```

Ограничение уровня столбца
```sql
--script
USE sample;
CREATE TABLE employee (emp_no INTEGER NOT NULL CONSTRAINT prim_empl PRIMARY KEY,
                       emp_fname CHAR(20) NOT NULL,
                       emp_lname CHAR(20) NOT NULL,
                       dept_no CHAR(4) NULL);
```

## **`CHECK` clause**
`CREATE TABLE` или `ALTER TABLE`.
```sql
--template
[CONSTRAINT c_name]
    CHECK [NOT FOR REPLICATION] expression
```
Параметр `expression` должен быть логическим выражением. Не применяется принудительно при репликации данных, если присутсвует параметр `NOT FOR REPLICATION`.

```sql
--script
USE sample;
CREATE TABLE customer (cust_no INTEGER NOT NULL,
                       cust_group CHAR(3) NULL,
                       CHECK (cust_group IN ('c1', 'c2', 'c10')));
```

## **`FOREIGN KEY` clause**
Внешний ключ (foreign key) - столбец (или группа столбцов) содержащий значения, совпадающие со значениями первичного ключа другой таблицы. Внешний ключ определяется с помощью предложения `FOREIGN KEY` в комбинации с предложением `REFERENCES`.

```sql
--template
[CONSTRAINT c_name]
    [[FOREIGN KEY] ({col_name1},.)]
    REFERENCES table_name ({col_name2},.)
        [ON DELETE (NO ACTION | CASCADE | SET NULL | SET DEFAULT)]
        [ON UPDATE (NO ACTION | CASCADE | SET NULL | SET DEFAULT)]
```

```sql
--script
CREATE TABLE works_on (emp_no INTEGER NOT NULL,
                       project_no CHAR(4) NOT NULL,
                       job CHAR(15) NULL,
                       enter_date DATE NULL,
                        CONSTRAINT prim_works PRIMARY KEY(emp_no, project_no),
                        CONSTRAINT foreign_works FOREIGN KEY(emp_no)
                            REFERENCES employee (emp_no));
```

## **Referential integrity**
(referencial integrity) обеспечивает выполнение правил для вставок и обновлений таблиц, содержащих ключ и соответсвующее ограничение первичного ключа.

## [**Возможные проблемы со ссылочной целостностью**](#a-возможные-проблемы-со-ссылочной-целостностью)
Модификация значений внешнего или первичного или первичного ключа может создавать проблемы в 4х случаях. 

### Случай 1 (модификация ссылающейся таблицы)
```sql
--script
USE sample;
INSERT INTO works_on (emp_no, ...)
    VALUES (11111,...);
```
При вставке новой строки в дочернюю таблицу `works_on` используется номер сотрудника `emp_no`, для которого нет совпадающего сотрудника (и номера) в родительской таблице `employee`. Если для обеих таблиц определена ссылочная целостность, то компоненто Database Engine не допустит такой вставки. 
### Случай 2 (модификация ссылающейся таблицы)
```sql
--script
USE sample;
UPDATE works_on
    SET emp_no=11111 WHERE emp_no=10102;
```
Также как в случае 1. 
### Случай 3 (модификация родительской таблицы)
```sql
--script
USE sample;
UPDATE employee
    SET emp_no=22222 WHERE emp_no=10102;
```
Предпринимается попытка заменить существующее значение 10102 номера сотрудника `emp_no` значением 22222 только в родительской таблице `employee`, не меняя соответсвующие значения `emp_no` в ссылающейся таблице. Система не разрешает выполнения этой операции. 
### Случай 4 (модификация родительской таблицы)
Удаление строки в таблице `employee` со значением `emp_no` равным 10102.

## **`ON DELETE` and `ON UPDATE` options**
При попытке внести обновления в значения первичного ключа, вызывающие несогласованность в соответсвующем внешнем ключе, система БД может реагировать достаточно гибко. 
* `NO ACTION` - Модифицируются только те значения в родительской таблице, для которых нет соответствующих значений во внещнем ключе дочерней таблицы. 
* `CASCADE` - При обновлении первичного ключа или удалении всей строки, в дочерней таблице обновляются все строки с соответсвующими значениями внешнего ключа. 
* `SET NULL` - При несогласованности система БД присваивает внешнемпу ключу всех соответсвующих строк в дочерней таблице значение NULL. 
* `SET DEFAULT` - Аналогично `SET NULL`
```sql
--script
USE sample;
CREATE TABLE works_on1
    (emp_no INTEGER NOT NULL
     project_no CHAR(4) NOT NULL,
     job CHAR(15) NULL,
     enter_date DATE NULL,

     CONSTRAINT prim_works1 PRIMARY KEY (emp_no, project_no),
     CONSTRAINT foreign1_works1 FOREIGN KEY(emp_no)
        REFERENCES employee(emp_no) ON DELETE CASCADE,
     CONSTRAINT foreign2_works1 FOREIGN KEY(project_no)
        REFERENCES project(project_no)( ON UPDATE CASCADE);
```
## **Creating other DB objects**
* `VIEW` - представление, виртуальная таблица, извлекается из одной или нескольких базовых таблиц (DML).
* `INDEX`
* `PROCEDURE` - хранимая процедура, stored procedure
* `TRIGGER` - задает определенное действие в ответ на определенное событие.
* `SYNONYM` - предоставляет связь между самим собой и другим объектом, управляемым одним и тем же или связанным сервером БД. 
```sql
--script
USE AdventureWorks;
CREATE SYNONYM prod FOR AdventureWorks.Poduction.Product;
```
* `schema` - схема, содержит инструкции для создания таблиц, представлений и пользщовательских разрешений. 

## **Integrity constraints and domains**
Домен (domain) - набор все возможных разрешенных значений, которые могут содержать столбцы таблицы.

Псевдоним типа данных (alias data type) - специальный тип, определяемый пользователем.
```sql
--template
CREATE TYPE [type_schema_name.] type_name
{[FROM base_type[(precision[,scale])] [NULL | NOT NULL]]
| [EXTERNAL NAME assembly_name [.class_name]]}
```

```sql
--script
USE sample;
CREATE TYPE zip 
    FROM SMALLINT NOT NULL;
```

Создание табличного типа:
```sql
--script
USE sample;
CREATE TYPE person_table AS TABLE
    (name VARCHAR(30), salary DECIMAL(8,2));
```

## **Modifying DB objects**
* БД
* таблицы
* хранимых процедур
* представлений
* схем
* триггеров

### БД
* добавлять/удалять один/несколько файлов БД
* добавлять/удалять один/несколько файлов журнала
* добавлять/удалять файловые группы
* изменять свойства файлов/файловых групп
* устанавливать параметры БД
* изменять имя БД с помощью хранимой процедуры sp_rename
```sql
--script
USE master;
GO
ALTER DATABASE projects
ADD FILE (NAME=projects_dat1,
          FILENAME='C:\projects1.mdf',
          SIZE=10,
          MAXSIZE=100,
          FILEGROWTH=5);
```

```sql
--script
MODIFY FILE -- изменение свойств файлов
MODIFY FILEGROUP -- и файловых групп
ALTER DATABASE SET -- изменение опций БД
```

## **Altering a table**
```sql
--script
USE sample;
ALTER TABLE employee
    ADD telephone_no CHAR(12) NULL;
-------
USE sample;
ALTER TABLE employee
    DROP COLUMN telephone_no;
-------
ALTER TABLE department
    ALTER COLUMN location CHAR(25) NOT NULL;
-------
USE sample;
CREATE TABLE sales
    (order_no INTEGER NOT NULL,
     order_date DATE NOT NULL,
     ship_date DATE NOT NULL);

ALTER TABLE sales 
    ADD CONSTRAINT order_check CHECK(order_date <= ship_date);
-------
USE sample;
ALTER TABLE sales
    ADD CONSTRAINT primaryk_sales PRIMARY KEY(order_no);
--------
USE sample;
ALTER TABLE sales
    DROP CONSTRAINT order_check;
-------
Use sample;
ALTER TABLE sales
    NOCHECK CONSTRAINT ALL
-------
sp_helpconstraint -- посмотреть все имена ограничений таблицы
```

## **Renaming tables and other DB objects**
```sql
--script
USE sample;
EXEC sp_rename @objname=departmnent, @newname=subdivision
-------
USE sample;
EXEC sp_rename @objname='sales.order_no', @newname=ordernumber
```
## **Removing DB objects**
```sql
--template
DROP object_type object_name
DROP DATABASE database1 {, ...}
DROP TYPE|SYNONYM|PROCEDURE|INDEX|VIEW|TRIGGER|SCHEMA
```