##### LIMIT

```mysql
SELECT *
FROM Persons
LIMIT 5
```







##### DISTINCT

```mysql
SELECT DISTINCT 列名称 FROM 表名称
```



##### IN

```mysql
SELECT * FROM Persons
WHERE LastName IN ('Adams','Carter')
```

| Id   | LastName | FirstName | Address        | City    |
| ---- | -------- | --------- | -------------- | ------- |
| 1    | Adams    | John      | Oxford Street  | London  |
| 3    | Carter   | Thomas    | Changan Street | Beijing |



s

##### 表连接

内连接 inner join

```mysql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Persons.Id_P = Orders.Id_P
ORDER BY Persons.LastName
```



左连接 left join

```mysql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
LEFT JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

| LastName | FirstName | OrderNo |
| -------- | --------- | ------- |
| Adams    | John      | 22456   |
| Adams    | John      | 24562   |
| Carter   | Thomas    | 77895   |
| Carter   | Thomas    | 44678   |
| Bush     | George    |         |



右连接 right join

```mysql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
RIGHT JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

| LastName | FirstName | OrderNo |
| -------- | --------- | ------- |
| Adams    | John      | 22456   |
| Adams    | John      | 24562   |
| Carter   | Thomas    | 77895   |
| Carter   | Thomas    | 44678   |
|          |           | 34764   |

全连接 full join

```mysql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
FULL JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

| LastName | FirstName | OrderNo |
| -------- | --------- | ------- |
| Adams    | John      | 22456   |
| Adams    | John      | 24562   |
| Carter   | Thomas    | 77895   |
| Carter   | Thomas    | 44678   |
| Bush     | George    |         |
|          |           | 34764   |



更改密码

```mysql
SET PASSWORD FOR root=PASSWORD('123456'); #root登陆
```

