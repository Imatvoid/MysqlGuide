# sql interview







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







#### [184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

```mysql
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








#### [182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

```mysql
select Email from Person group by Email having count(Email) > 1;
select Email from Person group by Email having count(1) > 1;
select Email from Person group by Email having count(*) > 1;

```

#### [196. 删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)]

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

