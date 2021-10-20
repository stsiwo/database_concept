# RDBMS (esp MySQL)

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
