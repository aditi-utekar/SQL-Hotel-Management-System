# Hotel Management System - SQL Project

## 📌 Project Overview
This project analyzes hotel booking data using SQL.

## 🛠 Tools Used
- MySQL Workbench


## 📊 Set up the Hotel Management System Database: Create and populate the database with tables for hotels, bookings, customers, rooms, payments.
CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
CTAS (Create Table As Select): Utilize CTAS to create new tables based on query results.
Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.

## 🚀 Skills Demonstrated
- Joins
- Aggregations
- Group By & Having
- Subqueries
- CTE
- Window Functions

## Database Setup
<img width="921" height="644" alt="Screenshot" src="https://github.com/user-attachments/assets/684af944-cc2c-4c38-9eaa-1c4d3a13f267" />

```sql
create database Hotel_mang;
-- Hotel Management System Project

Create table Hotels(
     hotel_id INT PRIMARY KEY,
     hotel_name varchar(20) Not Null,
     city varchar(25) Not Null,
     rating decimal(2,1) check (rating between 1 And 5)
);     

Create table Rooms(
      room_id INT PRIMARY KEY,
      hotel_id INT Not Null,
      room_type varchar(25) not null,
      price_per_night decimal(10,2) not null,
      status varchar(20) Default 'Available'
);      
    
Create table Customers(
      customer_id INT PRIMARY KEY,
      name varchar(50) Not Null,
      email varchar(50) Unique,
      phone varchar(15),
      city varchar(50)
);      

Create table bookings(
      booking_id INT PRIMARY KEY,
      customer_id INT Not Null,
      room_id INT Not Null,
      check_in_date Date Not Null, 
	  check_out_date Date Not Null,
      booking_date Date Not Null,
      total_amount Decimal(12,2),
      booking_status varchar(20) Default 'Confirmed'
);      

Create table Payments(
      payment_id INT PRIMARY KEY,
      booking_id INT UNIQUE,
      payment_date Date,
      payment_method varchar(50),
      payment_status varchar(20)
);      
show tables;

-- Assigning FK
Alter table rooms
Add constraint fk_rooms_hotel
FOREIGN KEY (hotel_id)
References hotels(hotel_id)
ON DELETE CASCADE
ON UPDATE CASCADE
;

Alter table bookings
Add constraint fk_booking_room
FOREIGN KEY (room_id)
References rooms(room_id)
ON DELETE CASCADE
ON UPDATE CASCADE
;     

Alter table bookings
Add constraint fk_booking_customer
FOREIGN KEY (customer_id)
References customers(customer_id)
ON DELETE CASCADE
ON UPDATE CASCADE
;

Alter table payments
Add constraint fk_payment_booking
FOREIGN KEY (booking_id)
References bookings(booking_id)
ON DELETE CASCADE
ON UPDATE CASCADE
;
DESC hotels;
DESC rooms;
DESC customers;
DESC bookings;
DESC payments;

-- Business Analysis & Findings


-- Write a query to find all confirmed bookings
SELECT *
FROM bookings
WHERE booking_status = 'Confirmed';


-- Write a query to Find bookings of a particular customer
SELECT * 
FROM bookings
WHERE customer_id = 1;


-- write a query to find total no of bookings
SELECT 
   count(*) As total_bookings
FROM bookings;


-- Write a query to find Revenue per Hotel
SELECT 
    h.hotel_name,
    SUM(b.total_amount) AS total_revenue
FROM bookings b
JOIN rooms r ON b.room_id = r.room_id
JOIN hotels h ON r.hotel_id = h.hotel_id
   WHERE b.booking_status = 'Completed'   -- optional, to consider only completed bookings
   GROUP BY h.hotel_name
   ORDER BY total_revenue DESC;
 

-- Write a query to Find Top 3 highest rated hotels
SELECT h.hotel_name,  
       AVG(r.rating) AS avg_rating 
FROM hotels r
JOIN hotels h ON r.hotel_id = h.hotel_id
     GROUP BY h.hotel_name
     ORDER BY avg_rating DESC
LIMIT 3;     


-- Write a query to calculate total cost when a customer books multiple rooms
SELECT 
    r.room_id,
    r.price_per_night * DATEDIFF(b.check_out_date, b.check_in_date) AS total_booking_rate
FROM rooms r
JOIN bookings b 
    ON b.room_id = r.room_id
WHERE b.booking_status != 'Cancelled'
GROUP BY b.booking_id;


-- Write a query to categorize hotel of basis of Good,Average & Excellent
SELECT 
    h.hotel_name,
    AVG(r.rating) AS avg_rating,
    CASE
        WHEN AVG(r.rating) >= 4.5 THEN 'Excellent'
        WHEN AVG(r.rating) >= 4.1 THEN 'Good'
        ELSE 'Average'
    END AS rating_category
FROM Hotels r
JOIN hotels h ON r.hotel_id = h.hotel_id
GROUP BY h.hotel_name
ORDER BY avg_rating DESC;


-- Write a query to find Room Types With Average Price Greater Than 7000
SELECT 
    room_type,
    AVG(price_per_night) AS avg_price
FROM rooms
GROUP BY room_type
HAVING AVG(price_per_night) > 7000;


-- Write a query to find monthly revenue rank per hotel
SELECT
     h.hotel_name,
     MONTH(b.booking_date) AS month,
     SUM(b.total_amount) AS monthly_revenue,
     RANK() 
     OVER(PARTITION BY h.hotel_name
         ORDER BY SUM(b.total_amount) DESC
         )AS monthly_rank
FROM bookings b
JOIN rooms r ON b.room_id = r.room_id
JOIN hotels h ON r.hotel_id = h.hotel_id
         GROUP BY h.hotel_name, 
         MONTH(b.booking_date)
;        


-- Write a Query to find Monthly Revenue Growth Analysis
WITH monthly_revenue AS (
    SELECT 
    DATE_FORMAT(booking_date, '%Y-%M') AS month,
      SUM(total_amount) AS revenue
    FROM bookings
    WHERE booking_status = 'Confirmed'
    GROUP BY date_format(booking_date, '%Y-%M')
    )
SELECT month,
       revenue,
       LAG(revenue) 
       OVER(order by month) AS previous_month,
       revenue - LAG(revenue) OVER (order by month) AS growth
FROM monthly_revenue; 


-- Create a table of confirmed bookings
CREATE TABLE confirmed_bookings AS
    SELECT *
    FROM bookings
WHERE booking_status = 'Confirmed';    
SELECT * FROM confirmed_bookings;


/* Write a query to find which customers are repeat customers more than 3
CITAS with CTE */
CREATE TABLE repeat_customers AS
WITH customer_booking_count AS (
    SELECT customer_id,
           COUNT(*) AS total_bookings
    FROM bookings
    GROUP BY customer_id
)
SELECT *
FROM customer_booking_count
WHERE total_bookings > 3;
SELECT * FROM repeat_customers;

-- END OF THE PROJECT
```
