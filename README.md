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




