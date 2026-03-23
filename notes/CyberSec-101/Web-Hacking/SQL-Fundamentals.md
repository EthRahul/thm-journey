# SQL Fundamentals
**Path:** CyberSec_101 - Web Hacking
**Date:** 2026-03-14
**Difficulty:** Easy

---

## Databases 101

- Database → organized collection of structured data
- DBMS (Database Management System) → software to manage databases
- Two types:
  - Relational (SQL) → data in tables with relationships → MySQL, PostgreSQL, SQLite
  - Non-relational (NoSQL) → flexible structure → MongoDB, Redis
- Table → rows (records) and columns (fields)
- Primary Key → unique identifier for each row
- Foreign Key → links one table to another

---

## SQL Basics

- SQL = Structured Query Language
- Used to interact with relational databases
- Case insensitive but convention is UPPERCASE for keywords
- Every statement ends with semicolon ;
- Comments:
```sql
-- This is a single line comment
/* This is a
   multi line comment */
```

---

## Database and Table Statements
```sql
-- Create a database
CREATE DATABASE mydb;

-- Select database to use
USE mydb;

-- Show all databases
SHOW DATABASES;

-- Show all tables
SHOW TABLES;

-- Create a table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    role VARCHAR(20) DEFAULT 'user'
);

-- Show table structure
DESCRIBE users;

-- Delete a table
DROP TABLE users;

-- Delete a database
DROP DATABASE mydb;

-- Modify table - add column
ALTER TABLE users ADD COLUMN age INT;

-- Modify table - remove column
ALTER TABLE users DROP COLUMN age;
```

---

## CRUD Operations

CRUD = Create Read Update Delete → core of all database interaction
```sql
-- CREATE → INSERT
INSERT INTO users (username, password, email, role)
VALUES ('rahul', 'pass123', 'rahul@email.com', 'admin');

-- INSERT multiple rows
INSERT INTO users (username, password) VALUES
('alice', 'pass1'),
('bob', 'pass2');

-- READ → SELECT
SELECT * FROM users;                          -- all columns
SELECT username, email FROM users;            -- specific columns
SELECT * FROM users WHERE role = 'admin';     -- with condition
SELECT * FROM users WHERE id = 1;

-- UPDATE
UPDATE users SET password = 'newpass' WHERE id = 1;
UPDATE users SET role = 'admin' WHERE username = 'rahul';

-- DELETE
DELETE FROM users WHERE id = 1;
DELETE FROM users WHERE username = 'bob';

-- DANGER - no WHERE clause affects ALL rows
UPDATE users SET password = 'hacked';  -- changes everyone
DELETE FROM users;                     -- deletes everyone
```

---

## Clauses
```sql
-- WHERE → filter rows
SELECT * FROM users WHERE role = 'admin';

-- ORDER BY → sort results
SELECT * FROM users ORDER BY username ASC;   -- A to Z
SELECT * FROM users ORDER BY id DESC;        -- highest first

-- LIMIT → restrict number of results
SELECT * FROM users LIMIT 5;
SELECT * FROM users LIMIT 5 OFFSET 10;      -- skip first 10

-- DISTINCT → remove duplicate values
SELECT DISTINCT role FROM users;

-- GROUP BY → group rows by column value
SELECT role, COUNT(*) FROM users GROUP BY role;

-- HAVING → filter after GROUP BY (like WHERE for groups)
SELECT role, COUNT(*) FROM users 
GROUP BY role 
HAVING COUNT(*) > 2;

-- LIKE → pattern matching
SELECT * FROM users WHERE username LIKE 'a%';   -- starts with a
SELECT * FROM users WHERE email LIKE '%@gmail%'; -- contains @gmail
SELECT * FROM users WHERE username LIKE '_ob';   -- 3 chars ending ob

-- BETWEEN → range filter
SELECT * FROM users WHERE id BETWEEN 1 AND 5;

-- IN → match multiple values
SELECT * FROM users WHERE role IN ('admin', 'moderator');

-- IS NULL / IS NOT NULL
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE email IS NOT NULL;
```

---

## Operators
```sql
-- Comparison operators
=     equal
!=    not equal (also <>)
>     greater than
<     less than
>=    greater than or equal
<=    less than or equal

-- Logical operators
AND   → both conditions must be true
OR    → either condition must be true
NOT   → reverses the condition

-- Examples
SELECT * FROM users WHERE role = 'admin' AND id > 5;
SELECT * FROM users WHERE role = 'admin' OR role = 'moderator';
SELECT * FROM users WHERE NOT role = 'guest';

-- SQLi relevance - OR 1=1 always returns true
SELECT * FROM users WHERE username = '' OR 1=1-- ';
```

---

## Functions
```sql
-- Aggregate functions
COUNT(*) → count number of rows
SUM(column) → total of numeric column
AVG(column) → average of numeric column
MAX(column) → highest value
MIN(column) → lowest value

-- Examples
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM users WHERE role = 'admin';
SELECT MAX(id) FROM users;
SELECT MIN(id) FROM users;

-- String functions
UPPER(str) → convert to uppercase
LOWER(str) → convert to lowercase
LENGTH(str) → character count
SUBSTRING(str, start, length) → extract part of string
CONCAT(str1, str2) → join strings
REPLACE(str, old, new) → replace text
TRIM(str) → remove leading/trailing spaces

-- Examples
SELECT UPPER(username) FROM users;
SELECT LENGTH(password) FROM users;
SELECT CONCAT(username, '@email.com') FROM users;
SELECT SUBSTRING(password, 1, 3) FROM users; -- first 3 chars

-- Numeric functions
ROUND(num, decimals) → round number
FLOOR(num) → round down
CEIL(num) → round up
ABS(num) → absolute value

-- Database info functions (critical for SQLi)
DATABASE() → returns current database name
VERSION() → returns database version
USER() → returns current database user
@@hostname → server hostname
```

---

## JOIN (Extra — Critical for SQLi)
```sql
-- INNER JOIN → rows matching in both tables
SELECT users.username, orders.product
FROM users
INNER JOIN orders ON users.id = orders.user_id;

-- Used in UNION SQLi to extract data from other tables
SELECT username, password FROM users
UNION
SELECT table_name, column_name FROM information_schema.columns;
```

---

## information_schema (Very Important for SQLi)
```sql
-- Built in database that stores metadata about all databases
-- Used heavily in SQLi attacks to enumerate databases/tables/columns

-- List all databases
SELECT schema_name FROM information_schema.schemata;

-- List all tables in a database
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'mydb';

-- List all columns in a table
SELECT column_name FROM information_schema.columns
WHERE table_name = 'users';
```

---

## Gotchas / Surprises

- No WHERE in UPDATE or DELETE = affects entire table
- LIKE is case insensitive by default in MySQL
- % matches any number of characters in LIKE
- _ matches exactly one character in LIKE
- Single quotes are used for string values not double quotes
- -- (double dash + space) comments out rest of query
  this is the most used SQLi comment character
- OR 1=1 always evaluates to true — foundation of SQLi
- information_schema exists in every MySQL database
  and is the main target during SQL injection enumeration

---

## My Understanding in Plain English

SQL is how you talk to a database. SELECT gets data,
INSERT adds data, UPDATE changes data, DELETE removes it.
As a hacker you inject SQL commands into user inputs
to manipulate these queries and extract data the
application never intended to show you.
OR 1=1 and UNION SELECT are the two most powerful
concepts in SQL injection.

---

## Related Notes

- [[SQLi]]
- [[Web Application Basics]]
- [[Burp Suite]]
- [[OWASP Top 10]]
