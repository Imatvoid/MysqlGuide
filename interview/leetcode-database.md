## uncategorized

#### [597. 好友申请 I ：总体通过率](https://leetcode-cn.com/problems/friend-requests-i-overall-acceptance-rate/)

注意,每张表都可能有重复

```mysql
SELECT round(ifnull((
		SELECT COUNT(DISTINCT concat(requester_id, '-', accepter_id))
		FROM request_accepted
	) / (
		SELECT COUNT(DISTINCT concat(sender_id, '-', send_to_id))
		FROM friend_request
	), 0), 2) AS accept_rate
```



#### [262. 行程和用户](https://leetcode-cn.com/problems/trips-and-users/)

```mysql
# 子查询
select t.request_at Day,   (
        round(count(if(status != 'completed', status, null)) / count(status), 2)
    ) as 'Cancellation Rate' 
    
    from Trips t  
where
    t.Request_at >= '2013-10-01'
and
    t.Request_at <= '2013-10-03'
and (select Banned from Users where Users_Id = t.Client_Id) != 'Yes' 
group by Request_at;


# 表连接
select
    t.request_at Day, 
    (
        round(count(if(status != 'completed', status, null)) / count(status), 2)
    ) as 'Cancellation Rate'
from
    Users u inner join Trips t
on
    u.Users_id = t.Client_Id
and
    u.banned != 'Yes'
where
    t.Request_at >= '2013-10-01'
and
    t.Request_at <= '2013-10-03'
group by
    t.Request_at;
```





#### [626. 换座位](https://leetcode-cn.com/problems/exchange-seats/)

```mysql
SELECT (CASE 
            WHEN MOD(id,2) = 1 AND id = (SELECT COUNT(*) FROM seat) THEN id
            WHEN MOD(id,2) = 1 THEN id+1
            ElSE id-1
        END) AS id, student
FROM seat
ORDER BY id;
```



#### [607. 销售员](https://leetcode-cn.com/problems/sales-person/)

```mysql
SELECT name
FROM salesperson
WHERE sales_id NOT IN (
	SELECT DISTINCT sales_id
	FROM orders
	WHERE com_id = (
		SELECT com_id
		FROM company
		WHERE name = 'RED'
	)
);
```



#### [183. 从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)

```mysql
SELECT cu.Name AS Customers
FROM Customers cu
	LEFT JOIN Orders ord ON cu.Id = ord.CustomerId
GROUP BY cu.Id
HAVING COUNT(ord.CustomerId) = 0
```



#### [580. 统计各专业学生人数](https://leetcode-cn.com/problems/count-student-number-in-departments/)

```mysql
SELECT d.dept_name, COUNT(s.dept_id) AS 'student_number'
FROM department d
	LEFT JOIN student s ON d.dept_id = s.dept_id
GROUP BY d.dept_id
ORDER BY student_number DESC, dept_name;
```





#### [619. 只出现一次的最大数字](https://leetcode-cn.com/problems/biggest-single-number/)


```mysql
SELECT (
		SELECT num
		FROM (
			SELECT num
			FROM my_numbers
			GROUP BY num
			HAVING COUNT(num) = 1
		) t
		ORDER BY t.num DESC
		LIMIT 1
	) AS num
```



#### [596. 超过5名学生的课](https://leetcode-cn.com/problems/classes-more-than-5-students/)

```mysql
SELECT (
		SELECT class
		FROM courses
		GROUP BY class
		HAVING COUNT(DISTINCT student) >= 5
	) AS class;
```




#### [178. 分数排名](https://leetcode-cn.com/problems/rank-scores/)

```mysql
SELECT
	Score,
	( SELECT count( DISTINCT score ) FROM Scores WHERE score >= s.score ) AS Rank 
FROM
	Scores s 
ORDER BY
	Score DESC;
```



#### [176.第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary) 
```mysql
# 子查询 注意distinct 的作用
SELECT
    (SELECT DISTINCT
            Salary
        FROM
            Employee
        ORDER BY Salary DESC
        LIMIT 1 OFFSET 1) AS SecondHighestSalary
;
# 函数
SELECT
    IFNULL(
      (SELECT DISTINCT Salary
       FROM Employee
       ORDER BY Salary DESC
        LIMIT 1 OFFSET 1),
    NULL) AS SecondHighestSalary;


```

#### [177. 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  SET N = N - 1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT
      ( SELECT DISTINCT
            Salary
        FROM
            Employee
        ORDER BY Salary DESC
        LIMIT N,1) 

	);
END
```



#### [75. 组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

```mysql
SELECT
	p.FirstName,
	p.LastName,
	a.City,
	a.State 
FROM
	Person p
	LEFT JOIN Address a ON p.PersonId = a.PersonId
```



#### [181. 超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/).  自我连接

```mysql
SELECT
	a.NAME AS Employee 
FROM
	Employee a
	LEFT JOIN Employee b ON a.ManagerId = b.Id 
WHERE
	a.Salary > b.Salary;
```

#### [197. 上升的温度](https://leetcode-cn.com/problems/rising-temperature/)

```mysql
select 
  w1.Id 
from 
  Weather w1 
   inner join 
  Weather w2 on datediff(w1.RecordDate,w2.RecordDate)=1 
    and 
  w1.Temperature>w2.Temperature;
```



#### 





#### [184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

```mysql
SELECT d.Name AS Department, e.Name AS Employee, e.Salary AS Salary
FROM Employee e
	INNER JOIN Department d on e.DepartmentId = d.Id
WHERE (
	SELECT COUNT( distinct Salary)
	FROM Employee
	WHERE Salary > e.Salary
		AND DepartmentId = e.DepartmentId
) < 1
ORDER BY Department, Salary DESC


# 另外的方式
select 
    d.Name as Department,
    e.Name as Employee,
    e.Salary 
from 
    Employee e,Department d 
where
    e.DepartmentId=d.id 
    and
    (e.Salary,e.DepartmentId) in (select max(Salary),DepartmentId from Employee group by DepartmentId);
```



#### [185. 部门工资前三高的员工](https://leetcode-cn.com/problems/department-top-three-salaries/)

```mysql
SELECT d.Name AS Department, e.Name AS Employee, e.Salary AS Salary
FROM Employee e
	INNER JOIN Department d on e.DepartmentId = d.Id
WHERE (
	SELECT COUNT( distinct Salary)
	FROM Employee
	WHERE Salary > e.Salary
		AND DepartmentId = e.DepartmentId
) < 3
ORDER BY Department, Salary DESC
```








#### [182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

```mysql
select Email from Person group by Email having count(Email) > 1;
select Email from Person group by Email having count(1) > 1;
select Email from Person group by Email having count(*) > 1;

```

#### [196. 删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)

```mysql
# 表连接
SELECT p1.*
FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id
;

DELETE p1 FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id


# 子查询
DELETE from Person 
Where Id not in (
    Select Id 
    From(
    Select MIN(Id) as id
    From Person 
    Group by Email
   ) t
)
```

#### [586. 订单最多的客户](https://leetcode-cn.com/problems/customer-placing-the-largest-number-of-orders/)

```mysql
SELECT
	t.customer_number 
FROM
	( SELECT customer_number, count( customer_number ) AS cs FROM orders GROUP BY customer_number ) t 
ORDER BY
	t.cs DESC 
	LIMIT 1;
```



## Union



#### [602. 好友申请 II ：谁有最多的好友](https://leetcode-cn.com/problems/friend-requests-ii-who-has-the-most-friends/)

```mysql
SELECT r3.requester_id AS id, COUNT(r3.requester_id) AS num
FROM (
	SELECT r1.requester_id
	FROM request_accepted r1
	UNION ALL
	SELECT r2.accepter_id
	FROM request_accepted r2
) r3
GROUP BY r3.requester_id
ORDER BY COUNT(r3.requester_id) DESC
LIMIT 1
```



#### [614. 二级关注者](https://leetcode-cn.com/problems/second-degree-follower/)

在 facebook 中，表 follow 会有 2 个字段： followee, follower ，分别表示被关注者和关注者。

请写一个 sql 查询语句，对每一个关注者，查询他的关注者数目。



```mysql

# 考虑重复情况
SELECT f1.follower, COUNT(DISTINCT f2.follower) AS num
FROM follow f1
	JOIN follow f2 ON f1.follower = f2.followee
GROUP BY f1.follower
```



## Join

#### [613. 直线上的最近距离](https://leetcode-cn.com/problems/shortest-distance-in-a-line/)

```mysql
SELECT min(abs(p1.x - p2.x)) shortest
FROM  point p1 inner join point p2 
ON p1.x != p2.x
```



#### [612. 平面上的最近距离](https://leetcode-cn.com/problems/shortest-distance-in-a-plane/)

```mysql
SELECT round(MIN(sqrt(pow(t1.x - t2.x, 2) + pow(t1.y - t2.y, 2))), 2) AS shortest
FROM point_2d t1
	INNER JOIN point_2d t2 ON (t1.x, t1.y) != (t2.x, t2.y)
	
# 或者简写
select 
round(min(sqrt(pow(t1.x-t2.x,2)+pow(t1.y-t2.y,2))),2) shortest
from point_2d as t1,point_2d as t2
where (t1.x,t1.y) != (t2.x,t2.y)
```



#### [608. 树节点](https://leetcode-cn.com/problems/tree-node/)

给定一个表 tree，id 是树节点的编号， p_id 是它父节点的 id 。
```
+----+------+
| id | p_id |
+----+------+
| 1  | null |
| 2  | 1    |
| 3  | 1    |
| 4  | 2    |
| 5  | 2    |
+----+------+
```
树中每个节点属于以下三种类型之一：

叶子：如果这个节点没有任何孩子节点。
根：如果这个节点是整棵树的根，即没有父节点。
内部节点：如果这个节点既不是叶子节点也不是根节点。


写一个查询语句，输出所有节点的编号和节点的类型，并将结果按照节点编号排序。上面样例的结果为：
```
+----+------+
| id | Type |
+----+------+
| 1  | Root |
| 2  | Inner|
| 3  | Leaf |
| 4  | Leaf |
| 5  | Leaf |
+----+------+
```

解释

节点 '1' 是根节点，因为它的父节点是 NULL ，同时它有孩子节点 '2' 和 '3' 。
节点 '2' 是内部节点，因为它有父节点 '1' ，也有孩子节点 '4' 和 '5' 。
节点 '3', '4' 和 '5' 都是叶子节点，因为它们都有父节点同时没有孩子节点。
样例中树的形态如下：

```
			  1
			/   \
	                  2       3
	                /   \
	              4       5

```


```mysql
SELECT DISTINCT t1.id
	, CASE 
		WHEN t1.p_id IS NULL THEN 'Root'
		WHEN t2.id IS NOT NULL THEN 'Inner'
		ELSE 'Leaf'
	END AS Type
FROM tree t1
	LEFT JOIN tree t2 ON t1.id = t2.p_id
ORDER BY t1.id
```



## Easy

#### [584. 寻找用户推荐人](https://leetcode-cn.com/problems/find-customer-referee/) 

#### 注意包含null 

null 不能使用 = , != 永远返回false

```mysql
select name from customer where referee_id != 2 or referee_id is null
```



#### [585. 2016年的投资](https://leetcode-cn.com/problems/investments-in-2016/)

```mysql 
   select ROUND(SUM(TIV_2016),2) as  TIV_2016 
    from insurance
    where (LAT,LON) not IN
    (
        select LAT,LON from insurance
        group by LAT,LON
        having count(PID) > 1
    )
    and TIV_2015 IN
    (
        select TIV_2015 from insurance
        group by TIV_2015
        having count(PID) > 1
    )
```



#### [577. 员工奖金](https://leetcode-cn.com/problems/employee-bonus/)

选出所有 bonus < 1000 的员工的 name 及其 bonus。

```mysql
SELECT e.name, b.bonus
FROM Employee e
	LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus IS NULL
	OR b.bonus < 1000
```

#### [620. 有趣的电影](https://leetcode-cn.com/problems/not-boring-movies/)

作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。

```mysql
SELECT id, movie, description, rating
FROM cinema
WHERE description != 'boring'
	AND id % 2 = 1
ORDER BY rating DESC;

# 或者 mod(id, 2) = 1
```

#### [1045. 买下所有产品的客户](https://leetcode-cn.com/problems/customers-who-bought-all-products/)

写一条 SQL 查询语句，从 `Customer` 表中查询购买了 `Product` 表中所有产品的客户的 id。

```mysql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (
	SELECT COUNT(DISTINCT product_key)
	FROM Product
);
```



## 筛选至少



#### [1050. 合作过至少三次的演员和导演](https://leetcode-cn.com/problems/actors-and-directors-who-cooperated-at-least-three-times/)   distinct作用于多列

写一条SQL查询语句获取合作过至少三次的演员和导演的 id 对 `(actor_id, director_id)`



```mysql
SELECT DISTINCT actor_id AS ACTOR_ID, director_id AS DIRECTOR_ID
FROM ActorDirector a
WHERE (
	SELECT COUNT(1)
	FROM ActorDirector
	WHERE actor_id = a.actor_id
		AND director_id = a.director_id
) >= 3
```



#### [570. 至少有5名直接下属的经理](https://leetcode-cn.com/problems/managers-with-at-least-5-direct-reports/)

`Employee` 表包含所有员工和他们的经理。每个员工都有一个 Id，并且还有一列是经理的 Id

```mysql
SELECT DISTINCT Name
FROM Employee e
WHERE (
	SELECT COUNT(1)
	FROM Employee
	WHERE ManagerId = e.id
) >= 5
```

## 根据条件多输出一列 CASE WHEN 



#### [610. 判断三角形](https://leetcode-cn.com/problems/triangle-judgement/)

#### CASE WHEN

```mysql
SELECT x, y, z
	, CASE 
		WHEN (x + y > z
		AND x + z > y
		AND z + y > x) THEN 'Yes'
		ELSE 'No'
	END AS triangle
FROM triangle
```





#### [627. 交换工资](https://leetcode-cn.com/problems/swap-salary/) case语句

```mysql
UPDATE salary
SET
    sex = CASE sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;

```





## Group by

#### [574. 当选者/票数最高的人](https://leetcode-cn.com/problems/winning-candidate/)

表: `Candidate`

```
+-----+---------+
| id  | Name    |
+-----+---------+
| 1   | A       |
| 2   | B       |
| 3   | C       |
| 4   | D       |
| 5   | E       |
+-----+---------+  
```

表: `Vote`

```
+-----+--------------+
| id  | CandidateId  |
+-----+--------------+
| 1   |     2        |
| 2   |     4        |
| 3   |     3        |
| 4   |     2        |
| 5   |     5        |
+-----+--------------+
id 是自动递增的主键，
CandidateId 是 Candidate 表中的 id.
```

请编写 sql 语句来找到当选者的名字，上面的例子将返回当选者 `B`.

```
+------+
| Name |
+------+
| B    |
+------+
```

**注意:**

1. 你可以假设**没有平局**，换言之，**最多**只有一位当选者。



```mysql
SELECT Name
FROM Candidate
WHERE id = (
	SELECT CandidateId
	FROM Vote
	GROUP BY CandidateId
	ORDER BY count(1) DESC
	LIMIT 1
) 

```



#### [578. 查询回答率最高的问题](https://leetcode-cn.com/problems/get-highest-answer-rate-question/)

```mysql
select question_id survey_log
from 
survey_log
group by
question_id
order by 
sum(case when action='answer' then 1 else 0 end)/
sum(case when action='show' then 1 else 0 end)
desc limit 1;

# 或者count
select question_id  survey_log
from survey_log
group by question_id
order by
(count(action='answer' or null)/count(action='show' or null)) desc limit 1;
```







## 用户变量

#### [569. 员工薪水中位数](https://leetcode-cn.com/problems/median-employee-salary/)



```mysql
SELECT Ranking.Id, Ranking.Company, Ranking.Salary
FROM (
	   SELECT e.Id, e.Salary, e.Company
		  , IF(@prev = e.Company, @Rank := @Rank + 1, @Rank := 1) AS rank
		  , @prev := e.Company
	   FROM Employee e, (SELECT  @prev := null,@Rank := 0) temp
	   ORDER BY e.Company, e.Salary, e.Id
    ) Ranking
	INNER JOIN
	(
		SELECT COUNT(1) AS totalcount, Company
		FROM Employee 
		GROUP BY Company
	) Companycount
	ON Companycount.Company = Ranking.Company
WHERE Rank = FLOOR((totalcount + 1) / 2)
	OR Rank = FLOOR((totalcount + 2) / 2)
```



#### [180. 连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/)

```mysql
select distinct Num as ConsecutiveNums
from (
  select Num, 
    case 
      when @prev = Num then @count := @count + 1
      when @prev := Num then @count := 1
    end as CNT
  from Logs, (select @prev := null,@count := null) as t
) as temp
where temp.CNT >= 3
```



#### [603. 连续空余座位](https://leetcode-cn.com/problems/consecutive-available-seats/)

两个连在一起就算连续

```mysql

select
    distinct c1.seat_id
from
    cinema as c1 left join cinema as c2
on
    c1.seat_id+1 = c2.seat_id or c1.seat_id-1 = c2.seat_id
where
    c1.free =1 and c2.free = 1
order by
    seat_id

```

