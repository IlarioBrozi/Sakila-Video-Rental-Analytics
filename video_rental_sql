/*
=======================================================
Portfolio Project: Sakila Video Rental Analytics
Author: Ilario Brozi
Date: 15/04/2026
Description: below queries for KPI, customer insights, 
film performance, data cleaning e feature table
=======================================================
*/

-- ======================================================
-- 1. Store KPI: revenue, rentals, avg_ticket, pct_total
-- ======================================================
WITH store_revenue AS (
    SELECT s.store_id,
           SUM(p.amount) AS total_revenue,
           COUNT(r.rental_id) AS total_rentals,
           AVG(p.amount) AS avg_ticket
    FROM store s
    LEFT JOIN customer c ON s.store_id = c.store_id
    LEFT JOIN rental r ON c.customer_id = r.customer_id
    LEFT JOIN payment p ON r.rental_id = p.rental_id
    GROUP BY s.store_id
)
SELECT *,
       total_revenue / SUM(total_revenue) OVER () AS pct_total_revenue
FROM store_revenue
ORDER BY total_revenue DESC;

-- ======================================================
-- 2. Top 10 customer per revenue
-- ======================================================
SELECT c.customer_id,
       c.first_name,
       c.last_name,
       SUM(p.amount) AS total_spent,
       RANK() OVER (ORDER BY SUM(p.amount) DESC) AS revenue_rank
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY revenue_rank
LIMIT 10;

-- ======================================================
-- 3. Customers segmentation (VIP/Standard/Low)
-- ======================================================
WITH customer_kpi AS (
    SELECT c.customer_id,
           c.first_name,
           c.last_name,
           COUNT(r.rental_id) AS total_rentals,
           SUM(p.amount) AS total_spent
    FROM customer c
    LEFT JOIN rental r ON c.customer_id = r.customer_id
    LEFT JOIN payment p ON r.rental_id = p.rental_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT *,
       CASE
           WHEN total_spent > 100 AND total_rentals > 20 THEN 'VIP'
           WHEN total_spent BETWEEN 50 AND 100 THEN 'Standard'
           ELSE 'Low'
       END AS segment
FROM customer_kpi
ORDER BY total_spent DESC;

-- ======================================================
-- 4. Top 3 film per category
-- ======================================================
SELECT *
FROM (
    SELECT f.title,
           c.name AS category,
           COUNT(r.rental_id) AS rentals_count,
           RANK() OVER (PARTITION BY c.category_id ORDER BY COUNT(r.rental_id) DESC) AS rank_in_category
    FROM film f
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
    LEFT JOIN inventory i ON f.film_id = i.film_id
    LEFT JOIN rental r ON i.inventory_id = r.inventory_id
    GROUP BY f.film_id, c.category_id
) t
WHERE rank_in_category <= 3
ORDER BY category, rank_in_category;

-- ======================================================
-- 5. Monthly revenue trend per store
-- ======================================================
WITH monthly_revenue AS (
    SELECT s.store_id,
           YEAR(r.rental_date) AS year,
           MONTH(r.rental_date) AS month,
           SUM(p.amount) AS revenue,
           COUNT(r.rental_id) AS rentals
    FROM rental r
    JOIN customer c ON r.customer_id = c.customer_id
    JOIN store s ON c.store_id = s.store_id
    JOIN payment p ON r.rental_id = p.rental_id
    GROUP BY s.store_id, YEAR(r.rental_date), MONTH(r.rental_date)
)
SELECT *,
        round((revenue - LAG(revenue) OVER (PARTITION BY store_id ORDER BY year, month))/LAG(revenue) OVER (PARTITION BY store_id ORDER BY year, month) * 100,2) AS revenue_change
FROM monthly_revenue
ORDER BY store_id, year, month;

-- ======================================================
-- 6. Data cleaning e validation
-- ======================================================
-- Films that have never been rented
SELECT f.film_id, f.title
FROM film f
LEFT JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
WHERE r.rental_id IS NULL;

-- Rentals with incoherent dates
SELECT *
FROM rental
WHERE return_date < rental_date;

-- Duplicated customers (first name + last name)
SELECT first_name, last_name, COUNT(*) AS count_dup
FROM customer
GROUP BY first_name, last_name
HAVING COUNT(*) > 1;

-- ======================================================
-- 7. customer feature table per BI/Data Science
-- ======================================================
WITH customer_features AS (
    SELECT c.customer_id,
           COUNT(r.rental_id) AS total_rentals,
           SUM(p.amount) AS total_spent,
           AVG(p.amount) AS avg_ticket,
           DATEDIFF(NOW(), MAX(r.rental_date)) AS days_since_last_rental
    FROM customer c
    LEFT JOIN rental r ON c.customer_id = r.customer_id
    LEFT JOIN payment p ON r.rental_id = p.rental_id
    GROUP BY c.customer_id
)
SELECT *
FROM customer_features;
