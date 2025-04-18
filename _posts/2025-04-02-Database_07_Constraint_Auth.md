---
title: Integrity Constraints & Authorization
date: 2025-04-02 11:54:12 +9000
categories: [Konkuk_3-1, Database]
tags: [integrity, constraint, authorization, role]     # TAG names should always be lowercase
---

Integrity Constraints
==
Integrity constraints guard against accidental damage to the database, by ensuring the changes to the
database do not result in data inconsistency

Constraints on a Single Relation
--
- **not null**: 특정 attribute에 null값이 들어 올수 없도록 제한

- **primary key**: unique + not null

- **unique**: 특정 attribute의 모든 domain들이 unique해야함, in other word; attribute forms a candidate key (null 허용(PK랑 다른 점))

- **check(P), where P is a predicate**: Predicate P must be satisfied by every tuple in relation
  - Predicate안에 sub query 있어도 O.K.
```sql
create table section
    (course_id varchar (8),
    sec_id varchar (8),
    semester varchar (6),
    year numeric (4,0),
    building varchar (15),
    room_number varchar (7),
    id varchar (4),
    primary key (course_id, sec_id, semester, year),
    check (semester in ('Fall', 'Winter', 'Spring', 'Summer'))
)
```

Referential Integrity
--
특정 relation의 특정 attribute가 다른 relation에도 존재하게 제한을 둘 수 있다. 
```sql
-- Department table: Each department is uniquely identified by dept_name.
CREATE TABLE Department (
    dept_name VARCHAR(50) NOT NULL,
    building   VARCHAR(50),
    budget     DECIMAL(10,2),
    PRIMARY KEY (dept_name)
);

-- Instructor table: Each instructor is uniquely identified by instructor_id.
-- The dept_name in the Instructor table is a foreign key that references the primary key in the Department table.
CREATE TABLE Instructor (
    instructor_id INT NOT NULL,
    name          VARCHAR(100),
    dept_name     VARCHAR(50),
    PRIMARY KEY (instructor_id),
    FOREIGN KEY (dept_name) REFERENCES Department(dept_name)
    -- FOREIGN KEY (dept_name) REFERENCES Department // 이렇게도 알아먹는다. 
);
```
- Referential integrity constraint를 위반하면 dbms가 reject함
- 맨날 이러면 수정, 삭제하기 짜증 e.g. 위의 경우 dept_name을 수정하려면 두 table모두에서 작업이 이루어 져야한다.
- Delete과 Update의 경우 다음과 같은 문법 제공
  - **Cascade**: 다 전파시켜
  - **Set null**: null로
  - **Set default**: 기본값으로(기본값은 우리가 설정)
```sql
-- Create the Parent table with a primary key.
CREATE TABLE Parent (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- Create the Child table with three foreign key columns.
-- Note: setdefault_fk has a default value so it can be set to that when the referenced parent row is deleted.
CREATE TABLE Child (
    child_id INT PRIMARY KEY,
    cascade_fk INT,       -- Will use ON DELETE CASCADE
    setnull_fk INT,       -- Will use ON DELETE SET NULL
    setdefault_fk INT DEFAULT 1,  -- Will use ON DELETE SET DEFAULT
    FOREIGN KEY (cascade_fk) REFERENCES Parent (id) ON DELETE CASCADE,
    FOREIGN KEY (setnull_fk) REFERENCES Parent (id) ON DELETE SET NULL,
    FOREIGN KEY (setdefault_fk) REFERENCES Parent (id) ON DELETE SET DEFAULT
);
```

Assertions
--
내가 아는 그 assertion 맞습니당.
```sql
CREATE ASSERTION positive_balance_assertion
CHECK (
    NOT EXISTS (
        SELECT *
        FROM Account
        WHERE balance < 0
    )
);
```
- CREATE ASSERTION <assertion-name> CHECK (<predicate>)
- Database 전제에 적용
- Operation이 발생 할때마다 Assertion이 실행되서 만족하지 않을시 바로 reject

Built In Data Type
--
- **date:**
  - Stores dates containing a 4-digit year, month, and day.
  - e.g. `date '2005-7-27'`

- **time:**
  - Represents the time of day (hours, minutes, seconds).
  - e.g.`time '09:00:30'` or `time '09:00:30.75'`

- **timestamp:**
  - Combines a date and a time of day.
  - e.g`timestamp '2005-7-27 09:00:30.75'`

- **interval:**
  - Represents a period of time.
  - e.g.`interval '1' day`
  - Subtracting two date/time/timestamp values produces an interval.
  - Interval values can be added to date/time/timestamp values.

- **blob (Binary Large Object):**
  - A large collection of uninterpreted binary data.
  - The actual interpretation is managed by an external application.

- **clob (Character Large Object):**
  - A large collection of character data.

- When a query returns a large object, it returns a pointer rather than the object data itself.

User-Defined Types VS Domains
--
- User-Defined Types: **Integrity Constraints 적용 불가능**
- Domains : **Integrity Constraints 적용 가능**

### User-Defined Types
CREATE TYPE Dollars AS numeric(12,2) FINAL (end of the definition 명시해줌(not sure))

### Domains
CREATE DOMAIN person_name char(20) NOT NULL
```sql
create domain degree_level varchar(10)
constraint degree_level_test
check (value in ('Bachelors', 'Masters', 'Doctorate'));
```
- The constraint Keyword: After defining the domain, the constraint keyword is used to specify a constraint on this domain.
  - Domain이라 Integrity Constraints 적용 가능해서 CONSTRAINTS keyword쓸수 있는 거야! User-Defined Types는 적용 불가능 

Index Creation
--
- Database가 table을 full scan 떄리는건 시간 땅바닥에 버리기
- 인덱스를 사용하면 데이터베이스가 전체 테이블을 스캔하는 대신, 바로 원하는 행으로 접근
- 어떤 attribute를 기준으로 table을 building 할것인가
- index로 정한 attribute를 많이 사용하는 query가 나와야 효율적이게 된다. 
`CREATE INDEX <name> ON <relation-name> (attribute);`

Authorizations
==
Database를 사용하는 여러 user의 권한을 관리하자

Privilege
--
Read -> Insert -> Update -> Delete 순으로 권한의 정도가 강하다. 이것들을 privilege라고 해
- 다음과 같은 privilege들도 존재
  - Index: allows creation and deletion of indices
  - Resources: allows creation of new relations
  - Alteration: allows addition or deletion of attributes in a relation
  - Drop: allows deletion of relations
- Table뿐만 아니라 View에도 authorization 적용 가능
  - View와 Table에 대한 권한은 별개!! View에 대한 권한을 얻었다고 table에 대한 권한을 얻는 것은 아니다. 

Granting, Revoking authorization
--
- GRANT <privilege list> ON <relation or view> TO <user list>
- REVOKE <privilege list> ON <relation or view> FROM <user list> 
```sql
-- Example: Granting specific privileges on a table named "Employees" to a list of users.
GRANT SELECT, INSERT, UPDATE
ON Employees
TO user1, user2, user3;

-- Example: Revoking specific privileges on the table "Employees" from a user.
REVOKE INSERT, UPDATE
ON Employees
FROM user2;
```
- revoke에 <user list>에 **public**이 있으면 따로(explicitly) 그 privilege를 부여받은 사람 제외 모두가 권한을 잃어
- A, B, C 셋 에게 privilege를 grant 받은 경우 A, B, C모두에게 revoke 당해야, privilege가 revoke 됨 (이런 상황 해결 => role사용하면 이럴일 없음)
- A가 admin한테 권한을 받고, A가 B한테 권한을 주고, B가 C에게 권한을 준 상황을 가정하자. 만약 A가 admin에게 권한을 revoke 당하면 B, C도 권한을 revoke 당한다. (chain of privilege(?))

Roles
--
Main 기능: Authorization을 그룹화해서 권한, 유저들을 관리하게 쉽게 하자 
```sql
CREATE ROLE Manager;
GRANT SELECT, INSERT, UPDATE ON Employee TO Manager;
GRANT Manager TO alice;
REVOKE Manager FROM alice;
REVOKE SELECT, INSERT, UPDATE ON Employee FROM Manager;
```
- create role <name>
- grant <privilege list> on <table> to <role>
- grant <role> to <users>
- 얘도 chain of role 됨 (hierarchy 존재)
