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

## Q2. Which countries have the most Invoices?

```sql
SELECT 
    billing_country, COUNT(billing_country) AS count
FROM
    invoice
GROUP BY billing_country
ORDER BY count DESC
LIMIT 1;
Q2. Which countries have the most Invoices?
sql
Copy code
SELECT 
    billing_country, COUNT(billing_country) AS count
FROM
    invoice
GROUP BY billing_country
ORDER BY count DESC
LIMIT 1;
Q3. What are top 3 values of total invoice?
sql
Copy code
SELECT 
    total
FROM
    invoice
ORDER BY total DESC
LIMIT 3;
Q4. Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. Write a query that returns one city that has the highest sum of invoice totals. Return both the city name & sum of all invoice totals
sql
Copy code
SELECT 
    billing_city, ROUND(SUM(total), 2) AS invoice_sum
FROM
    invoice
GROUP BY billing_city
ORDER BY invoice_sum DESC
LIMIT 1;
