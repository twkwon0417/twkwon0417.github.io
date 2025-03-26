---
title: Introduction to SQL 
date: 2025-03-16 22:10:10 +9000
categories: [Konkuk_3-1, Database]
tags: [sql]     # TAG names should always be lowercase
---

Introduction to SQL
==

- SQL allows duplicates in relations where Relation Alg don't

Why SQL
--
- The SQL Data-Definition Language(DDL) allows the specification of information about relations, including
  - The schema for each relation
  - The **domain** of values associated with each attribute
  - Integrity constraints
  - and so on(provided by database system)
> **Integrity constraints e.g.**
> ```sql
>   create table instructor (
>       id        char(5),
>       name      varchar(20) not null,
>       dept_name varchar(20),
>       salary    numeric(8, 2),
>       primary key(ID),
>       foreign key(dept_name) references department);
>   );
> ```
> 이것들도 Integrity Constraints
> - Department table이 존재 해야함
> - Department table에는 dept_name이 존재해야 함

Basic Query Structure
--
```sql
select A1, A2, ..., An
from r1, r2, ..., rm
where P
```
- **P** is a predicate
- from 에 ','로 묶여 있어서 r1, r2, ..., rm이 cartesian product된 table에서 sql문 작동

Rename Operation
--
- The SQL allows renaming relations and attributes using the as clause:
```sql
select id, name, sa
```
String Operations
--

### %, _, ...
```sql
 SELECT * FROM name WHERE first_name LIKE 'y%';
 SELECT * FROM name WHERE first_name LIKE '%on%';
 SELECT * FROM name WHERE first_name LIKE '_on_';
```
1. **y**oura, **y**aml can be selected
2. t**on**y, j**on**ny can be selected
3. Only tony can be selected

- ‘_ _ _’ matches any string of exactly three characters.
- ‘_ _ _ %’ matches any string of at least three characters.

```sql
SELECT 
  UPPER(first_name) AS upper_case_name,     -- Converts the name to uppercase
  LENGTH(first_name) AS name_length,          -- Returns the number of characters in the name
  SUBSTRING(first_name, 1, 3) AS name_prefix    -- Extracts the first three characters from the name
FROM employees;
```
- Result

| upper_case_name | name_length | name_prefix |
|-----------------|-------------|-------------|
| ALICE           | 5           | Ali         |
| BOB             | 3           | Bob         |
| CHARLIE         | 7           | Cha         |

Between, order by
--
둘은 같다. order by 빼고
```sql
SELECT name
FROM instructor
WHERE salary >= 90000 AND salary <= 100000;
ORDER BY salary ASC -- Default
  
SELECT name
FROM instructor
WHERE salary BETWEEN 90000 AND 100000;
ORDER BY salary DESC
```

Set Operations
--
- Union
- Intersect
- Except

위 Operation은 자동으로 duplicate을 제거해 <br>
그거 실흐면 꼭 all을 붙이자! intersect all, except all...
```sql
(SELECT course_id
 FROM section
 WHERE sem = 'Fall' AND year = 2017)
UNION
(SELECT course_id
 FROM section
 WHERE sem = 'Spring' AND year = 2018);
```

NULL
--
> **NULL == UNKNOWN**
> **UNKNOWN is UNKNOWN은 true로 간주**
> ** where문의 결과가 UNKNOWN인 경우 false로 처리**

### AND Operator
|       AND       | TRUE   | FALSE  | UNKNOWN |
|-----------------|--------|--------|---------|
| **TRUE**      | TRUE   | FALSE  | UNKNOWN |
| **FALSE**     | FALSE  | FALSE  | FALSE   |
| **UNKNOWN**   | UNKNOWN| FALSE  | UNKNOWN |

### OR Operator
|       OR        | TRUE   | FALSE  | UNKNOWN |
|-----------------|--------|--------|---------|
| **TRUE**      | TRUE   | TRUE   | TRUE    |
| **FALSE**     | TRUE   | FALSE  | UNKNOWN |
| **UNKNOWN**   | TRUE   | UNKNOWN| UNKNOWN |

### NOT Operator
| x         | NOT x   |
|-----------|---------|
| TRUE      | FALSE   |
| FALSE     | TRUE    |
| UNKNOWN   | UNKNOWN |
