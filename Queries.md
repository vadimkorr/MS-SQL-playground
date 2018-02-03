# Запросы

## Содержание
* [Select](#select)

## [**Select**](#select)
```sql
--template
SELECT [ALL|DISTINCT] column_list
    [INTO new_table]
    FROM {table1 [tab_alias1]},...
    [WHERE search_condition]
    [GROUP BY group_by_expression]
    [HAVING search_condition]
    [ORDER BY order_expression [ASC | DESC]];
```
`DISTINCT` можно использовать лишь раз, должен предшествовать всем столбцам

## [**Where**](#where)
```sql
--script
USE sample;
SELECT dept_name, dept_no
    FROM department
    WHERE location='NY';
```

```sql
--template
<> (!=)
<
>
>=
<=
!> -- не больше чем
!< -- не меньше чем
```

```sql
--template
AND
OR
NOT
```







```sql
--template

```


```sql
--script

```