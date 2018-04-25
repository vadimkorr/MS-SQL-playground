# Locks (Tasks and solutions)

* 1 Какие типы блокировок совместимы с монопольной блокировкой?

None of other locks

* 2 В чем заключается разница между блокировкой уровня строк и блокировкой уровня страниц?

Row-level locking:

    + maximizes concurrency
    - increases system overhead 

Page-level locking:

    vise versa


* 3 Может ли пользователь явно влиять на реализацию блокировок системой? 

SET LOCK_TIMEOUT - a user can specify whether a transaction should wait or not for a lock to be released.

options in the FROM - TABLOCK, ROWLOCK and PAGLOCK.

* 4 В чем состоит разница между основными типами блокировки (разделяемой и монопольной) и блокировкой намерения? 

An intent lock - always placed at a next higher level in a hierarchy of database objects above the process intents to lock. 

An Exclusive lock, i.e., Shared lock - always used for the object that actually should be locked.

* 5 Что означает понятие укрупнения блокировки? 

(Lock escalation) is the process of converting many page-level locks into one table lock (or many row-level locks into one page lock).

* 6 Что такое взаимоблокировка? 

Two transactions block the progress of each other.

* 7 Какой процесс в качестве "жертвы" в случае взаимоблокировки? Может ли пользователь повлиять на решение системы в этом вопросе? 

The Database Engine always chooses the process that closed the loop in a deadlock. Users can use the SET DEADLOCK_PRIORITY statement to choose the “victim” process.