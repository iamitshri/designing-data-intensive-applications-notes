# Chapter 2. Data Models and Query Languages

- In this chapter, we will look at range of data models for the data storage and querying.
- We will compare relational mode, the document model and a few graph based data model
- We will also look at query languages and compare their use cases

## Relation Model vs Document Model

### Relational model proposed by Edgar Codd in 1970
- From mid 1980 to late 2000s, relational database were dominant
- Goal of relational database was to hide the implementation detail behind cleaner interface

### NoSql
  - Originally intended simply as a catchy Twitter hashtag for a meetup on open source, distributed, non-relational databases in 2009
  - Driving forces behind Nosql
    - A Need for greater scalability, including very large datasets or very high write throughput
    - A widespread preference for free and open source software over commercial db product
    - Specialized query operation that are not supported by relational model
    - Frustration with the restrictiveness of relational schemas
      - Desire for a more dynamic and expressive data model
  - Different applications have different requirements, so in the future polyglot persistence will be common

  - The Object Relational Mismatch
    - Translation layer to convert table, rows, columns into objects in the application code
    - ORM frameworks help to reduce boilerplate code. However, they can not completely hide the difference

  - Document-Oriented databases like MongoDB, RethinkDB, CouchDB and Espresso support JSON
  - Document Mode JSON  
    - Reduces the impedance mismatch between application code and the storage layer
    - One to Many representations fits well with JSON model
    - Better locality than multi table schema. One query is enough to fetch all the data in json representation
    - Support for Many to One is weak due to limited Join support
      - Application code has to do the Join 
  - Data has tendency of becoming more interconnected as features are added to applications
  - Relations DB
    - Query optimizer automatically decides which parts of the query to execute in which order and which indexes to use.
    - Easier to add new features. 
    - Easier to support new query patterns by creating indexes. Queries dont have to change to take advantage of the indexes. Query optimizer takes care of that
    - Key insight: Query Optimizer is built once and then all applications that use the db can benefit from it
  - Comparison
    - Document Data Model
      - Schema Flexibility
      - Better performance due to locality
      - Model is closer to the data structures used by the application code
    - Relational DM
      - Support for Join, Many to one, Many to Many
  - Which Data model leads to simpler application code?
    - If your data has tree like structure i.e. tree of one-to-many relationships, where typically entire tree is loaded then its good idea to use a document model
    - In relational world, emulating document like structure by creating relations using multiple tables can lead to complicated code 
    - If your app uses many-to-many relations then document DB is less appealing
      - Multiple queries are required for joins
      - Joins performed by the code are slower than database 
    - It's not possible to say in general which data model leads to simpler application code. 
      - It depends on the kinds of relationships that exist between data items
      - For highly interconnected data, 
        - the document model is awkward
        - Relational is acceptable
        - Graph models are the most natural
  - Schema flexibility in the document model
    - Document DB has implicit schema not enforced by the database
    - Schema-On-Read vs Schema-On-Write
    - In relations databases changing schemas can be slow and can require downtime
  - Data locality for queries
    - Documents can be better if the application needs to load entire json always compared to using joins to gather data from multiple tables
    - Locality advantage applies only if you need large part of document at the same time.
    - The database typically needs to load the entire document, even if you access only a small portion of it, which can be wasteful on large documents.
    - It is generally recommended that you keep documents fairly small and avoid writes that increase the size of a document. 
    - On updates to a document, the entire document usually needs to be rewritten—only modifications that don’t change the encoded size of a document can easily be performed in place
    - These performance limitations significantly reduce the set of situations in which document databases are useful.
    - Locality is not just the property of the document model
      - In relational world, Google's spanner db offers the locality properties in a relational model by allowing the schema to declare that table's row should be in the interleaved (nested) within the parent table
      - Oracle offers the feature called multi table index cluster tables 
      - The Column family concept in the bigtable data model (used in Cassandra and Hbase) has a similar purpose of managing locality
  - Convergence of document and relational databases
    - Relational DBs support XML data
      - Ability to query, index and make local modification within the xml document
    - Postgres, mysql has support of JSON datatypes
    - On document side:
      - RethinkDB supports relation like joins in its query language
      - MongoDB drivers automatically resolve document references 
        - (effectively performing a client-side join, although this is likely to be slower than a join performed in the database since it requires additional network round-trips and is less optimized)
    - It seems that relational and document databases are becoming more similar over time,
#### Query Languages for Data
  - Imperative vs Declarative query language
  - Sql is declarative. Hides the implementation details for the database engine
  - Declarative languages often lend themselves to parallel execution.
    - They only specify the pattern of the result not the algorithm to get the result 
    - Today CPUs are getting faster by adding more cores not by running clock at higher speeds than before
  - Imperative code is hard to parallelize across multiple cores and machines because it specifies commands to be executed in specific order

#### MapReduce Querying

  - map and reduce are pure functions  
  - map function: Executes the logic 
  - reduce function: Generally accumulates result into a single value

#### Graph like Data models
 - Great for modeling complex many-to-many relations
 - Vertices and edges
   - Social graph(people and their friends), web graph(page and links to other pages) etc.
 - In a graph database, you can refer directly to any vertex by its unique ID, or you can use an index to find vertices with a particular value
 - In a graph database, vertices and edges are not ordered (you can only sort the results when making a query).
 - In a graph database, you can write your traversal in imperative code if you want to, but most graph databases also support high-level, declarative query languages such as Cypher or SPARQL.
 - Property graph model implemented by Neo4j, Titan, InfiniteGraph
 - Declarative query languages for graphs: Cypher, SPARQL, and Datalog
 - Graphs are good for evolvability: as you add features to your application, a graph can easily be extended to accommodate changes in your application’s data structures.
 - After one example that demonstrated complex sql query to mimic the same query Cypher author says
   - If the same query can be written in 4 lines in one query language but requires 29 lines in another, that just shows that different data models are designed to satisfy different use cases. It’s important to pick a data model that is suitable for your application.
 - SPARQL is a query language for triple-stores using the RDF data model. (It is an acronym for SPARQL Protocol and RDF Query Language, pronounced “sparkle.”) It predates Cypher, and since Cypher’s pattern matching is borrowed from SPARQL, they look quite similar
 - Datalog’s data model is similar to the triple-store model, generalized a bit. Instead of writing a triple as (subject, predicate, object), we write it as predicate(subject, object)

#### Summary
- Historically data started out as one big tree ( hierarchical model), but many to many relations were hard to represent so the relational model was invented
- Recently Nosql was discovered as relational model did not fit well for certain applications
  - Document databases are good when data comes in self-contained document and the relations between documents are rare
  - Graph databases satisfy use cases where anything is related to everything
- All three models (document, relational, and graph) are widely used today, and each is good in its respective domain.
- One model can be emulated in terms of another model—for example, graph data can be represented in a relational database—but the result is often awkward. 
- That’s why we have different systems for different purposes, not a single one-size-fits-all solution.
- Document and graph model dont have strict schema, this is good for adapting to changing requirements but schema is assumed on th read
- Trade off between explicit schema(enforced at write) vs implicit (assumed on read)
- Each data model comes with its own query language or framework, and we discussed several examples: SQL, MapReduce, MongoDB’s aggregation pipeline, Cypher, SPARQL, and Datalog. We also touched on CSS and XSL/XPath, which aren’t database query languages but have interesting parallels.