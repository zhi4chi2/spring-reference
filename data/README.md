- [JPA](/data/jpa/README.md)
- [Spring Data 实战](/data/book/README.md)


releasetrain 即 BOM


releasetrain 的版本使用字符串，这是因为 modules 之间的版本不一致。 Spring Data is an umbrella project consisting of independent projects with, in principle, different release cadences. 


版本字符串其实是 names of famous computer scientists and software developers ，以字母顺序排序。所以 Ingalls > Hopper


版本还有些后缀
- -M1 - Milestones
- -RC1 - Release candidate
- -RELEASE - GA version
- -SR1 - Services release (bugfixes) for that release train


main modules
- Spring Data Commons - Core Spring concepts underpinning every Spring Data project.
- Spring Data Gemfire - Provides easy configuration and access to GemFire from Spring applications.
- Spring Data JPA - Makes it easy to implement JPA-based repositories.
- Spring Data KeyValue - Map-based repositories and SPIs to easily build a Spring Data module for key-value stores.
- Spring Data LDAP - Provides Spring Data repository support for Spring LDAP.
- Spring Data MongoDB - Spring based, object-document support and repositories for MongoDB.
- Spring Data REST - Exports Spring Data repositories as hypermedia-driven RESTful resources.
- Spring Data Redis - Provides easy configuration and access to Redis from Spring applications.
- Spring Data for Apache Cassandra - Spring Data module for Apache Cassandra.
- Spring Data for Apache Solr - Spring Data module for Apache Solr.


Community modules
- Spring Data Aerospike - Spring Data module for Aerospike.
- Spring Data Couchbase - Spring Data module for Couchbase.
- Spring Data DynamoDB - Spring Data module for DynamoDB.
- Spring Data Elasticsearch - Spring Data module for Elasticsearch.
- Spring Data Hazelcast - Provides Spring Data repository support for Hazelcast.
- Spring Data Neo4j - Spring based, object-graph support and repositories for Neo4j.


Related modules
- Spring Data JDBC Extensions - Provides extensions to the JDBC support provided in the Spring Framework.
- Spring for Apache Hadoop - Simplifies Apache Hadoop by providing a unified configuration model and easy to use APIs for using HDFS, MapReduce, Pig, and Hive.
- Spring Content - Associate content with your Spring Data Entities and store it in a number of different stores including the File-system, S3, Database or Mongo’s GridFS.


Ingalls 中包含 modules
- Spring Data Commons
- Spring Data JPA
- Spring Data KeyValue
- Spring Data LDAP
- Spring Data MongoDB
- Spring Data Gemfire
- Spring Data REST
- Spring Data Redis
- Spring Data for Apache Cassandra
- Spring Data for Apache Solr
- Spring Data Couchbase (community module)
- Spring Data Elasticsearch (community module)
- Spring Data Neo4j (community module)
- Spring Data Envers


# Reference
- http://projects.spring.io/spring-data/
- https://github.com/spring-projects/spring-data
- https://github.com/spring-projects/spring-data-examples
- https://github.com/spring-projects/spring-data-examples/tree/master/bom
