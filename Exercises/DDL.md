# DDL (Tasks and solutions)

* 5.1 Используя инструкцию `CREATE DATABASE`, создайте новую базу данных `test_db`, задав явные спецификации для файлов базы данных и журнала транзакций. Файл базы данных с логическим именем `test_db_dat` сохраняется в физическом файле `D:\tmp\test_db.mdf`, его начальный размер — 5 Мбайт, автоувеличение по 8%, максимальный размер не ограничен. Файл журнала транзакций с логическим именем `test_db_log` сохраняется в физическом файле `D:\tmp\test_db_log.ldf`, его начальный размер — 2 Мбайта, автоувеличение по 500 Кбайт, максимальный размер 10 Мбайт. 

```sql
-- solution
CREATE DATABASE test_db
	ON (NAME=test_db_dat,
		FILENAME='D:\tmp\test_db_dat.mdf',
		SIZE=5,
		FILEGROWTH=8%)
		
	LOG ON (NAME=test_db_log,
			FILENAME='D:\tmp\test_db_log.ldf',
			SIZE=2,
			FILEGROWTH=500KB,
			MAXSIZE=10);
```		
* 5.2 Используя инструкцию `ALTER DATABASE`, добавьте новый файл журнала в базу данных `test_db`. Файл сохраняется в физическом файле `D:\tmp\emp_log.ldf`, его начальный размер — 2 Мбайта, автоувеличение по 2 Мбайта, максимальный размер не ограничен.

```sql
-- solution
ALTER DATABASE test_db
ADD FILE (NAME=test_db_log2,
          FILENAME='D:\tmp\emp_log.ldf',
          SIZE=2,
          FILEGROWTH=2);
```

* 5.3 Используя инструкцию `ALTER DATABASE`, измените начальный размер файла базы данных `test_db` на 10 Мбайт. 

```sql
-- solution
ALTER DATABASE test_db
MODIFY FILE (NAME=test_db_dat,
			 SIZE=10);
```

* 5.4 В примере 5.4 для некоторых столбцов четырех созданных таблиц запрещены значения `NULL`. Для каких из этих столбцов это определение является обязательным, а для каких нет? 

* 5.6 Создайте таблицы `customers` и `orders`, содержащие перечисленные в следующей таблице столбцы. Не объявляйте соответствующие первичный и внешние ключи.

```sql
-- solution
CREATE TABLE customers (
	customerId CHAR(5) NOT NULL,
	companyName VARCHAR(40) NOT NULL,
	contactName CHAR(30) NULL,
	address VARCHAR(60) NULL,
	city CHAR(15) NULL,
	phone CHAR(24) NULL,
	fax CHAR(24) NULL);
	
CREATE TABLE orders (
	orderId INTEGER NOT NULL,
	customerId CHAR(5) NOT NULL,
	orderDate DATE NULL,
	shippedDate DATE NULL,
	freight MONEY NULL,
	shipName VARCHAR(40) NULL,
	shipAddress varchar(60) NULL,
	quantity INTEGER NULL);
```

* 5.7 Используя инструкцию `ALTER TABLE`, добавьте в таблицу orders новый столбец `shipregion`. Столбец должен иметь целочисленный тип данных и разрешать значения `NULL`. 

```sql
-- solution
ALTER TABLE orders
	ADD shipRegion INTEGER NULL
```

* 5.8 Используя инструкцию `ALTER TABLE`, измените тип данных столбца `shipregion` с целочисленного на буквенно-цифровой длиной 8 символов. Столбец может содержать значения `NULL`.

```sql
-- solution
ALTER TABLE orders
	ALTER COLUMN shipRegion VARCHAR(8) NULL
```

* 5.9 Удалите созданный ранее столбец `shipregion` 

```sql
-- solution
ALTER TABLE orders
	DROP COLUMN shipRegion
```

* 5.10 Дайте точное описание происходящему при удалении таблицы с помощью инструкции `DROP TABLE`.

При удалении таблицы удаляются все ее данные, индексы и триггеры. Но представления, созданные по удаленной таблице, не удаляются. Таблицу может удалить только пользователь, имеющий соответствующие разрешения.

* 5.11 Создайте заново таблицы `customers` и `orders`, усовершенствовав их определение всеми ограничениями первичных и внешних ключей.

```sql
-- solution
DROP TABLE customers;
DROP TABLE orders;

CREATE TABLE customers (
	customerId CHAR(5) NOT NULL,
	companyName VARCHAR(40) NOT NULL,
	contactName CHAR(30) NULL,
	address VARCHAR(60) NULL,
	city CHAR(15) NULL,
	phone CHAR(24) NULL,
	fax CHAR(24) NULL,
		CONSTRAINT primary_cust PRIMARY KEY(customerId));
	
CREATE TABLE orders (
	orderId INTEGER NOT NULL,
	customerId CHAR(5) NOT NULL,
	orderDate DATE NULL,
	shippedDate DATE NULL,
	freight MONEY NULL,
	shipName VARCHAR(40) NULL,
	shipAddress varchar(60) NULL,
	quantity INTEGER NULL,
		CONSTRAINT primary_ord PRIMARY KEY(orderId),
		CONSTRAINT foreign_ord FOREIGN KEY(customerId)
			REFERENCES customers(customerId));
```

* 5.12 Используя среду SQL Server Management Studio, попробуйте вставить следующую новую строку в таблицу `orders: (10, 'cli0l', getdate(), getdate(), 100.0, 'Windstar', 'Ocean', 1)`. Почему система отказывается вставлять эту строку в таблицу? 

Т.к. еще нет клиента с id cli0l

```sql
-- solution
INSERT INTO customers (customerId, companyName, contactName, address, city, phone, fax)
	VALUES ('cli0l', 'comp1', 'cont1', 'addr1', 'city1', 'phone1', 'fax1')

INSERT INTO orders (orderId, customerId, orderDate, shippedDate, freight, shipName, shipAddress, quantity)
	VALUES (10, 'cli0l', getdate(), getdate(), 100.0, 'Windstar', 'Ocean', 1) 
```

* 5.13 Используя инструкцию `ALTER TABLE`, определите значение по умолчанию столбца `orderdate` таблицы orders в виде текущей даты и времени системы.

```sql
-- solution
ALTER TABLE orders
	DROP DEFAULT(CURRENT_TIMESTAMP) FOR orderDate;
```

* 5.14 Используя инструкцию `ALTER TABLE`, создайте ограничение для обеспечения целостности, ограничивающее допустимые значения столбца `quantity` таблицы orders диапазоном значений от 1 до 30. 

```sql
-- solution
ALTER TABLE orders
	ADD CONSTRAINT quantity_val CHECK(quantity >= 1 AND quantity <= 30);
```

* 5.15 Отобразите все ограничения для обеспечения целостности таблицы `orders`. 

```sql
-- solution
sp_helpconstraint 'orders';
```

* 5.16 Попытайтесь удалить первичный ключ таблицы `customers`. Почему это не удается?

```sql
-- solution
ALTER TABLE customers
	DROP COLUMN customerId
/* error:
failed because one or more objects access this column
*/
```

* 5.17 Удалите ограничение для обеспечения целостности `prim_empl`, определенное в примере 5.7. 

```sql
-- solution
-- pretty close example
ALTER TABLE orders
	DROP CONSTRAINT primary_ord
```

* 5.18 В таблице `customers` измените имя столбца `city` на `town`. 

```sql
-- solution
sp_rename @objname='customers.city', @newname=town
```