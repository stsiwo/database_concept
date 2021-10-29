# RDBMS (esp MySQL)

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



3. Closure Tables



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
