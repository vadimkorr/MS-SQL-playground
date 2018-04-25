# Transactions  (Tasks and solutions)

* 1 Какая цель использования транзакций? 

To keep the data consistent using its "all or nothing" property.

* 2 В чем заключается разница между локальной и распределенной транзакцией? 

A distributed transaction - needs a coordinator that coordinates the execution of all transaction parts on different servers. 

BEGIN TRANSACTION statement to start a local transaction and BEGIN DISTRIBUTED TRANSACTION to start a distributed transaction.

* 3 В чем заключается разница между явным и неявным режимом транзакции? 

Each Transact-SQL statement always belongs either implicitly or explicitly to a transaction.

In implicit transaction mode, selected statements implicitly issue the BEGIN TRANSACTION statement. This means that you do nothing to start such a transaction. An explicit transaction is specified with the pair of statements BEGIN TRANSACTION and COMMIT TRANSACTION (or ROLLBACK TRANSACTION).

* 4 Как можно проверить, было ли успешным выполнение каждой инструкции Transact-SQL? 

Using the global variable called @@error or the combination of the TRY and CATCH statements.

* 5 В каких случаях следует использовать инструкцию `SAVE TRANSACTION`?

To execute parts of an entire transaction.
 
* 6 Изложите разницу между уровнями изоляции `READ UNCOMMITTED` и `SERIALIZABLE`.

READ UNCOMMITTED is the simplest isolation level and therefore allows the maximum of data inconsistency of all isolation levels. On the other hand, the advantage of this isolation level is that it allows the highest concurrency.

The advantage of SERIALIZABLE is that there will be no data inconsistency at all when you apply this isolation level for a process. On the other hand, it decreases concurrency of the processes at most.