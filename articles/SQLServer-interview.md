# SQL Server Interview

## Normalizzazione

Source [Descrizione delle nozioni di base di normalizzazione del database](https://docs.microsoft.com/it-it/office/troubleshoot/access/database-normalization-description)

**Descrizione della normalizzazione**

Esistono alcune regole per la normalizzazione dei database. Ogni regola è denominata "forma normale". Se viene osservata la prima regola, il database viene definito in "First Normal Form". Se vengono rispettate le prime tre regole, il database viene considerato in "terza forma normale". Anche se sono possibili altri livelli di normalizzazione, la terza forma normale è considerata il livello più alto necessario per la maggior parte delle applicazioni.

**Primo modulo normale:**
* Eliminare i gruppi ripetuti nelle singole tabelle.
* Creare una tabella separata per ogni set di dati correlati.
* Identificare ogni set di dati correlati con una chiave primaria.

Non utilizzare più campi in un'unica tabella per archiviare dati simili. Ad esempio, per tracciare un elemento di inventario che può provenire da due origini possibili, un record di inventario può contenere campi per il codice fornitore 1 e il codice fornitore 2.

Cosa succede quando si aggiunge un terzo fornitore? L'aggiunta di un campo non è la risposta. richiede modifiche al programma e alla tabella e non può contenere agevolmente un numero dinamico di fornitori. Al contrario, inserire tutte le informazioni sui fornitori in una tabella separata denominata vendors, quindi collegare l'inventario ai fornitori con una chiave del numero di elemento o ai fornitori per l'inventario con una chiave del codice fornitore.

**Secondo modulo normale:**
* Creare tabelle separate per insiemi di valori che si applicano a più record.
* Correlare queste tabelle con una chiave esterna.

I record non devono dipendere da una chiave primaria di una tabella, se necessario, una chiave composta. Si consideri, ad esempio, l'indirizzo di un cliente in un sistema contabile. L'indirizzo è necessario per la tabella Customers, ma anche per le tabelle ordini, spedizioni, fatture, account attivi e raccolte. Invece di archiviare l'indirizzo del cliente come voce separata in ognuna di queste tabelle, archiviarlo in un'unica posizione, nella tabella Customers o in una tabella di indirizzi distinta.

**Terza maschera normale:**
* Eliminare i campi che non dipendono dalla chiave.

I valori di un record che non fanno parte della chiave di quel record non appartengono alla tabella. In generale, ogni volta che il contenuto di un gruppo di campi può essere applicato a più di un singolo record nella tabella, prendere in considerazione l'inserimento di tali campi in una tabella separata.

## SELECT

```sql
SELECT column1, column2, ...
FROM table_name;
```

```sql
SELECT * FROM table_name;
```

Is possible to limit selected rows max quantity

```sql
SELECT TOP number|percent column_name(s)
FROM table_name
WHERE condition;
```

The SELECT INTO statement copies data from one table into a new table.

```sql
SELECT * | column1, column2, column3, ...
INTO newtable [IN externaldb]
FROM oldtable
WHERE condition;
```

## DISTINCT

```sql
SELECT DISTINCT column1, column2, ...
FROM table_name;
```

## WHERE

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

## AND OR NOT

```sql
SELECT column1, column2, ...
FROM table_name
WHERE NOT condition1 AND condition2 OR condition3 ...;
```

## ORDER BY

```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY column1, column2, ... ASC|DESC;
```

## INSERT INTO

```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

The `INSERT INTO SELECT` statement copies data from one table and inserts it into another table.

`INSERT INTO SELECT` requires that data types in source and target tables match
The existing records in the target table are unaffected

```sql
INSERT INTO table2
SELECT * FROM table1
WHERE condition;
```


## IS NULL, IS NOT NULL

```sql
SELECT column_names
FROM table_name
WHERE column_name IS NULL|IS NOT NULL;
```

## UPDATE

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

## DELETE

```sql
DELETE FROM table_name WHERE condition;
```

## MIN MAX

The MIN() function returns the smallest value of the selected column.

The MAX() function returns the largest value of the selected column.

```sql
SELECT MIN|MAX(column_name)
FROM table_name
WHERE condition;
```

## COUNT, AVG, SUM

The COUNT() function returns the number of rows that matches a specified criterion.

The AVG() function returns the average value of a numeric column.

The SUM() function returns the total sum of a numeric column.

```sql
SELECT COUNT|AVG|SUM(column_name)
FROM table_name
WHERE condition;
```

## LIKE

The LIKE operator is used in a WHERE clause to search for a specified pattern in a column.

There are two wildcards often used in conjunction with the LIKE operator:
* % - The percent sign represents zero, one, or multiple characters
* _ - The underscore represents a single character

```sql
SELECT column1, column2, ...
FROM table_name
WHERE columnN LIKE pattern;
```

## SQL Wildcard Characters

Wildcard Characters in SQL Server:

|Symbol|Description|Example|
|:-|:-|:-|
|%|Represents zero or more characters|bl% finds bl, black, blue, and blob|
|_|Represents a single character|h_t finds hot, hat, and hit|
|[]|Represents any single character within the brackets|h[oa]t finds hot and hat, but not hit|
|^|Represents any character not in the brackets|h[^oa]t finds hit, but not hot and hat|
|-|Represents a range of characters|c[a-b]t finds cat and cbt|

> All the wildcards can also be used in combinations!

## IN

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1, value2, ...)|(SELECT STATEMENT);
```

## BETWEEN

The BETWEEN operator selects values within a given range. The values can be numbers, text, or dates.

The BETWEEN operator is inclusive: begin and end values are included. 

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
```

## Aliases

```sql
SELECT column_name AS alias_name
FROM table_name AS alias_name;
```

## JOINS

```sql
SELECT Orders.OrderID, Customers.CustomerName, Orders.OrderDate
FROM Orders
INNER JOIN Customers ON Orders.CustomerID=Customers.CustomerID;
```

Here are the different types of the JOINs in SQL:
* (INNER) JOIN: Returns records that have matching values in both tables
* LEFT (OUTER) JOIN: Returns all records from the left table, and the matched records from the right table
* RIGHT (OUTER) JOIN: Returns all records from the right table, and the matched records from the left table
* FULL (OUTER) JOIN: Returns all records when there is a match in either left or right table

![inner_join](images/img_innerjoin.gif)

![left_join](images/img_leftjoin.gif)

![right_join](images/img_rightjoin.gif)

![full_outer_join](images/img_fulljoin.gif)

A self JOIN is a regular join, but the table is joined with itself.

```sql
SELECT column_name(s)
FROM table1 T1, table1 T2
WHERE condition;
```

## UNION

The UNION operator is used to combine the result-set of two or more SELECT statements.
* Each SELECT statement within UNION must have the same number of columns
* The columns must also have similar data types
* The columns in each SELECT statement must also be in the same order

```sql
SELECT column_name(s) FROM table1
UNION|UNION ALL
SELECT column_name(s) FROM table2;
```

## GROUP BY

The GROUP BY statement groups rows that have the same values into summary rows, like "find the number of customers in each country".

The GROUP BY statement is often used with aggregate functions (COUNT, MAX, MIN, SUM, AVG) to group the result-set by one or more columns.

```sql
SELECT column_name(s)
FROM table_name
WHERE condition
GROUP BY column_name(s)
ORDER BY column_name(s);
```

## HAVING

The HAVING clause was added to SQL because the WHERE keyword could not be used with aggregate functions.

```sql
SELECT column_name(s)
FROM table_name
WHERE condition
GROUP BY column_name(s)
HAVING condition
ORDER BY column_name(s);
```

## EXISTS

The EXISTS operator is used to test for the existence of any record in a subquery.

The EXISTS operator returns true if the subquery returns one or more records.

```sql
SELECT column_name(s)
FROM table_name
WHERE EXISTS
(SELECT column_name FROM table_name WHERE condition);
```

## ANY and ALL

The ANY and ALL operators are used with a WHERE or HAVING clause.

The ANY operator returns true if any of the subquery values meet the condition.

The ALL operator returns true if all of the subquery values meet the condition.

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name operator ANY|ALL
(SELECT column_name FROM table_name WHERE condition);
```

> The operator must be a standard comparison operator (=, <>, !=, >, >=, <, or <=)

## CASE

The `CASE` statement goes through conditions and returns a value when the first condition is met (like an `IF-THEN-ELSE` statement). So, once a condition is true, it will stop reading and return the result. If no conditions are true, it returns the value in the `ELSE` clause.

If there is no `ELSE` part and no conditions are true, it returns `NULL`.

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END;
```

## IFNULL(), ISNULL(), COALESCE(), and NVL()

The SQL Server ISNULL() function lets you return an alternative value when an expression is NULL:
```sql
SELECT ProductName, UnitPrice * (UnitsInStock + ISNULL(UnitsOnOrder, 0))
FROM Products;
```

## Stored Procedures

A stored procedure is a prepared SQL code that you can save, so the code can be reused over and over again.

So if you have an SQL query that you write over and over again, save it as a stored procedure, and then just call it to execute it.

You can also pass parameters to a stored procedure, so that the stored procedure can act based on the parameter value(s) that is passed.

```sql
CREATE PROCEDURE procedure_name
AS
sql_statement
GO;
```

```sql
EXEC procedure_name;
```

## Coments

```sql
--Single line

/*
    Multiline
*/

```

## Operators

Arithmetic Operators:
* +	Add	
* -	Subtract	
* *	Multiply	
* /	Divide	
* %	Modulo

Bitwise Operators:
* &	Bitwise AND
* |	Bitwise OR
* ^	Bitwise exclusive OR

Comparison Operators:
* =	Equal to	
* >	Greater than	
* <	Less than	
* >=	Greater than or equal to	
* <=	Less than or equal to	
* <>	Not equal to

Compound Operators:
* +=	Add equals
* -=	Subtract equals
* *=	Multiply equals
* /=	Divide equals
* %=	Modulo equals
* &=	Bitwise AND equals
* ^-=	Bitwise exclusive equals
* |*=	Bitwise OR equals

Logical Operators:
* ALL	TRUE if all of the subquery values meet the condition	
* AND	TRUE if all the conditions separated by AND is TRUE	
* ANY	TRUE if any of the subquery values meet the condition	
* BETWEEN	TRUE if the operand is within the range of comparisons	
* EXISTS	TRUE if the subquery returns one or more records	
* IN	TRUE if the operand is equal to one of a list of expressions	
* LIKE	TRUE if the operand matches a pattern	
* NOT	Displays a record if the condition(s) is NOT TRUE	
* OR	TRUE if any of the conditions separated by OR is TRUE	
* SOME	TRUE if any of the subquery values meet the condition

## CREATE DATABASE

```sql
CREATE DATABASE databasename;
```

## DROP DATABASE

```sql
DROP DATABASE databasename;
```

## BACKUP DATABASE

```sql
BACKUP DATABASE databasename
TO DISK = 'filepath';
```

## CREATE TABLE

```sql
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    column3 datatype,
   ....
);
```

## DROP TABLE

```sql
DROP TABLE table_name;
```

## ALTER TABLE

```sql
ALTER TABLE table_name
ADD column_name datatype;

ALTER TABLE table_name
DROP COLUMN column_name;
```

## Constraints

SQL constraints are used to specify rules for data in a table.

```sql
CREATE TABLE table_name (
    column1 datatype constraint,
    column2 datatype constraint,
    column3 datatype constraint,
    ....
);
```

The following constraints are commonly used in SQL:
* NOT NULL - Ensures that a column cannot have a NULL value
* UNIQUE - Ensures that all values in a column are different
* PRIMARY KEY - A combination of a NOT NULL and UNIQUE. Uniquely identifies each row in a table
* FOREIGN KEY - Uniquely identifies a row/record in another table
* CHECK - Ensures that all values in a column satisfies a specific condition
```sql
CREATE TABLE Persons (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int CHECK (Age>=18)
);
```
* DEFAULT - Sets a default value for a column when no value is specified
```sql
CREATE TABLE Persons (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int,
    City varchar(255) DEFAULT 'Sandnes'
);

-- The DEFAULT constraint can also be used to insert system values, by using functions like GETDATE():

CREATE TABLE Orders (
    ID int NOT NULL,
    OrderNumber int NOT NULL,
    OrderDate date DEFAULT GETDATE()
);

-- DEFAULT on ALTER TABLE

ALTER TABLE Persons
ADD CONSTRAINT df_City
DEFAULT 'Sandnes' FOR City;

-- DROP a DEFAULT Constraint

ALTER TABLE Persons
ALTER COLUMN City DROP DEFAULT;
```
* INDEX - Used to create and retrieve data from the database very quickly

## CREATE INDEX

The `CREATE INDEX` statement is used to create indexes in tables.

Indexes are used to retrieve data from the database more quickly than otherwise. The users cannot see the indexes, they are just used to speed up searches/queries.

> Updating a table with indexes takes more time than updating a table without (because the indexes also need an update). So, only create indexes on columns that will be frequently searched against.

```sql
CREATE INDEX index_name
ON table_name (column1, column2, ...);

CREATE UNIQUE INDEX index_name
ON table_name (column1, column2, ...);
```

> The syntax for creating indexes varies among different databases. Therefore: Check the syntax for creating indexes in your database.

## DROP INDEX

```sql
DROP INDEX table_name.index_name;
```

## AUTO INCREMENT

Auto-increment allows a unique number to be generated automatically when a new record is inserted into a table.

Often this is the primary key field that we would like to be created automatically every time a new record is inserted.

```sql
CREATE TABLE Persons (
    Personid int IDENTITY(1,1) PRIMARY KEY,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int
);
```

The MS SQL Server uses the IDENTITY keyword to perform an auto-increment feature.

In the example above, the starting value for IDENTITY is 1, and it will increment by 1 for each new record.

>  To specify that the "Personid" column should start at value 10 and increment by 5, change it to IDENTITY(10,5).

To insert a new record into the "Persons" table, we will NOT have to specify a value for the "Personid" column (a unique value will be added automatically)

## Dates

SQL Server comes with the following data types for storing a date or a date/time value in the database:

DATE - format YYYY-MM-DD
DATETIME - format: YYYY-MM-DD HH:MI:SS
SMALLDATETIME - format: YYYY-MM-DD HH:MI:SS
TIMESTAMP - format: a unique number

## CREATE VIEW

In SQL, a view is a virtual table based on the result-set of an SQL statement.

A view contains rows and columns, just like a real table. The fields in a view are fields from one or more real tables in the database.

You can add SQL functions, WHERE, and JOIN statements to a view and present the data as if the data were coming from one single table.

```sql
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

> A view always shows up-to-date data! The database engine recreates the data, using the view's SQL statement, every time a user queries a view.

## Injection

SQL injection is a code injection technique that might destroy your database.
SQL injection is one of the most common web hacking techniques.
SQL injection is the placement of malicious code in SQL statements, via web page input.

[Security Considerations (Entity Framework)](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/ef/security-considerations?redirectedfrom=MSDN)

LINQ to Entities injection attacks:
Although query composition is possible in LINQ to Entities, it is performed through the object model API. Unlike Entity SQL queries, LINQ to Entities queries are not composed by using string manipulation or concatenation, and they are not susceptible to traditional SQL injection attacks.


