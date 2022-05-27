The project aims to implement the RDF Database by modifying its primary data structure, Triple which consists of Subject, Predicate and Object, into a Quadruple, which contains another field called Confidence. The project was executed as part of the course CSE 510 Database Management Systems implementation, and the execution was carried out in three phases.

PHASE 1
The first phase involved using command line prompts to test the features of Minibase, which is a database management system consisting of parser, optimizer, buffer pool manager, storage mechanisms (heap files, secondary indexes based on B+ Trees), and a disk space management system. This phase also involved verifying the functionality and implementation of Minibase using queries such as join tests, sort tests, index tests, heap file tests, disk management tests and buffer management tests.

PHASE 2
The second phase of the project consisted of modifying the underlying structure of Minibase which uses Tuples, to a new architecture using Quadruples. Each quadruple is formatted to have 4 fields: Subject, Object, Predicate and Confidence stored in some order selected by the user. The records are also indexed based on the user’s choice. The system enables the user to insert data into the database using the BatchInsert command. Post insertion, the user can query the database using the Query command to retrieve records from the database. Depending on a few factors like duplicates in data and type of indexing, the number of Page counts, Read counts and Write counts will vary. We leverage several classes in Minibase like BTree, Disk Manager, Global, Heap and Iterator and have also used our own class DBUtils which contains classes to perform operations into the db. We also use the predefined $makedb class to compile the db prior to running commands.

We implemented the program batchinsert which is invoked by the command:
                                                  java dbUtils.BatchInsert DATAFILE INDEXOPTION RDFDBNAME
                                                  
The program first checks if there is an existing DB and deletes all the records present in the DB in order to insert new records based on the index option given. The new data entered from the DATAFILE is first stored in a heap file (tempresult). The sorting occurs based on the index option chosen, specified in the INDEXOPTION variable. Based on this INDEXOPTION, we create a BTree file which will be used for retrieving the records later using indexing.

The tempresult heap file is used to sort the quadruples according to the INDEXOPTION. The indexing options that we implemented are as follows:
Note: <key, value> ordered pair is used for all the indexing options given below. A quadruple BTree file is created (Quadruple_BTreeIndex) where these ordered pairs are stored for indexing.
1. BTree index file on confidence: The key in this case is confidence and the values are the matching QIDs formed as <confidence, QIDs>. Whenever there is a query request, the Quadruple_BTreeIndex file is used to fetch the matching QIDs which are indexed by the confidence key.
2. BTree index file on subject and confidence: The key in this case is a combination of subject and confidence, and the values are the matching QIDs formed as <subject:confidence, QIDs>. Whenever there is a query request, the Quadruple_BTreeIndex file is used to fetch the matching QIDs which are indexed by subject and confidence key.
3. BTree index file on object and confidence: The key for this particular case is a combination of object and confidence, and the values are the matching QIDs formed as <object:confidence, QIDs>. Whenever there is a query, the Quadruple_BTreeIndex file is used to fetch the matching QIDs which are indexed by object and confidence key.
4. BTree index file on predicate and confidence: The key for this particular case is a combination of predicate and confidence, and the values are the matching QIDs formed as <predicate:confidence, QIDs>. Whenever there is a query based on predicate and confidence, the Quadruple_BTreeIndex file is used to fetch the matching QIDs which are indexed by predicate and confidence.
5. BTree index file on predicate: The key in this case is the predicate, and the values are the matching QIDs formed as <predicate, QIDs>. Whenever there is a query, the Quadruple_BTreeIndex file is used to fetch the matching QIDs which are indexed by the predicate key.
The openStream method is invoked by passing a subjectFilter, predicateFilter, objectFilter and confidenceFilter. The method returns a stream of quadruples, whose subject label is identical to subjectFilter, predicate label is identical to predicateFilter, object label is same as objectFilter, and confidence is greater than or equal to the confidenceFilter. The order in which the stream of quadruples is returned depends on the orderType which is passed as an argument to the openStream method.
![sortingschemes](https://user-images.githubusercontent.com/74524978/170631353-48b3c852-6f74-4702-8252-6f01d2e8fb74.png)

This is done by forming an unclustered BTree sorted heap file or index file on the labels as per the sorting scheme for the given orderType. This is performed after destroying the existing index by deleting the records having that key and QID.

PHASE 3
The third phase of the project is intended to modify the Tuple class, to create a BasicPattern class, storing nodeIDs. The BasicPattern class has a default parameter Confidence. The Iterator class is tailored to form BPIterator class, which iterates over Basic Patterns. The BP_Triple_Join class is implemented to perform sort-merge join operations on the generated Basic Patterns with RDF Quadruples based on Subject or Object node ID. The class contains functions like get_next(), close(), etc. to aid with this functionality. The BPSort class helps in sorting the Basic Pattern using the BPIterator class based on input parameters (labels of specified nodes or confidence). To perform the join operation, we implemented 3 different strategies, namely, tuple-oriented join (derived from Simple Nested Loop Join) with a time complexity of O(mn), hash oriented join (derived from Block Nested Loop Join) with a time complexity of O(n), and sort merge join. 
Here, m refers to the number of basic patterns (on the left side of the join) and n is the number of filtered records (on the right side of the join).

The sample query format is given below.
S( J(
    J([SF1,PF1,OF1,CF1],
      JNP,JONO,RSF,ROF,RCF,LONP,ORS,ORO
    ),
    JNP,JONO,RSF,ROF,RCF,LONP,ORS,ORO
  )
  SO, NP
)

The acronyms are expanded are follows:
S - select
J - join
SF1 - subject filter
PF1 - predicate filter
OF1 - object filter
CF1 - confidence filter
JNP - join node position
JONO - join on object
RSF - right subject filter
ROF - right object filter
RCF - right confidence filter
LONP - left out node position
ORS - output right subject
ORO - output right object
SO - sort order
NP - number of pages

Our interface is a command line format which is invoked by the following command:
                                              java dbUtils.BatchInsert tests/8-\ phase3_test_data.txt 1 "db14"
                                                  java JoinQuery.java "vkdb1_1" dbUtils/queryfile 100
                                                  
We can also retrieve information per run like page, read and write counts and number of subjects, objects, predicates, etc. and perform high-level analysis on them to identify root causes for the varying count values. Built on top of our phase 2 implementation, we have implemented 3 strategies to perform join in our RDF database. For every join strategy, we have built a testbench of varying data size and buffer size and recorded the observations.
![runtime](https://user-images.githubusercontent.com/74524978/170632003-4272483f-2c29-46e6-ad7c-7a5773b2347e.png)

Since this was part of coursework, the code is not included in this repository. 
