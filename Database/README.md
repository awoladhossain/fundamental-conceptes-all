# 🗄️ Database Learning Notes

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [SQL Basics](SQL%20Basics.md) | Basic SQL Queries | ✅ Done |
| [SQL Practice](SQL%20Practice.md) | Practice Problems | ✅ Done |
| [Complete SQL Guide](complete_sql_query_guide.md) | Comprehensive SQL Reference | ✅ Done |
| [Normalization](database_normalization_guide.md) | 1NF, 2NF, 3NF, BCNF | ✅ Done |
| [Relationships & Joins](database_relationships_keys_joins.md) | Keys, Joins, Relations | ✅ Done |
| [SQL Journey Story](sql_journey_story.md) | Learning Journey | ✅ Done |

## 🎯 Quick Reference

### Basic Queries
```sql
-- Select all
SELECT * FROM users;

-- Select with condition
SELECT name, email FROM users WHERE age > 18;

-- Insert data
INSERT INTO users (name, email) VALUES ('John', 'john@email.com');

-- Update data
UPDATE users SET name = 'Jane' WHERE id = 1;

-- Delete data
DELETE FROM users WHERE id = 1;
```

### JOIN Types
```sql
-- INNER JOIN
SELECT * FROM A INNER JOIN B ON A.id = B.a_id;

-- LEFT JOIN
SELECT * FROM A LEFT JOIN B ON A.id = B.a_id;

-- RIGHT JOIN
SELECT * FROM A RIGHT JOIN B ON A.id = B.a_id;
```

### Normalization Quick Notes
- **1NF**: No repeating groups, atomic values
- **2NF**: 1NF + No partial dependencies
- **3NF**: 2NF + No transitive dependencies
- **BCNF**: Every determinant is a candidate key

## 📚 Resources
- [W3Schools SQL](https://www.w3schools.com/sql/)
- [SQLBolt](https://sqlbolt.com/)
- [LeetCode Database](https://leetcode.com/problemset/database/)

---
*Last Updated: March 2026*
