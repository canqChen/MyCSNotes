# [分数排名](https://leetcode-cn.com/problems/rank-scores/)

​	<img src="imgs/sql/practice1.png" alt="practice1" style="zoom:80%;" />

* **四大排名函数**

  * row_number()：依次递增排名，无重复排名，1，2，3，4，5，6
  * rank()：相同分数归为一组排名相同，但是重复后下一个人按照实际排名，**排名不连续**，1，1，3，4，4，6
  * dense_rank()：排名连续，**分数一致排名一致**，分数不一致排名 + 1，1，1，2，3，3，4
  * NTILE(4)：将所有记录分成group_num个组，组内序号一样，**排名按组递增**，1，1，1，2，2，3，3(3组)
  * 语句
  * 用法：
    <窗口/排名函数> OVER ( [PARTITION BY <用于分组的字段> ] ORDER BY <排序的字段> ）
    * 其中[ ]中的内容可以忽略

  ```mysql
  select Score, dense_rank() OVER(order by Score desc) as `Rank` from Scores; 
  ```

* 嵌套查询：包含两个部分，第一部分是降序排列的分数，第二部分是每个分数对应的排名

  * 第一部分

    ```mysql
    select a.Score as Score from Scores a order by a.Score DESC -- Score降序排序
    ```

  * 第二部分，查询每个分数前去重后的个数

    ```mysql
    select count(distinct b.Score) from Scores b where b.Score >= X as `Rank`;
    ```

  * 合并

    ```mysql
    select a.Score as Score,
    (select count(distinct b.Score) from Scores b where b.Score >= Score) as `Rank`
    from Scores a
    order by Score DESC;
    ```

# [第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

<img src="imgs/sql/practice2.png" alt="practice2" style="zoom:80%;" />

* 解法一：group by去重，limit中不能进行计算

  ```mysql
  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
  BEGIN
      set N := N-1;
    -- declare m int;
    -- set m = N-1;
    RETURN (
      select Salary from Employee group by salary order by Salary desc limit N, 1 
      #select IFNULL((select distinct Salary from Employee order by Salary DESC limit N, 1), NULL)
    );
  END
  ```

* 解法二：子查询

  * 排名第N的薪水意味着该表中存在N-1个比其更高的薪水
  * 注意这里的N-1个更高的薪水是指去重后的N-1个，实际对应人数可能不止N-1个
  * 最后返回的薪水也应该去重，因为可能不止一个薪水排名第N
  * 由于对于每个薪水的where条件都要执行一遍子查询，注定其效率低下

  ```mysql
  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
  BEGIN
    RETURN (
        SELECT 
            DISTINCT e.salary
        FROM 
            employee e
        WHERE 
            (SELECT count(DISTINCT salary) FROM employee WHERE salary>e.salary) = N-1
    );
  END
  ```

* 解法三：排序函数(dense_rank)

  ```mysql
  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
  BEGIN
    RETURN (
          SELECT 
              DISTINCT salary
          FROM 
              (SELECT salary, dense_rank() over(ORDER BY salary DESC) AS rnk
               FROM employee) tmp
          WHERE rnk = N
    );
  END
  ```

# [获取当前薪水第二多的员工的emp_no以及其对应的薪水salary](https://www.nowcoder.com/practice/c1472daba75d4635b7f8540b837cc719?tpId=82&&tqId=29770&rp=1&ru=/activity/oj&qru=/ta/sql/question-ranking)

* 题目描述

  <img src="imgs/sql/practice3.png" alt="practice3" style="zoom:80%;" />

* 解法一：

  使用聚合去掉最高薪水，再取最高薪水

  ```sql
  select e.emp_no, s.salary, e.last_name, e.first_name
  from employees e join salaries s on e.emp_no = s.emp_no
  where s.salary = (					-- 3.将第二高薪水作为查询条件
      select max(s1.salary) from salaries as s1 where s1.salary <  -- 2.去掉最高薪水
      (select max(s2.salary) from salaries as s2)  -- 1.选出最高薪水
  );
  ```

* 解法二：

  ```sql
  select s.emp_no, s.salary, e.last_name, e.first_name
  from salaries s join employees e
  on s.emp_no = e.emp_no
  where s.salary =
      (
      select s1.salary
      from salaries s1 join salaries s2      -- 自连接查询
      on s1.salary <= s2.salary
      group by s1.salary                     -- 当s1<=s2连接并以s1.salary分组时一个s1会对应多个s2
      having count(distinct s2.salary) = 2   -- 去重之后的数量就是对应的名次
      and s1.to_date = '9999-01-01'
      and s2.to_date = '9999-01-01'
      )
  and s.to_date = '9999-01-01'
  ```

  表自连接以后:

  |  s1  |  s2  |
  | :--: | :--: |
  | 100  | 100  |
  |  98  |  98  |
  |  98  |  98  |
  |  95  |  95  |

  当**s1<=s2**连接并以**s1.salary**分组时一个s1会对应多个s2

  |  s1  |  s2  |
  | :--: | :--: |
  | 100  | 100  |
  |  98  | 100  |
  |  98  |  98  |
  |  98  |  98  |
  |  95  | 100  |
  |  95  |  98  |
  |  95  |  98  |
  |  95  |  95  |

  **对s2进行去重统计数量, 就是s1对应的排名**

* 解法三：使用排序函数

  ```sql
  select b.emp_no, b.salary, e.last_name, e.first_name from
  employees as e inner join
  (select emp_no, salary, dense_rank() over (partition by to_date order by salary desc) as rk
  from salaries where to_date='9999-01-01') as b
  on e.emp_no = b.emp_no
  where rk=2;
  ```


# 查找在职员工自入职以来的薪水涨幅情况

* 题目描述

  <img src="imgs/sql/practice4.png" alt="practice4" style="zoom:80%;" />

* 解题思路：
  * 本题应该考虑**降薪**的问题:
    题中没有讲员工不降薪，所以不能直接使用最大薪水-最小薪水求值，例如：我入职薪水100元，过程中降薪20元，现在工资80元，那么按照（最大薪水-最小薪水求值）,薪水还涨了20元，但实际是降薪20元，所有有错误。
    **实际应该使用最后记录薪水-入职时薪水（80-100）,涨幅为-20元**

* 解法

  ```sql
  select b.emp_no, (b.salary-a.salary) as growth
  from 
  (select e.emp_no, s.salary
  from employees e left join salaries s on e.emp_no=s.emp_no
  and e.hire_date=s.from_date) a -- 入职工资表
  inner join 
  (select e.emp_no, s.salary
  from employees e left join salaries s on e.emp_no=s.emp_no
  where s.to_date='9999-01-01') b -- 现在工资表
  on a.emp_no=b.emp_no
  order by growth;
  ```

  * a表(入职工资表)

    ![practice4_1](imgs/sql/practice4_1.png)

  * b表(现在工资表)

    ![practice4_2](imgs/sql/practice4_2.png)

  * a、b表连接

    ![practice4_3](imgs/sql/practice4_3.png)



# 异常的邮件概率

* https://www.nowcoder.com/practice/d6dd656483b545159d3aa89b4c26004e?tpId=82&&tqId=35083&rp=1&ru=/activity/oj&qru=/ta/sql/question-ranking

* 代码

  ```sql
  select e.date, round(sum(e.type = "no_completed") / count(e.type), 3) as p
  from email e 
  join user as u1 on (e.send_id=u1.id and u1.is_blacklist=0)
  join user as u2 on (e.receive_id=u2.id and u2.is_blacklist=0)
  group by e.date
  order by e.date;
  
  select email.date, round(sum(case email.type when'completed' then 0 else 1 end)*1.0/count(email.type),3) as p
  from email
  join user as u1 on (email.send_id=u1.id and u1.is_blacklist=0)
  join user as u2 on (email.receive_id=u2.id and u2.is_blacklist=0)
  group by email.date order by email.date;
  ```

  
