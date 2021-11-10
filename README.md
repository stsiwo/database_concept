# RDBMS (esp MySQL)

## Terms

### Natual Key

a unique column that has a business meaning such as SSN, email

### Surrogate Key

a unique column that does not has a business meaning such UUID, auto-increment

### Discriminator (Partial Key)

a column in a weak entity to be a partial key to be unique by combining with another columns. 

sounds like composit key?? what is the different??

### Weak Entity

an entity does not have a primary key attribute. 

### Temporary Table

a special type of table that allows you to store a temporary result set, which you can reuse several times in a single session.

this is very handy when it is impossible or expensive to query data that requires a single SELECT statement with the JOIN clauses. In this case, you can use a temporary table to store the immediate result and use another query to process it.

a temporary can keep the data in the table so you don't need to run the expensive query again. 

you can use the temporary table as a usual table withing the session. 

```
-- create a temporary table based on DDL
CREATE TEMPORARY TABLE credits(
    customerNumber INT PRIMARY KEY,
    creditLimit DEC(10,2)
);

-- create a temporary table based on queries
CREATE TEMPORARY TABLE top_customers
SELECT p.customerNumber, 
       c.customerName, 
       ROUND(SUM(p.amount),2) sales
FROM payments p
INNER JOIN customers c ON c.customerNumber = p.customerNumber
GROUP BY p.customerNumber
ORDER BY sales DESC
LIMIT 10;

-- you can use it as usual table.
SELECT 
    customerNumber, 
    customerName, 
    sales
FROM
    top_customers
ORDER BY sales;
```

reference: [here](https://www.mysqltutorial.org/mysql-temporary-table/)

### Views

Views are stored queries that when invoked produce a result set. A view acts as a virtual table.

```
-- create a view called 'accounts_v_members'
CREATE VIEW `accounts_v_members` AS SELECT `membership_number`,`full_names`,`gender` FROM `members`;

-- call the view
SELECT membership_number, full_names FROM accounts_v_members
```

- __every time__ you run the view, it gerenate the result on the view as a virtual table. 
- you can use views to make your query simple. if you have a complex select with lots of joins, you can implement it in a view and simply call the view without need to consider all these joins. You can then reuse this view.
- A view can provide a simple, public interface to a complex SELECT statement, so applications can just 'SELECT column-list FROM my-easy-view'.

reference: [here](https://www.guru99.com/views.html)

## full text search

Full-text search is a technique that enables you to search for records that might not perfectly match the search criteria.

## Antipatterns

### Jaywalking

### Adjacency List

one-to-one relationship with the same table. this design is troublesome when you want to store the tree structure data (e.g., a comment has multiple comments and each child comment has mutliple comment too)

#### whey it bothers 

1. how to retrieve a particular level of nodes? you need to use mutliple join keyword based on the how much level of node you want to retrieve. (e.g., if you want to get a note which reside in 3 level, you need to use 3 join keywords.

#### solutions (how to store tree structure data in database the efficient way)

1. __Path Enumeration__ 

create a column and store the path. it represents hiarachy. for example, one row has '/a/b/c' in the column and another has 'a/b'. this represents that the 2nd row is the parent of the first row.

drawbacks:

  1. __no validation for the path column in database__. for example, if the parent path does not exists? you always have to validate the input in application side.

2. Nested Sets

create two columns called 'nsleft' and 'nsright' to locate the position of a node. 

there is a rule to create the Nested Sets. __the nsleft number is less than the numbers of all the node’s children, whereas the nsright number is greater than the numbers of all the node’s children. These numbers have no relation to the comment_id values__.

to find a particular node and its desendants, you need to use 'between' in 'where' condition and put 'nsleft' and 'nsright' value of the target node. 

updating the tree is more complex than other solutions. see textbook at 47 page. you need to recalculate 'nsleft' and 'nsright' value of all nodes. (sound like not practical)


3. Closure Tables

create a additional table besides comments table. then, store the relationship info (e.g., anscestor and descendant).

relatively easier than others esp Nested Sets for CRUD operation. 

### Primary Key When Needed

Are all tables required to have a primary key?

it is not mandatory that you have to use the primary key in every table. if you think you don't need the primary key, you don't need it. (e.g., can use compound keys on join table for many-to-many relationship)

### Entity-Attribute-Value (EVA) Design

a generalization of row modeling. the idea is that you try to records all of your data in this single table with columns called entity, attribute, and values.

ex)
| entity | attribute | value |
| ------- | --------- | ----- |
| person | id        | 1 |
| product | id       | 12 |
| cateory | name | 'shoes' |

#### Why Antipattern?

- you can't use data type. value can be any type otherwise, it lose its benefits.
- you can't use referential intergiry on attribute column. a value of attribute column can be anything. 

#### Solutions

- [Single Table Inheritance](#single-table-inheritance)
- [Concrete Table Inheritance](#concrete-table-inheritance)
- [Class Table Inheritance](#class-table-inheritance)
- [Semistructured Data (Serialized LOB)](#semistructured-data-serialized-lob)

## Techniques 

### Single Table Inheritance

create a inheritance with a single table. you accommodate all of fields in subclasses in a single table. 

ex) student and teacher subclasses are combined in 'person' table in database. 'gpa' column is only used by student and 'evaluation_score' is only used by teacher.

| id | name | type | gpa | evaluation_score | 
| ------- | --------- | ----- | ---- |
| 1 | 'john' | student | 3.4 | NULL |
| 2 | 'yoko' | teacher | NULL | 4.0 |
| 3 | 'satoshi' | teacher | NULL | 3.0 |

### Concrete Table Inheritance

create a separate table for each subclass, but this does not create a table for the superclass. therefore, the common fields are duplicated in each table for subclasses.

__table: teacher__
| id | name | evaluation_score | 
| ------- | --------- | ----- | 
| 1 | 'john' | 4.0 |
| 2 | 'yoko' | 5.0 |
| 3 | 'satoshi' | 3.0 |

__table: student__
| id | name | gpa | 
| ------- | --------- | ----- | 
| 1 | 'john' | 4.0 |
| 2 | 'yoko' | 5.0 |
| 3 | 'satoshi' | 3.0 |

### Class Table Inheritance

best?

mimic inheritance in database. create tables for a superclass and each subclasses.

the trick is use the same id in superclass and each subclass so that you know its subclass.

__table: person__
| id | name | 
| ------- | --------- | 
| 1 | 'john' |
| 2 | 'yoko' |
| 3 | 'satoshi' |

__table: student__
| id | gpa | 
| - | - | 
| 1 | 4.0 |
| 2 | 5.0 |
| 3 | 3.0 |

__table: teacher__
| id | evaluaton_score | 
| - | - | 
| 1 | 4.0 |
| 2 | 5.0 |
| 3 | 3.0 |

#### pros

- no duplication of columns

#### cons

- a little complciated sql statement (e.g., join) for SQL statement

### Semistructured Data (Serialized LOB) 

similar to [Single Table Inheritance](#single-table-inheritance), but you create a single column to store a set of attribute as JSON/XML so that you can serialize it at your applicaiton.

for example, 'attribute' column contains a json about additional keys and values based on the type (e.g., { gpa: 4.0, key1: value1, key2: value2 } for student type.

| id | name | type | attributes |
| - | - | - | - |
| 1 | 'john' | student | '{ gpa: 4.0 }' |
| 2 | 'yoko' | teacher | '{ evaluation_score: 3.0 }' |
| 3 | 'satoshi' | teacher | '{ evaluation_score: 4.2 }' |

#### pros

- extensible: you don't need to create additional columns if you need to add other fields.

#### cons

- you cannot use SQL features (e.g., query, where, and so on) on the 'attributes' column

### Polymorphic Association

this antipattern is used when you want to create multiple association for two different parent tables with a single foreign key. obviously, reference integrity does not allow you to store a single foreign key column from more than one tables.

![polymorphic assciation image](/polymorphic-association.png)


### Metadata Tribbles

this antipattern is used when you want to prevent a single table from increase the number of row for performance (e.g., the more data in a table, the worse the performance). 

What this antipattern allows you to do is that you create a table per a range of a value on an attibute. for example, you create a table based on the year (e.g., 2000, 2001, 2002, and so on) to prevent performance degradation. 

#### Why Anitipattern?

- hard to manage primary key. you need to manage the uniqueness of primary key across those tables.
- hard to manage query. you have to use 'union' to connect those tables.
- cannot use the same foreign key for those tables.
- hard to add a new column. you need to add a new column for each table and must consistent name & data. 

#### Solutions

1. __horizontal partitioning (esp sharding)__: 

- create replicas (e.g., duplicate schema) and partition rows (e.g., ID: 1 - 100 goes to replica 1 and ID: 201 - 400 goes to replica2, and so on)

2. __vertical partitioning with dependent table__:

- split a table with columns (esp, you split columns which is seldom used). specifically, you identify the columns takes a large size and seprate those columns in a dependent table. you can use the primary key as a foreign key for the depedent table. 

the benefit is that you can exclude thoes large columns from the main table and when you do queries, it improve performance. 

### Rounding Errors

use DECIMAL or NUMERIC if you need a value to be precise. don't use FLOAT. 

by specification, FLOAT store its value on base-2 format and some number cannot be represented in base-2 format precisely so need to be rounded, so when you do calculation/comparison, it does not work as expected.

### Flavors

how to store enum in database?

this antipattern recommends you to use ENUM type or similar data type. 

#### Why Anitipattern?

- hard to update the data type. what if you want to add another value to the enum? you need to redefine the data type again
- less portability. what if other database brands don't support ENUM or have different data type to support ENUM. 

#### Solutions

- __create a table to store the enum values__: rather than using data type, you can create a table to keep track of your enum values. 

table: user_types
| id | name |
| - | - | 
| 1 | 'guest' | 
| 2 | 'member' | 
| 3 | 'admin' | 

benefits:

- good portability. if you migrate to another database brands, it is easy since this is just a table
- easy to update. if you want to add another value, you can use 'insert' statement. 

### Phantom Files

image storage should be in database or filesystem. 

this textbook said that it should be database because of the following reasons:

1. easy to update. deleting image in database is much easier than one in filesystem since you can delete it with 'delete' statement automatically.
2. easy to migrate. you can easily migrate database to another if images are included in database.
3. respect access priviledge. if you store images in database, you can apply the access priviledge to the columns since it is inside database.
4. easy to do TX. you can easily wrap with transaction and do rollback easily. 

However, many developers are for filesystems.

1. https://www.quora.com/Is-it-better-to-store-images-in-a-database-or-a-file-system
2. https://stackoverflow.com/questions/3748/storing-images-in-db-yea-or-nay

it is up to you which you prefer.

### Index Shotgun

use indecies properly for performance.

#### Anitipatterns

- __no index__: the main benefit of indeces is faster access your desired rows for SELECT, UPDATE, DELETE rather than full scan of a table. 
- __too many index__: adding extra index on the same column does not give you benefits.

1. primary key column: primary key has an index so you don't need to additinal index.

#### Solutions

- __use MENTOR (Measure, Explain, Nominate, Test, Optimize, Rebuild)__: 

1. __Measure__: identify which query causes performance problems. you can use 'slow.query.log' in MySQL. it is important to focus on that you can get great benefits by this performance tuning since if you focus on the column which is not used for queries often, you are wasting your time. 

2. __Explain__: identify why the target query is so slow. use __query execution plan (QEP)__ (e.g., in MySQL, it is 'EXPLAIN' keyword) 

3. __Nominate__: 

#### Caveats When Performance Tuning

- disable cache.
- enable profiling only when performance tuning so that you can get enough information for it and disable after you done the performance tuning.


- create an index on a column only you need. 


## index (with MySQL):
	- goal: to find data from a tables quickly. without index, you need to find the data from the beginning. 
	- analogy: indeces on a book
	- trade off between reading (SELECT) and writing (INSERT, UPDATE, DELETE). adding an index makes the reading faster and makes writing slower and vice versa. this is because index are used to speed up the reading. when the writing, it requires the engine to modify the index table too so that's why adding index makes the writing slower. 
	
### clustered index:
	- only one clustered index per table
	- sort the data rows in the table based on the indexed column
	- in RDBMS, usually primary key is a clustered index
	
### non-clustered index (secondary indexes):
	- stores the data at one location and indeces at another location. 
	- the index contains pointers to the location of that data.
	- a single table can have amny non-clustered indexes because an index in the non-clustered in dex is stored in different places. 
	- improve the performance of quieries that use keys which are not assigned as a primary key. 
	
### B-tree index: 
	tree structure that has many children. (*binary tree has only 2 children).
	enabling fast lookup for exact matches (equals operator) and ranges (for example, greater than, less than, and BETWEEN operators).
	available for most storage engines, such as InnoDB and MyISAM
	for primary key, unique, index, and fulltext 
	
  - multiple level indeces 
  
  * what is diff btw B-Tree vs B+tree

ref (youtube): https://www.youtube.com/watch?v=aZjYr87r1b8&ab_channel=AbdulBari
  
### hash index:
	intended for queries that sue equality operators (= or <=> (null safe))
	only available at "Memory" storage engines 
	
### FULLTEXT index:

enable full text search if a column has FULLTEXT index.

FULLTEXT can help with that query if bar is a "word". FULLTEXT handles words, not arbitrary substrings (as LIKE does).

in MySQL, there are three modes

1. __natural language search type__: search a given word in a text collection (one or more text columns) and sort by the highest relevance first.
2. __query expansion type__: frequently used when the user relies on implied knowledge - for example, the user might search for “DBMS” hoping to see both “MongoDB” and “MySQL” in the search results.
3. __boolean search type__: you can customize query with operators (e.g., you want to search "words" not but "word2")

### covering index: 

covering index: a special case of an index in InnoDB where all required fields for a query are included in the index; in other words, the index itself contains the required data to execute the queries without having to execute additional reads.

simply said that if your columns in your query does not include, the engine need to refer to the original table besides the index. this makes performance worse if you don't include the columns in the index since it need to lookup the index (non-clustered) and the primary index (clustered). 

in order to prevent this, you can use covering index. you just need to include the columns in the index when creating the index.

![covering index image](./covering-index.png)

ref: [here](https://www.softel.co.jp/blogs/tech/archives/5139)





### Tips

1. don't use LIKE keywrod to search a text. use FULLTEXT search index for performance gains
2. the order of columns in a index matters. if you include the first column of the index in where/group by/order by clause, you can get benefits from the index, otherwise you cannot get benefits.

```
-- index 1
CREATE INDEX sample_index ON sample_tables (column1, column2, column3);

-- you CAN get benefits from the above index for the following query

SELECT * FROM sample_tables
WHERE column1 = 'XXX' -- column1 is the first column in the index so you can get benefit
AND column2 = 'YYY';

-- you CANNOT get benefits from the above index for the following query

SELECT * FROM sample_tables
WHERE column3 = 'XXX' -- colum1 is not included in the query so you cannot get the benefits
AND column2 = 'YYY'; 

-- index 2
CREATE INDEX sample_index ON sample_tables (column3, column2, column1);

-- you CANNOT get benefits from the above index for the following query

SELECT * FROM sample_tables
WHERE column1 = 'XXX' -- column3 is not included in the index so no benefits
AND column2 = 'YYY';

or 

SELECT * FROM sample_tables
WHERE column2 = 'XXX' -- only single colum which is not the first column the index does not help

-- you CAN get benefits from the above index for the following query

SELECT * FROM sample_tables
WHERE column3 = 'XXX' -- column3 is the first column so you can get benefit
AND column2 = 'YYY';

```

### filesort

you see 'using filesort' when you run a query with 'EXPLAIN'. this means that the engine need to sort the result since the natural order does not respect for the your query. this takes longer time to finish. 

ref: [here](https://stackoverflow.com/questions/65912380/what-does-using-filesort-mean-in-mysql)

## storage engines:
	- MyISAM
	- InnoDB
	- Memory

## Horizontal scaling: 

add more machines

## Vertical scaling 

add more powerful RAM, CPU

## Scaling up options

### Scaling up hardware

increase CPU, RAM, and so on (vertical scalling)

### Add replicas (could be problematic): add master (e.g., receive the write/read request from your app and coordinate slaves) and slave (replicas) 

- The disadvantage of replicas: eventual consistency might cause read access to get stale data. For example, after a write access to the master and still updating each slave for the write request, another request for the read reads the stale data from one of the slave. 

### Sharding (better solution? Too complicated):

Prepare the total n number of databases (called S1, S2, …, SN) and store a part of your data in each shard database. For example, data of customer id 1 to 50 is stored in S1, and data of customer id 51 to 100 are stored in S2, and so on. 
Need a logic (mapping table or another database to identify which customer id is stored in which shard database)
