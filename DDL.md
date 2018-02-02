# DDL

## Содержание
* [Создание БД](#Создание-БД)
* [Создание моментального снимка БД](#Создание-моментального-снимка-БД)
* [Присоединение и отсоединение БД](#Присоединение-и-отсоединение-БД)
* [CREATE TABLE (базовая инструкция)](#CREATE-TABLE-базовая-инструкция)
* [CREATE TABLE (с ограничениями декларативной целостности)](#CREATE-TABLE-с-ограничениями-декларативной-целостности)
* [Предложение `UNIQUE`](#Предложение-UNIQUE)
* [Предложение `PRIMARY KEY`](#Предложение-PRIMARY-KEY)
* [Предложение `CHECK`](#Предложение-CHECK)
* [Предложение `FOREIGN KEY`](#Предложение-FOREIGN-KEY)
* [Ссылочная целостность](#Ссылочная-целостность)

## **Создание БД**
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

## **Создание моментального снимка БД**
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
## **Присоединение и отсоединение БД**
Используется при перемещении БД. 

*Отсоединение:* системная функция `sp_detach_db` (ДБ должна находиться в однопользовательском режиме)

*Присоединение:* `CREATE DATABASE` с `FOR ATTACH`

## **CREATE TABLE (базовая инструкция)**
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

## **CREATE TABLE (с ограничениями декларативной целостности)**
Ограничения, которые используются для проверки данных при модификации или вставке (integrity constraints). Декларативные (`CREATE TABLE`, `ALTER TABLE`) и при помощи триггеров. Ограничения уровня столбцов, уровня таблицы. 

Декларативные ограничения можно сгруппировать:
* `DEFAULT`
* `UNIQUE`
* `PRIMARY KEY`
* `CHECK`
* `FOREIGN KEY` ссылочная целостность

## **Предложение `UNIQUE`**
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

## **Предложение `PRIMARY KEY`**
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

## **Предложение `CHECK`**
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

## **Предложение `FOREIGN KEY`**
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

## **Ссылочная целостность**






```sql
--template

```


```sql
--script

```