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




## index (with MySQL):
	- goal: to find data from a tables quickly. without index, you need to find the data from the beginning. 
	- analogy: indeces on a book
	
## clustered index:
	- only one clustered index per table
	- sort the data rows in the table based on the indexed column
	- in RDBMS, usually primary key is a clustered index
	
## non-clustered index (secondary indexes):
	- stores the data at one location and indeces at another location. 
	- the index contains pointers to the location of that data.
	- a single table can have amny non-clustered indexes because an index in the non-clustered in dex is stored in different places. 
	- improve the performance of quieries that use keys which are not assigned as a primary key. 
	
	
## B-tree index: 
	tree structure that has many children. (*binary tree has only 2 children).
	enabling fast lookup for exact matches (equals operator) and ranges (for example, greater than, less than, and BETWEEN operators).
	available for most storage engines, such as InnoDB and MyISAM
	for primary key, unique, index, and fulltext 
	
  - multiple level indeces 
  
  * what is diff btw B-Tree vs B+tree

ref (youtube): https://www.youtube.com/watch?v=aZjYr87r1b8&ab_channel=AbdulBari
  
## hash index:
	intended for queries that sue equality operators (= or <=> (null safe))
	only available at "Memory" storage engines 
	
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
