- `Partition By`

    ```python
    Partition By
    ```

    Comparison with GROUPBY: We will be able to isolate one column to groupby and remain other info through Partition By

    GROUP BY

    ![1.png](https://github.com/esther119/Learning/blob/master/SQL/1.png)

    PARTITION BY 

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/162bcff2-265b-47b8-a8df-86213a5fc321/Screen_Shot_2021-05-16_at_8.21.07_PM.png](https://github.com/esther119/Learning/blob/master/SQL/2.png)

    Explanation 

    [https://www.youtube.com/watch?v=D6XNlTfglW4](https://www.youtube.com/watch?v=D6XNlTfglW4)

    [[筆記][MSSQL]關於OVER子句的使用方式 - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天](https://ithelp.ithome.com.tw/articles/10190257)

    My practice code 

    ```python
    SELECT OCCUPATION, NAME, COUNT(OCCUPATION) OVER (PARTITION BY OCCUPATION ORDER BY NAME) as occu_count FROM OCCUPATIONS
    ```

    ```python
    SELECT OCCUPATION, NAME, ROW_NUMBER() OVER (PARTITION BY OCCUPATION ORDER BY NAME) as row_num FROM OCCUPATIONS
    ```

- Question: [Occupations](https://www.hackerrank.com/challenges/occupations/problem)

    Sample Code 1

    ```python
    set @r1=0, @r2=0, @r3=0, @r4=0;
    select min(Doctor), min(Professor), min(Singer), min(Actor)
    from(
      select case when Occupation='Doctor' then (@r1:=@r1+1)
                when Occupation='Professor' then (@r2:=@r2+1)
                when Occupation='Singer' then (@r3:=@r3+1)
                when Occupation='Actor' then (@r4:=@r4+1) end as RowNumber,
        case when Occupation='Doctor' then Name end as Doctor,
        case when Occupation='Professor' then Name end as Professor,
        case when Occupation='Singer' then Name end as Singer,
        case when Occupation='Actor' then Name end as Actor
      from OCCUPATIONS
      order by Name
    ) Temp
    group by RowNumber
    ```

    - Explanation

        ## **Step 1:**

        Create a virtual table in your head of the data given to us. It look like this [https://imgur.com/u6DEcNQ](https://imgur.com/u6DEcNQ)

        ```python
        SELECT
            case when Occupation='Doctor' then Name end as Doctor,
            case when Occupation='Professor' then Name end as Professor,
            case when Occupation='Singer' then Name end as Singer,
            case when Occupation='Actor' then Name end as Actor
        FROM OCCUPATIONS
        ```

        ## **Step 2:**

        Create an index column with respect to occupation as "RowNumber".[https://imgur.com/QzVCWFn](https://imgur.com/QzVCWFn)

        Notice from the image, under professor column, the first Name is indexed as 1, the next name "Birtney" as 2. That is what I mean by index w.r.t occupation.

        The below code will only give the "RowNumber" column, to get the result like in image proceed to step 3.

        ```python
        set @r1=0, @r2=0, @r3=0, @r4=0;

        SELECT case 
        	when Occupation='Doctor' then (@r1:=@r1+1)
                when Occupation='Professor' then (@r2:=@r2+1)
                when Occupation='Singer' then (@r3:=@r3+1)
                when Occupation='Actor' then (@r4:=@r4+1) end as RowNumber

        FROM OCCUPATIONS
        ```

        ## **Step 3:**

        Combine the result from step 1 and step 2:

        ```python
        set @r1=0, @r2=0, @r3=0, @r4=0;

        SELECT case 
        	when Occupation='Doctor' then (@r1:=@r1+1)
                when Occupation='Professor' then (@r2:=@r2+1)
                when Occupation='Singer' then (@r3:=@r3+1)
                when Occupation='Actor' then (@r4:=@r4+1) end as RowNumber,
                case when Occupation='Doctor' then Name end as Doctor,
                case when Occupation='Professor' then Name end as Professor,
                case when Occupation='Singer' then Name end as Singer,
                case when Occupation='Actor' then Name end as Actor

        FROM OCCUPATIONS
        ```

        ## **Step 4:**

        Now, Order_by name then Group_By RowNumber.

        Using Min/Max, if there is a name, it will return it, if not, return NULL.

        ```python
        set @r1=0, @r2=0, @r3=0, @r4=0;
        select min(Doctor), min(Professor), min(Singer), min(Actor)
        from(
          select case when Occupation='Doctor' then (@r1:=@r1+1)
                    when Occupation='Professor' then (@r2:=@r2+1)
                    when Occupation='Singer' then (@r3:=@r3+1)
                    when Occupation='Actor' then (@r4:=@r4+1) end as RowNumber,
            case when Occupation='Doctor' then Name end as Doctor,
            case when Occupation='Professor' then Name end as Professor,
            case when Occupation='Singer' then Name end as Singer,
            case when Occupation='Actor' then Name end as Actor
          from OCCUPATIONS
          order by Name
        	) temp
        group by RowNumber;
        ```

        - ***EDIT**** I can see many asking why MIN or temp?

        **temp** - Since I created a temporary table inside the query, I have to give it an alise. It is a good practise.

        **Why MIN in the select statement?** Since some of us here may not be fimilar with sql, I'll start with where I left so you get the whole picture.

        1. Once you complete step 3, add "ORDER BY Name" (Refer above code on where to add Order by clause). The result will look like this [https://imgur.com/aBHUqN6](https://imgur.com/aBHUqN6)

        What changed? the names in all four columns are sorted as per alphabetical order.

        1. Now, we only want the names and not the NULL values from our virtual table. How can we do that? - There maybe be multiple ways, lets us consider the MIN/MAX (Yes, you can replace MIN with MAX and you will get the same result)
        2. Without GROUP BY clause - When a MIN/MAX is used in a Select statement, it will return The "LOWEST" element from each column (which happened to be the first element because we used ORDER BY, if you use MAX, you will get the last element from each column). It will look like this [https://imgur.com/XDZzc4Z](https://imgur.com/XDZzc4Z) That means, you will always get a single row from a table.

        ```python
        SET @r1=0,@r2=0,@r3=0,@r4=0;
        SELECT MIN(Doctor),MIN(Professor),MIN(Singer),MIN(Actor)

        FROM (
        SELECT CASE
            WHEN OCCUPATION = 'Doctor' THEN (@r1:=@r1+1)
            WHEN OCCUPATION = 'Professor' THEN (@r2:=@r2+1)
            WHEN OCCUPATION = 'Singer' THEN (@r3:=@r3+1)
            WHEN OCCUPATION = 'Actor' THEN (@r4:=@r4+1) END AS RowNumber,
            CASE WHEN OCCUPATION = 'Doctor' THEN Name END AS Doctor,
            CASE WHEN OCCUPATION = 'Professor' THEN Name END AS Professor,
            CASE WHEN OCCUPATION = 'Singer' THEN Name END AS Singer,
            CASE WHEN OCCUPATION = 'Actor' THEN Name END AS Actor
        FROM OCCUPATIONS
        ORDER BY Name) as temp
        ```

        1. With GROUP BY clause - The result set will have one row for EACH group (which is RowNumber in our case).

    Sample Code 2: but only run in sql server 

    - they don't highlight PIVOT
    - pivot 需要一氣呵成的寫，沒有辦法被breakdown。無法先pivot再選columns

        ```python
        SELECT <non-pivoted column>,  
            [first pivoted column] AS <column name>,  
            [second pivoted column] AS <column name>,  
            ...  
            [last pivoted column] AS <column name>  
        FROM  
            (<SELECT query that produces the data>)   
            AS <alias for the source query>  
        PIVOT  
        (  
            <aggregation function>(<column being aggregated>)  
        FOR   
        [<column that contains the values that will become column headers>]   
            IN ( [first pivoted column], [second pivoted column],  
            ... [last pivoted column])  
        ) AS <alias for the pivot table>  
        <optional ORDER BY clause>;
        ```

    ```python
    SELECT
        [Doctor], [Professor], [Singer], [Actor]
    FROM
    (
        SELECT ROW_NUMBER() OVER (PARTITION BY OCCUPATION ORDER BY NAME) [RowNumber], * FROM OCCUPATIONS
    ) AS tempTable
    PIVOT
    (
        MAX(NAME) FOR OCCUPATION IN ([Doctor], [Professor], [Singer], [Actor])
    ) AS pivotTable
    ```

    - sql is sensitive to (), so be aware of bracket in the pivot statement

    ```python
    #It is wrong here bc of the bracket. It needs to be in front of AS TABLE 
    SELECT * 
    FROM 
    (
    SELECT ROW_NUMBER() OVER (PARTITION BY OCCUPATION ORDER BY NAME) [RowNumber], * FROM OCCUPATIONS

    PIVOT
    MAX(NAME)
    FOR OCCUPATION IN ([Doctor], [Professor], [Singer], [Actor]) as pvt 
    )
    ```

    Wrong code: I don't know where the problem is

    ```python
    SELECT [Doctor], [Professor], [Singer], [Actor]
    FROM 
    (
        SELECT Occupation, Name, ROW_NUMBER() OVER
        (PARTITION BY Occupation ORDER BY Name) [RowNumber]
        FROM OCCUPATIONS) AS Source
    PIVOT 
    (
        MAX(Name) FOR [Occupation] IN ([Doctor], [Professor], [Singer], [Actor])
    )
    AS Pivot
    ```

    Correct

    ```python
    SELECT [Doctor], [Professor], [Singer], [Actor]
    FROM
    (
        SELECT ROW_NUMBER() OVER (PARTITION BY OCCUPATION ORDER BY NAME) [RowNumber], * FROM OCCUPATIONS) AS Source
    PIVOT
        (MAX(NAME) FOR OCCUPATION IN ([Doctor], [Professor], [Singer], [Actor])
    ) As pvt
    ```

    Correct

    ```python
    SELECT [Doctor], [Professor], [Singer], [Actor]
    FROM
    (
        SELECT Occupation, Name, ROW_NUMBER() OVER (PARTITION BY OCCUPATION ORDER BY NAME)
        as Row_number
        FROM OCCUPATIONS
    ) AS Source
    PIVOT
        (MAX(NAME) FOR OCCUPATION IN ([Doctor], [Professor], [Singer], [Actor])
    ) As pvt
    ```

    ### `PIVOT` tutorial

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/230fbeaa-5551-465d-b2c1-4b9c2dfc9b62/Screen_Shot_2021-05-17_at_11.18.02_AM.png](https://github.com/esther119/Learning/blob/master/SQL/3.png)

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/25377b28-b9c2-4736-9d3e-5247dc67938d/Screen_Shot_2021-05-17_at_11.18.51_AM.png](https://github.com/esther119/Learning/blob/master/SQL/4.png)

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6e5e3881-ea2a-4965-8eb3-8e357b2029fa/Screen_Shot_2021-05-17_at_11.19.07_AM.png](https://github.com/esther119/Learning/blob/master/SQL/5.png)

    [https://www.youtube.com/watch?v=ozy31aJpW-o&t=437s](https://www.youtube.com/watch?v=ozy31aJpW-o&t=437s)

[Combine Two Tables](https://leetcode.com/problems/combine-two-tables/)

```python
SELECT Person.FirstName, Person.LastName, Address.City, Address.State 
FROM Person  
LEFT JOIN Address ON Person.PersonId = Address.PersonId
```

[Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)

```python
SELECT Salary as SecondHighestSalary FROM employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1
```

The SET command is used with UPDATE to specify which columns and values that should be updated in a table.

- [Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/)

```python
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
SET N = N-1; #if n = 8, we want to ignore the first 7 rows 
  RETURN (
      SELECT DISTINCT Salary 
      FROM Employee
      ORDER BY Salary DESC
      LIMIT 1 OFFSET N  
  );
END
```

[Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/)

```python
select E.Name as Employee
From Employee E
JOIN Employee M
ON E.ManagerId = M.Id
WHERE E.Salary > M.Salary
```

[Duplicate Emails](https://leetcode.com/problems/duplicate-emails/)

```python
Select Email 
From Person 
GROUP BY Email 
HAVING
COUNT(Email) > 1
```

[Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/)

```python
SELECT Customers.Name as Customers
FROM Customers 
LEFT JOIN Orders ON Orders.CustomerId = Customers.Id 
WHERE Orders.Id is NULL
```

[Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)

```python
DELETE p1 
FROM  Person p1, Person p2 
WHERE p1.Email = p2.Email and p1.Id > p2.Id
```

[Department Highest Salary](https://leetcode.com/problems/department-highest-salary/)

```python
SELECT E.Name as Employee, E.Salary, D.Name as Department
FROM Employee E, Department D 
WHERE E.DepartmentId = D.Id 
AND (E.DepartmentId, E.Salary) in 
(SELECT DepartmentId, max(Salary) 
FROM Employee 
GROUP BY DepartmentId)
```

#breakitdown

1. Step 1: Find a list for max salary and the department

    ```python
    SELECT DepartmentId, max(Salary) 
    FROM Employee 
    GROUP BY DepartmentId
    ```

2. Step 2: include employees name and their salary

    ```python
    SELECT E.Name as Employee, E.Salary
    FROM Employee E
    WHERE (E.DepartmentId, E.Salary) in 
    (SELECT DepartmentId, max(Salary) 
    FROM Employee 
    GROUP BY DepartmentId)
    ```

    - it will output the name, salary if an employee is the top
3. Add department table in

    Method 1

    ```python
    SELECT E.Name as Employee, E.Salary, D.Name as Department
    FROM Employee E, Department D 
    WHERE E.DepartmentId = D.Id 
    AND (E.DepartmentId, E.Salary) in 
    (SELECT DepartmentId, max(Salary) 
    FROM Employee 
    GROUP BY DepartmentId)
    ```

    Method 2

    ```python
    SELECT E.Name as Employee, E.Salary, D.Name as Department
    FROM Employee E
    JOIN Department D ON D.Id = E.DepartmentId
    WHERE (E.DepartmentId, E.Salary) in 
    (SELECT DepartmentId, max(Salary) 
    FROM Employee 
    GROUP BY DepartmentId)
    ```

[Rising Temperature](https://leetcode.com/problems/rising-temperature/)

```python
SELECT W2.id
FROM Weather W1, Weather W2
WHERE TO_DAYS(W1.recordDate) + 1 = TO_DAYS(W2.recordDate)
AND W1.Temperature < W2.Temperature

#need to use date rather than id, bc date might not follow the id order 
```

[Exchange Seats](https://leetcode.com/problems/exchange-seats/)

My first attempt 

```python
SELECT s2.id, s1.student 
FROM seat s1, seat s2
WHERE s1.id % 2 = s2.id % 2 + 1
AND FLOOR(s1.id/2) + 1 = FLOOR(s2.id / 2)
```

Solution 

```python
SELECT
	CASE
		WHEN seat.id % 2 <> 0 AND seat.id = (SELECT COUNT(*) FROM seat) 
THEN seat.id
		WHEN seat.id % 2 = 0 THEN seat.id - 1
		ELSE
			seat.id + 1
	END as id,
	student 
FROM seat
ORDER BY id
```

Practice 2 

```python
SELECT 
CASE WHEN MOD(id, 2) = 1 AND id != (SELECT COUNT(*) as count FROM seat) THEN id + 1
WHEN MOD(id, 2) = 0 THEN id - 1
ELSE id 
END AS id, student
FROM seat
ORDER BY id;
```

[The PADS](https://www.hackerrank.com/challenges/the-pads/problem)

- Should have ; at the end
- `CASE WHEN` is an object. Either comes right after select, or need a , before using it.

```python
SELECT concat(Name,
CASE WHEN OCCUPATION = 'Doctor' THEN '(D)'
WHEN OCCUPATION = 'Actor' THEN '(A)'
WHEN OCCUPATION = 'Singer' THEN '(S)'
ELSE '(P)'
END)
FROM OCCUPATIONS
ORDER BY Name; 

SELECT 'There are a total of', COUNT(*), concat(LOWER(OCCUPATION),'s.')
FROM OCCUPATIONS
GROUP BY OCCUPATION
ORDER BY COUNT(*); 
```
