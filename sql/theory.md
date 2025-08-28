# Structured Databases

## **Data**

It is referred to as a collection of values that provide some information.

## **Entity Relationship Data Model**

The Entity-Relationship (ER) model is a critical concept in structured databases, primarily used for designing and visualizing the structure of a database.

The ER model consists of three main components:

- **Entities**:
- **Attributes**:
- **Relationships**:

## SQL - CURD Operations

This page covers basic CURD (Create, Read, Update, Delete) and JOINs

### SQL commands

#### Data Definition Language (DDL):

DDL commands are used to define and modify the structure of database objects such as tables, indexes, and schemas.

**Common DDL Commands:**

- **CREATE**: Creates new database objects (e.g., tables, databases).

  ```sql
  CREATE TABLE Employees (ID INT, Name VARCHAR(255));
  ```

- **ALTER**: Modifies existing database objects.

  ```sql
  ALTER TABLE Employees ADD COLUMN Address VARCHAR(255);
  ```

- **DROP**: Deletes existing database objects.

  ```sql
  DROP TABLE Employees;
  ```

- **TRUNCATE**: Removes all records from a table but retains the table structure.
  ```sql
  TRUNCATE TABLE Employees;
  ```

#### Data Manipulation Language (DML):

DML commands are used for manipulating data within existing tables.

**Common DML Commands:**

- **INSERT**: Adds new records to a table.

  ```sql
  INSERT INTO Employees (ID, Name) VALUES (1, 'John Doe');
  ```

- **UPDATE**: Modifies existing records.

  ```sql
  UPDATE Employees SET Name = 'Jane Doe' WHERE ID = 1;
  ```

- **DELETE**: Removes records from a table.
  ```sql
  DELETE FROM Employees WHERE ID = 1;
  ```

#### Data Query Language (DQL):

DQL is primarily concerned with querying data from the database.

**Common DQL Command:**

- **SELECT**: Retrieves data from one or more tables.
  ```sql
  SELECT Name FROM Employees WHERE ID = 1;
  ```

#### Data Control Language (DCL):

DCL commands manage user permissions and access controls within the database.

**Common DCL Commands:**

- **GRANT**: Assigns privileges to users.

  ```sql
  GRANT SELECT ON Employees TO UserName;
  ```

- **REVOKE**: Removes previously granted privileges.
  ```sql
  REVOKE SELECT ON Employees FROM UserName;
  ```

#### Transaction Control Language (TCL):

TCL commands manage transactions within the database.

**Common TCL Commands:**

- **COMMIT**: Saves all changes made in the current transaction.

  ```sql
  COMMIT;
  ```

- **ROLLBACK**: Undoes changes made in the current transaction if an error occurs.
  ```sql
  ROLLBACK;
  ```

### Commands in Detail

#### SELECT

The SQL SELECT statement is a powerful tool used to retrieve data from databases.

- **Basic SELECT Statement:**
  Retrieves specific columns from a table.

  ```sql
  SELECT column1, column2 FROM table_name;
  ```

- **SELECT All Columns:**
  Uses an asterisk (\*) to select all columns from a table.

  ```sql
  SELECT * FROM table_name;
  ```

- **SELECT with WHERE Clause:**
  Filters records based on specified conditions.

  ```sql
  SELECT column1, column2 FROM table_name WHERE condition;
  ```

- **SELECT with ORDER BY Clause:**
  Sorts the result set by one or more columns.

  ```sql
  SELECT column1, column2 FROM table_name ORDER BY column1 ASC|DESC;
  ```

- **SELECT with GROUP BY Clause:**
  Groups rows that have the same values in specified columns into summary rows.

  ```sql
  SELECT column1, COUNT(*) FROM table_name GROUP BY column1;
  ```

- **SELECT with HAVING Clause:**
  Filters groups created by GROUP BY based on specified conditions.

  ```sql
  SELECT column1, COUNT(*) FROM table_name GROUP BY column1 HAVING COUNT(*) > value;
  ```

- **SELECT with JOINs:**
  Combines rows from two or more tables based on related columns.

  - **INNER JOIN**: Returns records with matching values in both tables.

    ```sql
    SELECT a.column1, b.column2 FROM table_a a
    INNER JOIN table_b b ON a.common_column = b.common_column;
    ```

  - **LEFT JOIN**: Returns all records from the left table and matched records from the right table.
    ```sql
    SELECT a.column1, b.column2 FROM table_a a
    LEFT JOIN table_b b ON a.common_column = b.common_column;
    ```

- **SELECT with DISTINCT:**
  Returns unique values from a specified column.

  ```sql
  SELECT DISTINCT column1 FROM table_name;
  ```

- **SELECT with Subqueries:**
  A nested query that retrieves data based on results from another query.

  ```sql
  SELECT column1 FROM table_name
  WHERE column2 IN (SELECT column2 FROM another_table WHERE condition);
  ```

- **SELECT INTO Statement:**
  Copies data from one table into another new table.
  ```sql
  SELECT * INTO new_table FROM existing_table WHERE condition;
  ```

#### INSERT

The SQL INSERT statement is essential for adding new records to a database table. There are several types of INSERT statements, each serving different purposes. Here's a comprehensive overview:

- **Single Row Insert:**
  This is the most basic form of the INSERT statement, used to add a single row to a table.

  ```sql
  INSERT INTO table_name (column1, column2, column3)
  VALUES (value1, value2, value3);
  ```

- **Insert Without Specifying Columns:**
  You can insert data without specifying the column names if you provide values for all columns in the correct order.

  ```sql
  INSERT INTO table_name VALUES (value1, value2, value3);
  ```

- **Multi-Row Insert:**
  This allows you to insert multiple rows in a single INSERT statement, which can improve performance.
  ```sql
  INSERT INTO table_name (column1, column2) VALUES
  (value1a, value2a),
  (value1b, value2b),
  (value1c, value2c);
  ```

#### CREATE

The CREATE statement in SQL is primarily used to create database objects such as tables, indexes, views, and types.

- **CREATE TABLE:**
  This statement is used to create a new table in the database. You define the table name, columns, and their data types.

  ```sql
  CREATE TABLE table_name (
      column1 datatype,
      column2 datatype,
      ...
  );
  ```

  **Example:**

  ```sql
  CREATE TABLE Employees (
      EmployeeID INT PRIMARY KEY,
      FirstName VARCHAR(50),
      LastName VARCHAR(50),
      HireDate DATE
  );
  ```

#### ALTER

The ALTER statement in SQL is used to modify the structure of an existing database object, primarily tables.

- **ALTER TABLE - ADD Column:**
  This command adds one or more new columns to an existing table.

  ```sql
  ALTER TABLE table_name ADD column_name datatype;
  ```

  **Example:**

  ```sql
  ALTER TABLE Employees ADD Age INT;
  ```

- **ALTER TABLE - DROP Column:**
  This command removes one or more columns from a table.

  ```sql
  ALTER TABLE table_name DROP COLUMN column_name;
  ```

  **Example:**

  ```sql
  ALTER TABLE Employees DROP COLUMN Age;
  ```

- **ALTER TABLE - MODIFY/ALTER Column:**
  This command changes the data type or constraints of an existing column.

  **Syntax for MySQL/Oracle:**

  ```sql
  ALTER TABLE table_name MODIFY column_name new_datatype;
  ```

  **Syntax for SQL Server:**

  ```sql
  ALTER TABLE table_name ALTER COLUMN column_name new_datatype;
  ```

  **Example:**

  ```sql
  ALTER TABLE Employees MODIFY LastName VARCHAR(100);
  ```

- **ALTER TABLE - RENAME Column:**
  This command renames an existing column in a table.

  **Syntax (varies by database):**

  ```sql
  ALTER TABLE table_name RENAME COLUMN old_name TO new_name;
  ```

  **Example (for PostgreSQL):**

  ```sql
  ALTER TABLE Employees RENAME COLUMN LastName TO Surname;
  ```

- **ALTER TABLE - RENAME Table:**
  This command changes the name of an existing table.

  ```sql
  ALTER TABLE old_table_name RENAME TO new_table_name;
  ```

  **Example:**

  ```sql
  ALTER TABLE Employees RENAME TO StaffMembers;
  ```

- **ALTER TABLE - ADD CONSTRAINT:**
  This command adds a new constraint (e.g., primary key, foreign key) to an existing table.

  ```sql
  ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_type (column_name);
  ```

  **Example:**

  ```sql
  ALTER TABLE Employees ADD CONSTRAINT pk_employee PRIMARY KEY (EmployeeID);
  ```

- **ALTER TABLE - DROP CONSTRAINT:**
  This command removes an existing constraint from a table.

  ```sql
  ALTER TABLE table_name DROP CONSTRAINT constraint_name;
  ```

  **Example:**

  ```sql
  ALTER TABLE Employees DROP CONSTRAINT pk_employee;
  ```

These ALTER statements provide flexibility in modifying database structures as requirements evolve, ensuring that databases can adapt to changing needs without requiring complete redesigns.

#### DROP vs DELETE

In SQL, both the DELETE and DROP commands are used to remove data, but they serve distinct purposes and operate at different levels.

**DELETE Command**

- **Purpose**: The DELETE command is used to remove specific rows from a table based on a condition. It allows for selective deletion of data while preserving the table structure.
- **Type**: It is classified as a Data Manipulation Language (DML) command.
- **Syntax**:
  ```sql
  DELETE FROM table_name WHERE condition;
  ```
**DROP Command**

- **Purpose**: The DROP command is used to completely remove an entire database object, such as a table or database, along with all its data and structure. Once executed, the object cannot be recovered unless a backup exists.
- **Type**: It is classified as a Data Definition Language (DDL) command.
- **Syntax**:
  ```sql
  DROP TABLE table_name;
  ```

#### UPDATE

The SQL UPDATE statement is used to modify existing records in a table. It allows for updating single or multiple columns for one or more rows based on specified conditions.

- **Basic UPDATE Statement:**
  This is used to update specific columns in a single row based on a condition.

  ```sql
  UPDATE table_name SET column1 = value1 WHERE condition;
  ```

  **Example:**

  ```sql
  UPDATE Customers SET ContactName = 'Alfred Schmidt' WHERE CustomerID = 1;
  ```

- **UPDATE Multiple Columns:**
  You can update multiple columns in a single UPDATE statement.

  ```sql
  UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;
  ```

  **Example:**

  ```sql
  UPDATE Customers SET ContactName = 'Alfred Schmidt', City = 'Frankfurt' WHERE CustomerID = 1;
  ```

- **UPDATE Multiple Rows:**
  This updates all rows that meet a specified condition.

  ```sql
  UPDATE table_name SET column_name = new_value WHERE condition;
  ```

  **Example:**

  ```sql
  UPDATE Customers SET ContactName = 'Juan' WHERE Country = 'Mexico';
  ```

- **UPDATE Without WHERE Clause:**
  If the WHERE clause is omitted, all records in the table will be updated. This should be used with caution.

  **Example:**

  ```sql
  UPDATE Customers SET ContactName = 'Juan';
  ```

- **UPDATE Using Subquery:**
  You can use a subquery to determine the values to be updated based on data from another table.

  ```sql
  UPDATE table_name SET column_name = (SELECT value FROM another_table WHERE condition) WHERE condition;
  ```

  **Example:**

  ```sql
  UPDATE Customers SET City = (SELECT City FROM Locations WHERE Country = 'Germany') WHERE Country = 'Germany';
  ```

- **UPDATE with JOIN (in some SQL dialects):**
  This allows you to update records in one table based on matching records in another table.

  **Syntax (for SQL Server):**

  ```sql
  UPDATE a SET a.column1 = b.column2 FROM table_a a
  JOIN table_b b ON a.common_column = b.common_column WHERE condition;
  ```

  **Example:**

  ```sql
  UPDATE c SET c.ContactName = b.NewContactName
  FROM Customers c
  JOIN NewContacts b ON c.CustomerID = b.CustomerID;
  ```

- **Conditional Updates with CASE Statement:**
  You can use a CASE statement within an UPDATE to set different values based on conditions.

  ```sql
  UPDATE table_name
  SET column_name = CASE
      WHEN condition1 THEN value1
      WHEN condition2 THEN value2
      ELSE default_value
      END
  WHERE condition;
  ```

  **Example:**

  ```sql
  UPDATE Customers
  SET ContactName = CASE
      WHEN Country = 'Germany' THEN 'Hans Müller'
      WHEN Country = 'Mexico' THEN 'Juan Pérez'
      ELSE 'Unknown'
      END;
  ```

These various forms of the UPDATE statement provide flexibility in modifying data within SQL databases, allowing users to perform targeted changes efficiently while maintaining data integrity.
