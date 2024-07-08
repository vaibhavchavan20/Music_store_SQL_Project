# SQL Queries and Answers

## Q1. Who is the senior most employee based on levels?

```sql
SELECT 
    employee_id,
    CONCAT(last_name, ' ', first_name) AS full_name,
    title,
    levels
FROM
    employee
ORDER BY levels DESC
LIMIT 1;
