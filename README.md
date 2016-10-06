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










##Chapter 7.  Table Partitioning
###Table partitioning


####Partition implementation















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
