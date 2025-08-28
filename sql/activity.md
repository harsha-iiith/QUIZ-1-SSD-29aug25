# SQL Activity

# Activity 1

## 1. Create Database

```sql
CREATE DATABASE IF NOT EXISTS <database_name>;
```

```sql
SHOW DATABASES;
```

```sql
USE <database_name>;
```

## 2. Create Table

```sql
CREATE TABLE IF NOT EXISTS <table_name> (
    <column_name> <data_type> <constraints>,
    <column_name> <data_type> <constraints>,
    ...,
    PRIMARY KEY (<column_name>)
    FOREIGN KEY (<column_name>) REFERENCES <table_name> (<column_name>)
);
```

example:

```sql

CREATE TABLE IF NOT EXISTS students (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    age INT,
    email VARCHAR(255)
    FOREIGN KEY (class_id) REFERENCES classes(id)
);
```

## 3. Insert Data

```sql
INSERT INTO <table_name> (<column_name>, <column_name>, ...) VALUES
(<value>, <value>, ...),
(<value>, <value>, ...),
...
(<value>, <value>, ...);
```

example:

```sql
INSERT INTO students (id, name, age, email) VALUES
(1, 'John Doe', 20, 'john.doe@example.com'),
(2, 'Jane Smith', 21, 'jane.smith@example.com'),
(3, 'Jim Beam', 22, 'jim.beam@example.com');
```

## 4. Alter Table

```sql
ALTER TABLE <table_name> ADD <column_name> <data_type> <constraints> AFTER <column_name>;
```

example:

```sql
ALTER TABLE students ADD class_id INT AFTER id;
```

## 5. Select Data

```sql
SELECT <column_name>, <column_name>, ... FROM <table_name> WHERE <condition>;
```

example:

```sql
SELECT * FROM students WHERE age > 20;
```

# Activity 2 (cursor and stored procedures)

# what are these?

- Cursor: a pointer to a row in a result set

```sql
DECLARE <cursor_name> CURSOR FOR <query>;
OPEN <cursor_name>;
FETCH <cursor_name> INTO <variable_name>;
CLOSE <cursor_name>;
DEALLOCATE <cursor_name>;
```

example:

```sql

DECLARE subscriber_cursor CURSOR FOR SELECT SubscriberName FROM Subscribers;
OPEN subscriber_cursor;
FETCH subscriber_cursor INTO @sub_name;
CLOSE subscriber_cursor;
DEALLOCATE subscriber_cursor;
```

- Stored Procedure: a precompiled SQL statement that can be stored in the database and can be called by a client application

```sql

CREATE PROCEDURE <procedure_name>
BEGIN
    <SQL statements>
END;
```

example:

```sql
CREATE PROCEDURE fetch_students()
BEGIN
    SELECT * FROM students;
END;

CALL fetch_students();
```

## What is a DELIMITER?

In SQL, a delimiter is a character or sequence of characters that marks the end of a SQL statement.
By default, SQL uses semicolon (;) as the statement delimiter.

## Purpose of using DELIMITER:

When creating stored procedures, functions, or triggers that contain multiple SQL statements,
we need to change the delimiter temporarily so that semicolons within the procedure body
don't get interpreted as the end of the CREATE statement itself.

After the procedure definition, we restore the default delimiter with "DELIMITER ;"

## 0. Initialize database and tables like above and then

<details>
<summary>## 1. List Data (use cursor) and show the data in a table</summary>

```sql
-- Q1: ListAllSubscribers() - Stored procedure that uses a cursor to iterate through all Subscribers and prints their names
USE SQLLAB2;
DELIMITER //

CREATE PROCEDURE ListAllSubscribers()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE sub_name VARCHAR(100);
    DECLARE subscriber_cursor CURSOR FOR
        SELECT SubscriberName FROM Subscribers;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    OPEN subscriber_cursor;
    read_loop: LOOP
        FETCH subscriber_cursor INTO sub_name;
        IF done THEN
            LEAVE read_loop;
        END IF;
        SELECT  sub_name AS SubscriberInfo;
    END LOOP;
    CLOSE subscriber_cursor;
END //
DELIMITER ;
```
</details>

<details>
<summary>## 2. List Data (use cursor)</summary>

```sql
-- Q2: GetWatchHistoryBySubscriber(IN sub_id INT) - Returns all shows watched by the subscriber along with watch time

USE SQLLAB2;
DELIMITER //

CREATE PROCEDURE GetWatchHistoryBySubscriber(IN sub_id INT)
BEGIN
    SELECT
        s.Title AS ShowTitle,
        s.Genre,
        s.ReleaseYear,
        wh.WatchTime,
        sub.SubscriberName
    FROM WatchHistory wh
    JOIN Shows s ON wh.ShowID = s.ShowID
    JOIN Subscribers sub ON wh.SubscriberID = sub.SubscriberID
    WHERE wh.SubscriberID = sub_id
    ORDER BY wh.WatchTime DESC;
END //

DELIMITER ;
```
</details>

<details>
<summary>## 3. List Data (use cursor)</summary>

```sql
-- Q3: AddSubscriberIfNotExists(IN sub_name VARCHAR(100)) - Adds a new subscriber checking if the subscriber name already exists

USE SQLLAB2;
DELIMITER //
CREATE PROCEDURE AddSubscriberIfNotExists(IN sub_name VARCHAR(100))
BEGIN
    DECLARE subscriber_count INT DEFAULT 0;
    DECLARE new_subscriber_id INT;
    SELECT COUNT(*) INTO subscriber_count
    FROM Subscribers
    WHERE SubscriberName = sub_name;
    IF subscriber_count = 0 THEN
        SELECT COALESCE(MAX(SubscriberID), 0) + 1 INTO new_subscriber_id FROM Subscribers;
        INSERT INTO Subscribers (SubscriberID, SubscriberName, SubscriptionDate)
        VALUES (new_subscriber_id, sub_name, CURDATE());
        SELECT CONCAT('added ', sub_name) AS Result;
    ELSE
        SELECT CONCAT(sub_name, 'exists') AS Result;
    END IF;
END //
DELIMITER ;
```
</details>

<details>
<summary>## 4. List Data (use cursor)</summary>

```sql
-- Q4: SendWatchTimeReport() - Calls GetWatchHistoryBySubscriber() for all subscribers who have watched something

USE SQLLAB2;
DELIMITER //
CREATE PROCEDURE SendWatchTimeReport()
BEGIN
    DECLARE current_sub_id INT;
    DECLARE no_more_rows INT DEFAULT 0;
    DECLARE all_subs CURSOR FOR
        SELECT SubscriberID FROM Subscribers;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET no_more_rows = 1;
    OPEN all_subs;
    check_loop: LOOP
        FETCH all_subs INTO current_sub_id;
        IF no_more_rows = 1 THEN
            LEAVE check_loop;
        END IF;
        IF (SELECT COUNT(*) FROM WatchHistory WHERE SubscriberID = current_sub_id) > 0 THEN
            CALL GetWatchHistoryBySubscriber(current_sub_id);
        END IF;
    END LOOP;
    CLOSE all_subs;
END //

DELIMITER ;
```
</details>

<details>
<summary>## 5. List Data (use cursor)</summary>

```sql
-- Q5: Procedure with cursor that loops through each subscriber and calls GetWatchHistoryBySubscriber() to print their watch history
-- DELIMITER // changes the statement delimiter to // so semicolons inside the procedure don't end the CREATE statement

USE SQLLAB2;
DELIMITER //

CREATE PROCEDURE DisplayAllWatchHistory()
BEGIN
    DECLARE current_subscriber_id INT;
    DECLARE done INT DEFAULT 0;
    DECLARE subscriber_cursor CURSOR FOR
        SELECT SubscriberID FROM Subscribers;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
    OPEN subscriber_cursor;
    read_subscribers: LOOP
        FETCH subscriber_cursor INTO current_subscriber_id;
        IF done = 1 THEN
            LEAVE read_subscribers;
        END IF;
        SELECT SubscriberName AS `Active_Subscriber`
        FROM Subscribers
        WHERE SubscriberID = current_subscriber_id;
        CALL GetWatchHistoryBySubscriber(current_subscriber_id);
    END LOOP;
    CLOSE subscriber_cursor;
END //

DELIMITER ;
```
</details>
