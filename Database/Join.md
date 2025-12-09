# Database Joins

**_SQL joins are fundamental tools for combining data from multiple tables in relational databases._**

## Inner Join

- Returns only rows where there is a match in both tables.

**Students**

| student_id | name  |
| ---------- | ----- |
| 1          | Rahim |
| 2          | Karim |
| 3          | Salma |

**StudentCourses**

| student_id | course_id |
| ---------- | --------- |
| 1          | 101       |
| 2          | 102       |

**Courses**

| course_id | course_name |
| --------- | ----------- |
| 101       | Math        |
| 102       | English     |
| 103       | Physics     |

**SQL Query (INNER JOIN)**

- First example

```
SELECT s.name, c.course_name
FROM Students s
INNER JOIN StudentCourses sc ON s.student_id = sc.student_id
INNER JOIN Courses c ON sc.course_id = c.course_id;
```

- Step 1: Students joins with StudentCourses → only students who have course entries remain.
- Step 2: That result joins with Courses → only courses that exist in the Courses table remain.

**Final Result:**

```
name course_name
Rahim Math
Karim English
```

- 👉 Notice: Salma is missing because she has no entry in StudentCourses.
- 👉 Physics is missing because no student is enrolled in it.

**❌ What if there is NO match?**

- INNER JOIN returns nothing if no rows match.
- Example: If StudentCourses was empty, the query result would be an empty set (no rows at all).
- Unlike LEFT JOIN, INNER JOIN does not keep unmatched rows.

- 2nd Example

```
SELECT s.student_id, s.name, sc.course_id
FROM Students s
INNER JOIN StudentCourses sc
ON s.student_id = sc.student_id;
```

**🔎 Result of the Query**

```
student_id name course_id
1 Rahim 101
2 Karim 102
```

**📌 Explanation**

- The INNER JOIN matches rows where Students.student_id = StudentCourses.student_id.
- Rahim and Karim appear because they have matching entries in StudentCourses.
- Salma does not appear because she has no matching record in StudentCourses.

**❌ What if there is no match?**

> If a student has no course (like Salma), they are excluded.
> If StudentCourses were empty, the result would be empty set (no rows at all).

- 👉 That’s the key difference: INNER JOIN only shows rows with matches.

## LEFT JOIN

**_A LEFT JOIN returns all rows from the left table, along with matching rows from the right table. If there is no match, NULL values are returned for columns from the right table. LEFT JOIN is also known as LEFT OUTER JOIN._**

**🔹 LEFT JOIN Query**

```
SELECT s.student_id, s.name, sc.course_id
FROM Students s
LEFT JOIN StudentCourses sc
ON s.student_id = sc.student_id;
```

**🔎 Result of the Query**

```
student_id name course_id
1 Rahim 101
2 Karim 102
3 Salma NULL
```

**📌 Explanation**

> LEFT JOIN returns all rows from the left table (Students).
> If a student has a matching record in StudentCourses, the course_id is shown.
> If there is no match, SQL fills the columns from the right table with NULL.
> That’s why Salma appears with NULL for course_id.

**❌ What if there are no matches at all?**

- If StudentCourses is empty, the query still returns all students, but the course_id column will be NULL for each.
  > Example result if StudentCourses is empty:

```
student_id name course_id
1 Rahim NULL
2 Karim NULL
3 Salma NULL
```

## SQL RIGHT JOIN

**_RIGHT JOIN returns all the rows of the table on the right side of the join and matching rows for the table on the left side of the join. It is very similar to LEFT JOIN for the rows for which there is no matching row on the left side, the result-set will contain null. RIGHT JOIN is also known as RIGHT OUTER JOIN._**

```
Students

student_id name
1 Rahim
2 Karim
3 Salma

StudentCourses

student_id course_id
1 101
2 102
4 103 ← notice: student_id 4 doesn’t exist in Students
```

**RIGHT JOIN Query**

```
SELECT s.student_id, s.name, sc.course_id
FROM Students s
RIGHT JOIN StudentCourses sc
ON s.student_id = sc.student_id;
```

**🔎 Result of the Query**

```
student_id name course_id
1 Rahim 101
2 Karim 102
NULL NULL 103
```

**📌 Explanation**

- RIGHT JOIN returns all rows from the right table (StudentCourses).
- If a match exists in the left table (Students), it shows the data.
- If there is no match, SQL fills the left table’s columns with NULL.
- That’s why student_id = 4 in StudentCourses appears, but with NULL for student info.

**❌ What if there are no matches at all?**

- If no student IDs match, the query still returns all rows from StudentCourses, but the student info columns will be NULL.

**Example result if no matches exist:**

```
student_id name course_id
NULL NULL 101
NULL NULL 102
```

## SQL FULL JOIN

**_FULL JOIN creates the result-set by combining results of both LEFT JOIN and RIGHT JOIN. The result-set will contain all the rows from both tables. For the rows for which there is no matching, the result-set will contain NULL values._**

**🔹 FULL JOIN Query**

```
SELECT s.student_id, s.name, sc.course_id
FROM Students s
FULL OUTER JOIN StudentCourses sc
ON s.student_id = sc.student_id;
```

**🔎 Result of the Query**

```
student_id name course_id
1 Rahim 101
2 Karim 102
3 Salma NULL
4 NULL 103
```

**📌 Explanation**

> FULL JOIN returns all rows from both tables.
> If there’s a match → shows combined data.
> If there’s no match on one side → fills missing columns with NULL.

- That’s why:
  > Salma (student_id 3) appears with NULL for course_id.
  > StudentCourses entry with student_id 4 appears with NULL for student info.

**❌ What if there are no matches at all?**
FULL JOIN still returns all rows from both tables, but unmatched columns will be NULL.

**Example if no student matches any course:**

```
student_id name course_id
1 Rahim NULL
2 Karim NULL
3 Salma NULL
NULL NULL 101
NULL NULL 102
```

## SQL SELF JOIN

**_A self join is a type of SQL join that allows a table to be joined with itself, enabling comparisons between rows within the same table. This is useful for finding relationships or hierarchical data, such as employees and their managers._**

**Employees**

```
employee_id name manager_id
1 Rahim NULL
2 Karim 1
3 Salma 1
4 Hasan 2
```

**🔹 SELF JOIN Query**

```
SELECT e.employee_id, e.name AS employee, m.name AS manager
FROM Employees e
LEFT JOIN Employees m
ON e.manager_id = m.employee_id;
```

**🔎 Result of the Query**

```
employee_id employee manager
1 Rahim NULL
2 Karim Rahim
3 Salma Rahim
4 Hasan Karim
```

**📌 Explanation**

- A SELF JOIN means joining a table to itself.
- Here, Employees is joined to itself:

  > e = employee alias.
  > m = manager alias.

- The join condition e.manager_id = m.employee_id links each employee to their manager.
- Rahim has no manager (NULL).
- Karim and Salma report to Rahim.
- Hasan reports to Karim.

**❌ What if there is no match?**

- If an employee has no manager (manager_id = NULL), the manager column will show NULL.
- That’s why Rahim appears with NULL as manager.

## Natural Join

**Definition: A join that automatically matches columns with the same name in both tables.**

- Key point: You don’t explicitly write the ON condition — SQL figures it out by column names.

**Example:**

```
Students

student_id name
1 Rahim
2 Karim
StudentCourses

student_id course_id
1 101
2 102
```

**SQL QUERY**

```
SELECT *
FROM Students
NATURAL JOIN StudentCourses;
```

**👉 SQL automatically joins on student_id because it exists in both tables. 👉 Result:**

```
student_id name course_id
1 Rahim 101
2 Karim 102
```
