---
layout: post
title: leetcode 数据库题目全解
categories: db
tags: leetcode database
---


####     https://leetcode.com/problems/customers-who-never-order/
    
    SELECT 
        Name
    FROM
        Customers c
    WHERE
        c.Id NOT IN (SELECT 
                CustomerId
            FROM
                Orders

    
####     https://leetcode.com/problems/delete-duplicate-emails/
    
    DELETE p1 FROM Person p1
            JOIN
        Person p2 ON p1.Id > p2.Id AND p1.Email = p2.Email
    
    
####    https://leetcode.com/problems/department-highest-salary/

    SELECT 
        d_name, e1.Name, max_salary
    FROM
        Employee e1
            JOIN
        (SELECT 
            d.Id d_id, d.Name d_name, MAX(Salary) max_salary
        FROM
            Department d
        JOIN Employee e2 ON d.id = e2.departmentId
        GROUP BY d.Name) d_max_salary ON e1.Salary = d_max_salary.max_salary
            AND e1.departmentId = d_max_salary.d_id;
    
    
    
####     https://leetcode.com/problems/department-top-three-salaries/

    SELECT 
        d.name, top3_salary.Name, top3_salary.Salary
    FROM
        Department d
            JOIN
        (SELECT 
            DepartmentId, Salary, Name
        FROM
            Employee e1
        WHERE
            NOT EXISTS( SELECT 
                    COUNT(e2.Salary)
                FROM
                    Employee e2
                WHERE
                    e1.DepartmentId = e2.DepartmentId
                        AND e2.Salary > e1.Salary
                HAVING COUNT(DISTINCT e2.Salary) >= 3)) top3_salary ON d.Id = top3_salary.DepartmentId
    
    
####     https://leetcode.com/problems/duplicate-emails/

    SELECT 
        Email
    FROM
        Person
    GROUP BY Email
    HAVING COUNT(Email) > 1
    
    
####     https://leetcode.com/problems/employees-earning-more-than-their-managers/
    SELECT 
        e1.Name
    FROM
        Employee e1
            JOIN
        Employee e2 ON e1.ManagerId = e2.Id
            AND e1.Salary > e2.Salary
    
    
####     https://leetcode.com/problems/nth-highest-salary/
    CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
    BEGIN
      Declare n_1 int ;
      set n_1 = N -1;
      RETURN (
    SELECT 
        MAX(e.Salary)
    FROM
        Employee e
            LEFT JOIN
        (SELECT 
            distinct Salary 
        FROM
            Employee
        ORDER BY Salary DESC
        LIMIT n_1) n_1_high ON e.Salary = n_1_high.Salary
    WHERE
        n_1_high.Salary IS NULL
      );
    END
    
    
####       https://leetcode.com/problems/rank-scores/
    
    SELECT 
        s3.Score, fuck.nth
    FROM
        Scores s3
            JOIN
        (SELECT 
            s1.Score, COUNT(DISTINCT s2.Score) nth
        FROM
            Scores s1
        RIGHT JOIN Scores s2 ON s1.Score <= s2.Score
        GROUP BY s1.Score) fuck ON s3.Score = fuck.Score
    ORDER BY s3.Score DESC
    
    
####     https://leetcode.com/problems/rising-temperature/
    #attention :  date(create_time) + 1 不要用，坑
    SELECT 
        a.Id
    FROM
        Weather a
            JOIN
        Weather b ON a.Date = DATE_ADD(b.Date, INTERVAL 1 DAY)
    WHERE
        a.Temperature > b.Temperature
    
    
####    https://leetcode.com/problems/second-highest-salary/

    SELECT 
        MAX(Salary)
    FROM
        Employee
    WHERE
        Salary < (SELECT 
                MAX(Salary)
            FROM
                Employee);
    
    
####    https://leetcode.com/problems/trips-and-users/
    
    #method1
    SELECT 
        all_table.Request_at,
        CASE
            WHEN cancel_cnt IS NULL THEN 0.00
            ELSE ROUND(cancel_cnt / cnt, 2)
        END
    FROM
        ((SELECT 
            Request_at, COUNT(*) cancel_cnt
        FROM
            Trips t
        JOIN Users u ON t.Client_Id = u.Users_id
        WHERE
            Banned = 'No'
                AND Request_at >= '2013-10-01'
                AND Request_at <= '2013-10-03'
                AND status <> 'completed'
        GROUP BY Request_at) cancel_table
        RIGHT JOIN (SELECT 
            Request_at, COUNT(*) cnt
        FROM
            Trips t
        JOIN Users u ON t.Client_Id = u.Users_id
        WHERE
            Banned = 'No'
                AND Request_at >= '2013-10-01'
                AND Request_at <= '2013-10-03'
        GROUP BY Request_at) all_table ON cancel_table.Request_at = all_table.Request_at)
    
    #method2
    SELECT 
        Request_at ,
        ROUND(SUM(IF(Status = 'completed', 0, 1)) / COUNT(*),
                2)
    FROM
        Trips t
            LEFT JOIN
        Users t1 ON t.Client_Id = t1.Users_Id
    WHERE
        t1.Banned = 'No'
            AND Request_at BETWEEN '2013-10-01' AND '2013-10-03'
    GROUP BY t.Request_at;
    
    
####    https://leetcode.com/problems/combine-two-tables/

    SELECT 
        Person.FirstName,
        Person.LastName,
        Address.City,
        Address.State
    FROM
        Person
            LEFT JOIN
        Address ON Person.PersonId = Address.PersonId
    
    
####    https://leetcode.com/problems/consecutive-numbers/

    SELECT 
        distinct c1.Num
    FROM
        Logs c1
            JOIN
        Logs c2 ON c1.Num = c2.Num AND c1.Id = c2.Id - 1
            JOIN
        Logs c3 on c2.Num = c3.Num and c2.Id = c3.Id - 1;
    
    
