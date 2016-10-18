# pkpsqldevesst
##Chapter 1.  Advanced SQL
###Creating views

create 2 tables
```
CREATE TABLE suppliers (supplier_id number primary key,Supplier_name varchar(30),Phone_number number);
CREATE TABLE orders(order_number number primary key,Supplier_id  number references suppliers(supplier_id),Quanity number,Is_active varchar(10),Price number);
```
create view
```
CREATE VIEW active_supplier_orders AS SELECT suppliers.supplier_id, suppliers.supplier_name  orders.quantity, orders.price 
FROM suppliers INNER JOIN orders ON suppliers.supplier_id = orders.supplier_id WHERE suppliers.supplier_name = '' And orders.active='TRUE';
```
select view
```
SELECT * FROM active_supplier_orders;
```
####Deleting and replacing views
```
DROP VIEW IF EXISTS view_name;
CREATE OR REPLACE VIEW view_name AS ....
```



###Materialized views
A table that actually contains rows but behaves like a view.  
A materialized view cannot subsequently be directly updated, 
and the query used to create the materialized view is stored in exactly the same way as the view's query is stored.  
####Why materialized views?
If you have an environment where you run the same type of SELECT query multiple times against the same set of tables,  
then you can create a materialized view for SELECT so that, on every run, this view does not go to the actual tables to fetch the data, which will obviously reduce the load on them 
as you might be running a Data Manipulation Language (DML) against your actual tables at the same time. 

#####Read-only, updatable, and writeable materialized views
A materialized view can be read-only, updatable, or writeable.  
Users cannot perform DML statements on read-only materialized views, but they can perform them on updatable and writeable materialized views.  
#####Read-only materialized views
```
CREATE MATERIALIZED VIEW  view_name AS SELECT  columns FROM table
```
#####Updatable materialized views
```
CREATE MATERIALIZED VIEW  view_name  FOR UPDATE AS SELECT columns FROM table;
```


###Creating cursors
A cursor in PostgreSQL is a read-only pointer to a fully executed SELECT statement's result set.  
An application can more efficiently manage which rows to retrieve from a result set at different times without re-executing the query with different LIMIT and OFFSET clauses.  
The four SQL commands involved with PostgreSQL cursors are DECLARE, FETCH, MOVE, and CLOSE.  

declare
```
BEGIN;
DECLARE order_cur CURSOR FOR SELECT * FROM orders;
```
####Using cursors
Use the FETCH command to retrieve rows from the open cursor  
The MOVE command moves the current location of the cursor within the result set  
The CLOSE command closes the cursor, freeing up any associated memory  

cursor is the name of the cursor from where we can retrieve row data.  
A cursor always points to a current position in the executed statement's result set and rows can be retrieved either ahead of the current location or behind it.   
The FORWARD and BACKWARD keywords may be used to specify the direction, though the default is forward.  
The NEXT keyword (the default) returns the next single row from the current cursor position.  
The PRIOR keyword causes the single row preceding the current cursor position to be returned.  
Ex: fetch first 4 rows
```
FETCH 4 FROM order_cur;
```
####Closing a cursor
```
CLOSE order_cur;
```



###Using the GROUP BY clause
The GROUP BY clause enables you to establish data groups based on columns.  
All columns that are used besides the aggregate functions must be included in the GROUP BY clause. The GROUP BY clause does not support the use of column aliases; you must use the actual column names. The GROUP BY columns may or may not appear in the SELECT list. The GROUP BY clause can only be used with aggregate functions such as  SUM , AVG , COUNT , MAX , and MIN .  

Ex
```
SELECT product, SUM(sale) AS "Total sales" FROM order_details GROUP BY product;
```
In the select statement, we have sales where we applied the SUM function and the other field product is not part of SUM, we must use in the GROUP BY clause.



####Using the HAVING clause
The WHERE clause cannot be used to return the desired groups. The WHERE clause is only used to restrict individual rows. When the GROUP BY clause is not used, the HAVING clause works like the WHERE clause.  
Ex:
```
SELECT product, SUM(sale) AS "Total sales" FROM order_details GROUP BY product Having sum(sales)>10000;
```



####Using the UPDATE operation clauses
```
update tbname set k=v where..
```

####Using subqueries
The query inside the brackets is called the inner query. The query that contains the subquery is called the outer query.  
PostgreSQL executes the query that contains a subquery in the following sequence:
- First, it executes the subquery
- Second, it gets the results and passes it to the outer query
- Third, it executes the outer query  

Ex
```
SELECT employee_id,first_name,last_name,salary FROM employee WHERE salary > 25000;
SELECT avg(salary) from employee;
```
same as
```
SELECT employee_id,first_name,last_name,salary FROM employee WHERE salary > (Select avg(salary) from employee);
```





###Subqueries that return multiple rows
[techonthenet subquery explained](https://www.techonthenet.com/postgresql/subqueries.php)
Subqueries that return multiple rows can be used with the ALL, IN, ANY, or SOME operators. We can also negate the condition like NOT IN.   

Ex:
```
SELECT last_name, salary, department_id FROM employee outer
WHERE salary > (SELECT AVG(salary) FROM employee WHERE department_id = outer.department_id);
```
in this query, outer is alias of employee table

####Existence subqueries
The PostgreSQL EXISTS condition is used in combination with a subquery, and is considered to be met if the subquery returns at least one row. It can be used in a SELECT, INSERT, UPDATE, or DELETE statement. If a subquery returns any rows at all, the EXISTS subquery is true, and the NOT EXISTS subquery is false.





###Using the Union join
[doc](https://www.postgresql.org/docs/devel/static/queries-union.html)  
By default, the UNION join behaves like DISTINCT, that is, eliminates the duplicate rows; however, using the ALL keyword with the UNION join returns all rows, including the duplicates. The queries are all executed independently, but their output is merged. To sort the records in a combined result set, you can use ORDER BY.




###Using the Self join
```
SELECT a.emp_id AS "Emp_ID", a.emp_name AS "Employee Name",
b.emp_id AS "Supervisor ID",b.emp_name AS "Supervisor Name"
FROM employee a, employee b  -- using different aliases
WHERE a.supervisor_id = b.emp_id;
```



###Using the Outer join
Three types of Outer joins:
- The PostgreSQL LEFT OUTER JOIN (or sometimes called LEFT JOIN)
- The PostgreSQL RIGHT OUTER JOIN (or sometimes called RIGHT JOIN)
- The PostgreSQL FULL OUTER JOIN (or sometimes called FULL JOIN)  


















##Chapter 2. Data Manipulation
###Conversion between datatypes
syntax
```
CAST ( expression AS type ) 
```
Or
```
expression :: type 
```
pg_cast stores datatype conversion paths that are built-in and the one that is being defined with the help of CREATE CAST:
```
SELECT castsource::regtype, casttarget::regtype FROM pg_cast limit 2; 
```


###Introduction to arrays
PostgreSQL supports columns to defined as an  array. This array is of a variable length and can be defined as a one-dimensional or multidimensional array.


####Array constructors
int array:
```
SELECT ARRAY[1,2,3] AS sample_array;   --- same as
SELECT ARRAY[1,2,3.2] :: integer[] AS sample_array; 
```

text array with insertion
```
CREATE TABLE supplier (name text, product  text[])
INSERT INTO supplier VALUES ('Supplier1','{ "table","chair" }' )
INSERT INTO supplier VALUES (' Supplier2 ', ARRAY['pen','page '] )
```

using array function convert records to array
```
SELECT array(SELECT distinct product_name FROM supplier WHERE name='Supplier1') AS product_name; 
```

more functions
#####String_to_array() , array_to_string()
```
SELECT string_to_array ('sample string to be converted',' ');
SELECT array_to_string (ARRAY[1, 2, 3], ',');
```


#####Array_dims( )
tbc

#####ARRAY_AGG()
The ARRAY_AGG function is used to concatenate values including null into an array.  
Ex
```
SELECT name FROM supplier;
```
name  
----|--  
 
Supplier1  
Supplier2  
```
SELECT ARRAY_AGG(NAME) FROM supplier;
```
Output:  
array_agg  
---------------------------------------------   
 {Supplier2,Supplier1,NULL}  
 
 
 We can use the array_to_string function as well in order to ignore NULL:
```
SELECT array_to_string(array_agg(name), ',') FROM supplier; 
//In version 9.x, we can use COALESCE in order to handle a null or an empty array:
SELECT array_to_string(array_agg(coalesce(name, '')), ',') FROM supplier; 
```

#####ARRAY_UPPER() ARRAY_UPPER()
Returns the upper bound of the array dimension. 

#####Array_length()
```
SELECT array_length(product,1) FROM supplier WHERE name='Supplier1'; 
```


####Array slicing and splicing



####UNNESTing arrays to rows






####Introduction to JSON






















##Chapter 3. Triggers
###Introduction to triggers
operations:
- A Data Manipulation Language (DML) statement—DELETE, INSERT, or UPDATE
- A Data Definition Language (DDL) statement—CREATE, ALTER, or DROP
- A database operation—SERVERERROR, LOGON, LOGOFF, STARTUP, or SHUTDOWN  


There are two type of triggers
- Row-level trigger: An event is triggered for each row updated, inserted, or deleted
- Statement-level trigger: When a SQL statement emits an event, any triggers that are registered to listen for that event will be fired


####Adding triggers to PostgreSQL
Create the employee table and the emp_salary_history table:
```
CREATE TABLE employee  (emp_id int4, employee_name varchar(25), department_id int4, salary numeric(7,2));  
CREATE TABLE emp_salary_history(emp_id int, employee_name varchar(25), salary numeric(7,2) ,changed_on timestamp(6)); 
```

```
This is the hierarchy followed when a trigger is fired. The BEFORE statement's trigger fires first. Next, the BEFORE row-level trigger fires, once for each row affected. Then the AFTER row-level trigger fires, once for each affected row. These events will alternate between the BEFORE and AFTER row-level triggers. Finally, the AFTER statement-level trigger fires.
```
(tbc)


##Chapter 4.  Understanding Database Design Concepts
###Anomalies in DBMS
These are the insertion, update, and deletion anomalies. 







##Chapter 5. Transactions and Locking
###Defining transactions
####ACID rules


####Explicit locking
Locks for rows or tables are only allowed inside the transaction. Once the transaction gets completed with a commit or rollback, all the locks acquired during the transaction will be released automatically.


#####Locking tables


```
LOCK TABLE table-name 
```
same as
```
LOCK TABLE table-name ACCESS EXCLUSIVE MODE 
```













##Chapter 6. Indexes and Constraints
###Introduction to indexes and constraints
####Primary key indexes
add pk
```
CREATE TABLE emp(
 empid integer,
 empname varchar,
 sal numeric);
ALTER TABLE emp ADD PRIMARY KEY(empid);
```

check indexes
```
select * from pg_indexes where tablename='emp';
```

####Unique indexes
A unique index is also used to maintain uniqueness; however, it allows NULL values. (multiple null do not consider duplicate)
```
CREATE TABLE emp(
 empid integer UNIQUE,
 empname varchar,
 sal numeric);
```
or
```
CREATE TABLE emp(
 empid integer,
 empname varchar,
 sal numeric,
 UNIQUE(empid));
```

####B-tree indexes
The B-tree index can be used by an optimizer whenever the indexed column is used with a comparison operator, such as <, <=, =, >=, >, and LIKE or the ~ operator; however, LIKE or ~ will only be used if the pattern is a constant and anchored to the beginning of the string, for example, my_col LIKE 'mystring%' or my_column ~ '^mystring', but not my_column LIKE '%mystring'.  



####Full text indexes













##Chapter 7.  Table Partitioning
###Table partitioning


####Partition implementation
Partitioning a table is nothing but creating a separate table called a child table for each partition. 

parent table
```
CREATE TABLE IF NOT EXISTS customers (id int, name varchar(126),countrycode varchar(3),contactnum text);
```
create child tables
```
CREATE TABLE customers_USA(CHECK (countrycode='USA'))  INHERITS(customers);
CREATE TABLE customers_UK(CHECK (countrycode='UK'))  INHERITS(customers);
```
show all child tables
```
\d+ customers
```
use inhrelid:regclass and inhparent::regclass
```
SELECT inhrelid::regclass, inhparent::regclass from  pg_inherits WHERE inhparent::regclass='customers'::regclass;
```

impl trigger function
```
Create or replace function customers_before_insert_trig_func() returns trigger as
$$
begin
if (new.countrycode='USA') then
insert into customers_USA values (new.*);
elsif (new.countrycode='UK') then
insert into customer_UK values (new.*);
else
raise exception 'error';
end if
return null;
end;
$$ language plpgsql;
```
create trigger

```
create trigger customers_ins_trig before insert on customers for each row execute procedure customers_before_insert_trig_func();
```

insert test:
```
WITH stuff AS
(
  SELECT array['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H']  as names,   
  array['USA', 'UK'] AS cnames)
INSERT INTO customers
SELECT Generate_series(1, 1000) id,
names[(random()*100)::int%8+1] cname,
cnames[(random()*100)::int%6+1] countrycode,
ltrim(round(random()::numeric, 10)::text, '0.') contactnumber 
FROM stuff;
```
(insert 0 0)


We have 1000 records; however, our actual INSERT operation returned 0 inserted records. The reason behind this discrepancy is the TRIGGER function. Our trigger function diverts the incoming INSERT into its corresponding child table, and returns INSERT 0 0 as a result to the parent table.   


We can explain anapyze querytot test unpartitioned table and partitioned table respectively.
```
EXPLAIN ANALYZE SELECT COUNT(*), countrycode FROM  customers WHERE countrycode IN ('USA', 'UK') GROUP BY countrycode;
```
For partitioned table, postgres only scanned the required child tables rather than scanning them all.




####Partitioning types
#####List partition
```
CREATE TABLE customers_APAC (CHECK(countrycode IN ('USA','UK')))  INHERITS(customers);
```
#####Range partition
parent table
```
CREATE TABLE LOGS(id INT, recorded_date DATE,logmsg TEXT);
```
child table
```
CREATE TABLE LOGS_2015JAN(CHECK(recorded_date>='2015-01-01' AND recorded_date<'2015-02-01')) INHERITS(LOGS);
CREATE TABLE LOGS_2015FEB(CHECK(recorded_date>='2015-02-01' AND recorded_date<'2015-03-01')) INHERITS(LOGS);
```

In PostgreSQL, we only have list and range partitions.


####Adding a new partition
Dynamic trigger function code that automatically INSERT into child tables:
```
CREATE OR REPLACE FUNCTION  public.customers_before_insert_trig_func()
RETURNS trigger AS
$$
BEGIN
EXECUTE 'INSERT INTO customers_'||NEW.countrycode||' VALUES  ($1.*)' USING NEW;  -- Need to memorize
RETURN NULL;      
END
$$
LANGUAGE plpgsql;
```
Here we have a update of. [official doc](https://www.postgresql.org/docs/current/static/sql-createtrigger.html)
```
CREATE or REPLACE TRIGGER emp_history_trigger  
BEFORE UPDATE OF salary  
ON employee  
FOR EACH ROW  
EXECUTE PROCEDURE insert_into_salary_history();
```


####Purging an old partition
remove customers_jap from customers:
```
ALTER TABLE customers_jap NO INHERIT customers;
```
[pg_partman](https://github.com/keithf4/pg_partman)


####Alternate partitioning methods
#####Method 1
disable triggers:
```
ALTER TABLE customers DISABLE trigger  customers_ins_trig;
```
create rule. Insert into child table instead of insert into parent table
```
create rule customers_usa_rule as on 
insert into customers WHERE countrycode = 'USA'
DO INSTEAD INSERT INTO customers_usa VALUES(NEW.*);
```


#####Method 2
union child tables
```
CREATE TABLE customers_usa(id int, name varchar(126),  countrycode varchar(3), contactnum text);
CREATE TABLE customers_uk(id int, name varchar(126),  countrycode varchar(3), contactnum text);
```
union
```
CREATE VIEW v_customers AS SELECT * FROM customers_usaUNION ALL SELECT * FROM customers_uk; 
```
(tbc)


####Constraint exclusion
create simple table
```
CREATE TABLE testing(t CHAR(1) CHECK(t='A')); 
INSERT INTO testing VALUES('A');
```
disable constraint_exclusion
```
SET CONSTRAINT_EXCLUSION TO OFF;
```
test this query with CONSTRAINT_EXCLUSION TO OFF/ON
```
EXPLAIN ANALYZE SELECT * FROM testing WHERE t='Z';
```
Enabling ```CONSTRAINT_EXCLUSION``` parameter will increase the planning time, but the actual execution time will be reduced if the given predicate doesn't match the constraints.



#####PL/Proxy
PL/Proxy works on a power of 2 number of partitions only. That is, we need to have 2^(N) number of partitions to configure PL/Proxy.  


#####Foreign inheritance (9.5+)
```
CREATE DATABASE customers_usa;
CREATE DATABASE customers_uk;
```

(tbc)









##Chapter 9. PostgreSQL Extensions and Large Object Support
[schema in psql](http://www.estelnetcomputing.com/index.php?/archives/10-Database-vs.-Schema-Definition-in-Mysql-vs.-Postgres.html)
In mysql a database is a schema. The two words can be used interchangeably in most commands. For instance, you could say create database dbname, or create schema dbname and achieve the same result.
Postgres has a different concept of schema. It is a namespace that contains tables, functions, operators and data types. It is essentially a layer between the database and the tables. In postgres, a database may contain many schemas and those schemas may contain the same tables or different ones. The "public" schema is the default schema in a database. 
###Creating an extension
```
CREATE EXTENSION pg_stat_statements;
```
check
```
\df pg_stat_statement*
\dv pg_stat_statement*
```
if we do not provide any schema as an owner while creating an extension, it creates an extension in the schema public
```
DROP EXTENSION IF EXISTS pg_stat_statements;
CREATE EXTENSION pg_stat_statements WITH SCHEMA owner;
```
change back
```
ALTER EXTENSION pg_stat_statements SET SCHEMA public;
```

####Compiling extensions
```
SELECT * FROM pg_available_extensions limit 1;
```
We cannot create an extension that is not already compiled.
Ex, postgis, we need to download and compile first. Then config
```
./configure --with-pgconfig=/path/to/your/pg_config
```
create extension
```
SELECT * FROM pg_available_extensions WHERE name='postgis';
CREATE EXTENSION postgis;
```


####Database links in PostgreSQL
```
CREATE SCHEMA for_dblink;
CREATE EXTENSION dblink WITH SCHEMA for_dblink;
\df for_dblink.dblink*
```
(tbc)

###Using binary large objects
####Server-side functions
lo_import, lo_export,
```
CREATE TABLE image_load(name text, image oid);
INSERT INTO image_load VALUES('my_image', lo_import('/tmp/pic.png'));
```
You can specify an OID while importing:
```
INSERT INTO image_load VALUES('my_image',  lo_import('/tmp/pic.png',123456));
select lo_export(image_load.image, '/tmp/pic.png') from  image_load ;
```
