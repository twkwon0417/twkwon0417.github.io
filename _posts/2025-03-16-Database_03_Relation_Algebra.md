---
title: 03. Relation Algebra
date: 2025-03-16 15:17:10 +9000
categories: [Konkuk_3-1, Database]
tags: [relation, algebra]     # TAG names should always be lowercase
---
## Relation Algebra

**Employees**

| EmpID | Name    | Age | DeptID |
|-------|---------|-----|--------|
| 1     | Alice   | 30  | D1     |
| 2     | Bob     | 25  | D2     |
| 3     | Charlie | 35  | D1     |

**Departments**

| DeptID | DeptName | Location      |
|--------|----------|---------------|
| D1     | HR       | New York      |
| D2     | IT       | San Francisco |

---

## Relational Algebra Operations

### 1. Selection (σ) (Duplicated Data should be removed)
_Select tuples that satisfy a condition._

**Example:** Select employees older than 30.  
**Expression:**  σ<sub>Age > 30</sub>(Employees) <br>
**Result:**

| EmpID | Name    | Age | DeptID |
|-------|---------|-----|--------|
| 3     | Charlie | 35  | D1     |

---

### 2. Projection (π) (Duplicated Data should be removed)
_Extract specific columns (attributes) from a relation._

**Example:** Project only the Name and Age attributes.  
**Expression:**  π<sub>Name, Age</sub>(Employees) <br>
**Result:**

| Name    | Age |
|---------|-----|
| Alice   | 30  |
| Bob     | 25  |
| Charlie | 35  |

---

### 3. Union (∪)
_Combine tuples from two union-compatible relations._

First, form two subsets of **Employees**:

- **R1:** Employees in department D1  
- R1 = σ<sub>DeptID = 'D1'</sub>(Employees)
- **R1 Result:**

| EmpID | Name    | Age | DeptID |
|-------|---------|-----|--------|
| 1     | Alice   | 30  | D1     |
| 3     | Charlie | 35  | D1     |

- **R2:** Employees younger than 30  
- R2 = σ<sub>Age < 30</sub>(Employees)
- **R2 Result:**

| EmpID | Name | Age | DeptID |
|-------|------|-----|--------|
| 2     | Bob  | 25  | D2     |

**Union Expression:** R1 ∪ R2
**Result:**

| EmpID | Name    | Age | DeptID |
|-------|---------|-----|--------|
| 1     | Alice   | 30  | D1     |
| 3     | Charlie | 35  | D1     |
| 2     | Bob     | 25  | D2     |

---

### 4. Set Difference (−)
_Return tuples in one relation that are not in the other._

**Example:** Employees in D1 that are **not** younger than 30 (using the same subsets).  
**Expression:** R1 − R2 <br>
**Result:**

| EmpID | Name    | Age | DeptID |
|-------|---------|-----|--------|
| 1     | Alice   | 30  | D1     |
| 3     | Charlie | 35  | D1     |
  
-----
### 5. Set Intersection (∩)
_Return tuples common to both relations._

**Example:** With our chosen subsets, there is no common tuple since no employee is both in D1 and younger than 30.  
**Expression:**  R1 ∩ R2 <br>
**Result:**  
*Empty relation*  
(If overlapping conditions were chosen, common tuples would be returned.)

---

### 6. Cartesian Product (×)
_Combine every tuple of one relation with every tuple of another._

**Expression:**  Employees × Departments <br>
**Result:**  
This produces 3 × 2 = 6 tuples. For example, one of the tuples is:  
---

### 7. Natural Join (⨝)
_Combine tuples from two relations based on common attribute(s)._

![natural-join.png](../assets/Konkuk_3-1/Database/natural-join.png)

**Expression:** Employees ⨝ Departments <br>
**Result:**  
The join is done on the common attribute **DeptID**. The result is:

| EmpID | Name    | Age | DeptID | DeptName | Location      |
|-------|---------|-----|--------|----------|---------------|
| 1     | Alice   | 30  | D1     | HR       | New York      |
| 3     | Charlie | 35  | D1     | HR       | New York      |
| 2     | Bob     | 25  | D2     | IT       | San Francisco |

---

### 8. Rename (ρ)
_Change the name of a relation or its attributes._
![rename-operator.png](../assets/Konkuk_3-1/Database/rename-operator.png)
**Example:** Rename **Employees** to **Emp** and change attribute **DeptID** to **Department**.  
**Expression:**  ρ<sub>Emp(EID, Name, Age, Department)</sub>(Employees) <br>
**Result:**  
The resulting relation **Emp** has attributes renamed as specified:

| EID | Name    | Age | Department |
|-----|---------|-----|------------|
| 1   | Alice   | 30  | D1         |
| 2   | Bob     | 25  | D2         |
| 3   | Charlie | 35  | D1         |

---
