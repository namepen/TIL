# 윈도우 함수(Window Function)

### 모양새

함수(컬럼) OVER (PARTITION BY 컬럼 ORDER BY 컬럼)

### 개요

- `GROUP BY` 와 다르게 데이터를 압축하지 않고 연산한 값을 각 ROW에 별도로 생성합니다.
- `PARTITION BY` 를 쓰지않고 `ORDER BY` 만 사용하게 되면, 누적합(CUMSUM)을 계산합니다.
- `ORDER BY` → `PARTITION BY` 로 구성하게되면, PARTION BY 의 컬럼별로 ORDER BY 컬럼 데이터에 따라 누적합을 계산합니다.

### 윈도우 함수에서만 사용할 수 있는 함수들

- 순위 정하기

    ROW_NUMBER() : 중복되는 순위없이 순위 부여

    RANK() : 값이 같으면 같은 순위를 부여(다음 순번은 중복된 순위를 제외한 순위 

     (EX. 1등 2명 → 다음 데이터는 3등)

    DENSE_RANK() : 순위가 같은 데이터가 있어도, 순차적인 순위 부여

     (EX. 1등 2명 → 다음 데이터는 2등)

    **예시 : 1위가 2명있다면, RANK는 세번째 데이터를 3위로, DENSE_RANK는 2위로 계산

- 데이터 위치 바꾸기

    LAG : 데이터를 특정 칸만큼 뒤로 미는 함수(맨 앞이 Null)

    LEAD : 데이터를 특정 칸만큼 앞으로 당기는 함수(맨 뒤가 Null)

    **예시 : LAG(COL_NAME, 2, 0) : COL_NAME 컬럼을 2칸 뒤로 이동하며, NULL값은 0으로 설정

    ## 문제 풀이

    1. [184. Department Highest Salary](https://leetcode.com/problems/department-highest-salary/)

    ```sql
    SELECT Department
         , Employee
         , Salary
    FROM (
        SELECT D.Name as Department
             , E.Name as Employee
             , E.Salary AS Salary
             , MAX(E.Salary) OVER (PARTITION BY E.DepartmentId) AS MaxSalary
        FROM Employee AS E
        INNER JOIN Department AS D
        ON E.DepartmentId = D.Id
        ) AS MAX_SALARY
    WHERE MaxSalary = Salary
    ORDER BY Salary
    ;
    ```

    2. Elevator

    ```sql
    # Solution 1. WINDOW FUNCTION
    SELECT *
       , SUM(kg) OVER (ORDER BY LINE PARTITION BY Id) AS Cumsum
    FROM Elevator;

    # Solution 2. INNER JOIN
    SELECT e1.Id
     , e1.Name
     , e1.kg
     , e1.Line
     , SUM(e2.kg) AS Cumsum
    FROM Elevator e1
    INNER JOIN Elevator e2
    ON e1.Id = e2.Id
    AND e1.Line >= e2.Line
    GROUP BY 1,2,3,4;

    # Solution 3. Select sub query : 한 줄(row)과 테이블을 비교해보면서 작성해보자.
    SELECT e1.Id
     , e1.Name
     , e1.kg
     , e1.Line
     , (SELECT SUM(kg) 
    FROM Elevator e2
    WHERE e1.Id = e2.Id
    AND e1.Line >= e2.Line) AS Cumsum
    FROM Elevator e1
    ```

    3. [180. Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)

    ```sql
    # Solution 1. 
    SELECT DISTINCT(l1.Num) AS ConsecutiveNums 
    FROM Logs l1
    INNER JOIN Logs l2
        ON l1.Id +1 = l2.Id 
    INNER JOIN logs l3 
        ON l1.Id +2 = l3.Id
    WHERE l1.Num = l2.Num AND l1.Num = l3.Num;

    # Solution 2.
    SELECT DISTINCT l.Num AS ConsecutiveNums
        FROM (
        SELECT Num
             , LEAD(Num, 1) OVER (ORDER BY ID) AS Next
             , LEAD(Num, 2) OVER (ORDER BY ID) AS Afternext
        FROM Logs
    ) AS l
    WHERE l.Num = l.Next AND l.Num = l.Afternext;
    ```

    4. [185. Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/)

    ```sql
    # Solution 1. Window function
    SELECT D.Name AS Department
         , S.Name AS Employee
         , S.Salary AS Salary
    FROM(
        SELECT *
             , DENSE_RANK() OVER (PARTITION BY DepartmentId ORDER BY Salary DESC) AS SALARY_RANK
        FROM Employee
    ) AS S
    INNER JOIN Department AS D ON S.DepartmentId = D.Id
    WHERE SALARY_RANK < 4
    ORDER BY S.ID;

    # Solution 2. Not Use window function
    # Think : 그룹 별 순위를 매기기 위해 @ 변수 생성
    #미리 정렬한 테이블에서 부서가 변동될 때마다 salary에 순위 부여
    # 상위 3개의 salary와 부서 정보 테이블 생성
    # employee 테이블과 조인하여 결과 생성
    SELECT department.name AS Department
         , employee.name AS Employee
         , employee.salary AS Salary 
    FROM employee
    INNER JOIN (
        SELECT departmentid
             , salary
        FROM (
            SELECT Salary
                 , CASE
                    WHEN DepartmentId = @group THEN @rownum:=@rownum+1 ELSE @rownum:=1
                   END AS r
                 , @group:=DepartmentId AS departmentid
            FROM (
                SELECT departmentid, salary
                FROM employee 
                GROUP BY departmentid, salary
                ORDER BY departmentid, salary DESC) AS t
            INNER JOIN (SELECT @rownum:=0, @group:='') AS tmp
            ) AS top_salary
        WHERE r < 4
        ) AS high_salary
    ON employee.DepartmentId = high_salary.departmentid
    AND employee.salary = high_salary.salary
    INNER JOIN department
    ON employee.DepartmentId = department.id;
    ```