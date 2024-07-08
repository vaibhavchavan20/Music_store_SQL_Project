# SQL Queries and Answers

**## Q1. Who is the senior most employee based on levels?**


SELECT 
    employee_id,
    CONCAT(last_name, ' ', first_name) AS full_name,
    title,
    levels
FROM
    employee
ORDER BY levels DESC
LIMIT 1;

**# Q2. Which countries have the most Invoices?**

```sql
SELECT 
    billing_country, COUNT(billing_country) AS count
FROM
    invoice
GROUP BY billing_country
ORDER BY count DESC
LIMIT 1;```

**#Q3. What are top 3 values of total invoice?**

```sql
SELECT 
    total
FROM
    invoice
ORDER BY total DESC
LIMIT 3;```

**#Q4. Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. Write a query that returns one city that has the highest sum of invoice totals. Return both the city name & sum of all invoice totals.**

```sql
SELECT 
    billing_city, ROUND(SUM(total), 2) AS invoice_sum
FROM
    invoice
GROUP BY billing_city
ORDER BY invoice_sum DESC
LIMIT 1;```

**#Q5. Who is the best customer? The customer who has spent the most money will be declared the best customer. Write a query that returns 3 person who have spent the most money.**

```sql
SELECT 
    customer.customer_id,
    customer.first_name,
    customer.last_name,
    ROUND(SUM(invoice.total), 2) AS bill
FROM
    customer
        INNER JOIN
    invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id , customer.first_name , customer.last_name
ORDER BY bill DESC
LIMIT 3;```

**#Q6. Write query to return the email, first name, last name, & Genre of all Rock Music listeners. Return your list ordered alphabetically by email starting with A.**

```sql
SELECT 
	DISTINCT(customer.email), customer.first_name, customer.last_name
FROM
    customer
        JOIN
    invoice ON customer.customer_id = invoice.customer_id
        JOIN
    invoice_line ON invoice.customer_id = invoice_line.invoice_id
WHERE
    track_id IN (SELECT 
            track_id
        FROM
            track
                JOIN
            genre ON track.genre_id = genre.genre_id
        WHERE
            genre.name = 'Rock')
ORDER BY email;```

**#Q7.  Let's invite the artists who have written the most rock music in our dataset. Write a query that returns the Artist name and total track count of the top 10 rock bands.**

```sql
SELECT 
    artist.artist_id,
    artist.name,
    COUNT(artist.artist_id) AS track_count
FROM
    track
        INNER JOIN
    album ON track.album_id = album.album_id
        INNER JOIN
    artist ON album.album_id = artist.artist_id
        INNER JOIN
    genre ON genre.genre_id = track.genre_id
WHERE
    genre.name LIKE 'Rock'
GROUP BY artist.artist_id , artist.name
ORDER BY track_count DESC
LIMIT 10;```


**#Q8. Return all the track names that have a song length longer than the average song length. Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first.**

```sql
SELECT 
    track.name, track.milliseconds
FROM
    track
WHERE
    track.milliseconds > (SELECT 
            AVG(track.milliseconds)
        FROM
            track)
ORDER BY track.milliseconds DESC;   ```

**#Q9. Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent.**


```sql
WITH cte AS (
		SELECT 
			artist.artist_id AS artist_id,
			SUM(invoice_line.unit_price * invoice_line.quantity) AS spent
		FROM
			invoice_line
				JOIN
			track ON track.track_id = invoice_line.track_id
				JOIN
			album ON album.album_id = track.album_id
				JOIN
			artist ON artist.artist_id = album.artist_id
		GROUP BY 1
		ORDER BY 2 DESC
		LIMIT 1 )
SELECT 
    customer.customer_id,
    customer.first_name,
    customer.last_name,
    cte.artist_id,
    SUM(invoice_line.quantity * invoice_line.unit_price) AS amount_spent
FROM  invoice INNER JOIN
    customer ON customer.customer_id = invoice.customer_id
        INNER JOIN
    invoice_line ON invoice_line.invoice_id = invoice.invoice_id
        INNER JOIN
    track ON track.track_id = invoice_line.track_id
        INNER JOIN
    album ON album.album_id = track.album_id
        INNER JOIN
    cte ON cte.artist_id = album.artist_id
GROUP BY customer.customer_id,
    customer.first_name,
    customer.last_name,
    cte.artist_id
ORDER BY amount_spent DESC;```


**#Q10. We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where the maximum number of purchases is shared return all Genres.**

```sql
WITH cte AS (
	SELECT 
    COUNT(invoice_line.quantity), 
    customer.country, 
    genre.name, 
    genre.genre_id,
	ROW_NUMBER() OVER(PARTITION BY customer.country 
		ORDER BY COUNT(invoice_line.quantity) DESC ) AS rowno
	FROM invoice_line
	JOIN invoice ON invoice.invoice_id=invoice_line.invoice_id
	JOIN customer ON customer.customer_id=invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC )
SELECT * FROM cte WHERE rowno <=1; ```

**#Q11. Write a query that determines the customer that has spent the most on music for each country. Write a query that returns the country along with the top customer and how much they spent. For countries where the top amount spent is shared, provide all customers who spent this amount.**

```sql
WITH RECURSIVE customer_with_country AS
	(SELECT
		customer.customer_id,
        first_name,
        last_name,
		billing_country,
        SUM(total) AS total_spending
		FROM customer 
        INNER JOIN invoice ON customer.customer_id=invoice.customer_id
        GROUP BY 1,2,3,4
        ORDER BY 2,3 DESC),
        
country_max_spending AS(
	SELECT billing_country,MAX(total_spending) AS max_spending
    FROM customer_with_country
    GROUP BY billing_country
    )
SELECT 
    cc.billing_country,
    cc.total_spending,
    cc.first_name,
    cc.last_name,
    cc.customer_id
FROM
    customer_with_country AS cc
        JOIN
    country_max_spending AS cms ON cc.billing_country = cms.billing_country
WHERE
    cc.total_spending = cms.max_spending
ORDER BY 1;```

