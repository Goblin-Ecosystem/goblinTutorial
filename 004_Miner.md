# GoblinMiner
The Miner source code is available on [GitHub](https://github.com/Goblin-Ecosystem/goblinDependencyMiner)

The Goblin miner allows you to generate and/or update a Maven Central dependency graph in a Neo4j database.

To get all Maven releases data, we use the Central index archive [here](https://repo.maven.apache.org/maven2/.index/)
Initially, this program will download the most recent archive and unpack it with the Maven Indexer CLI jar present on the lib folder.
This will create a "central-lucene-index" folder at the root of the project during the execution, this folder will be deleted at the end of the program.
Doc: https://maven.apache.org/repository/central-index.html

## Requirements
- Java 17
- Maven

## Configuration
### Configuration file
To run the application you need to edit the configuration file in: src/main/resources/configuration.yml.
- **dataBaseExport:** Choose the database you want to export data (can be Postgres, neo4J or both).
- **update:** Set true if you want to update an existing neo4j graph, set false to generate a graph from scratch.
- **thread:** Define the number of threads allocated to run the program.
### Database configuration
#### Postgres
To configure your Postgres database, you have to put your database information in the src/main/resources/META-INF/persistence.xml file.
#### Neo4J
To configure your Neo4J database, you have to put your database information in the src/main/resources/configuration.yml file.

## Run
**Generating the graph requires a lot of memory**, so we have to force the JVM to use 30 GB at execution time.
> _JAVA_OPTIONS="-Xmx30G" mvn clean compile exec:java

## Time
Generation and updating can be very time-consuming, times displayed below have been realized with 12 threads configuration and a machine with the following characteristics:
> OS: Red Hat Enterprise Linux <br>
> OS version: 8.7 <br>
> 16 CPUs:  Intel(R) Xeon(R) CPU E7-8880 v4 @ 2.20GHz <br>
> Memory: 64 GB <br>

Time to run the project from scratch on October 05, 2023: 4.1 days.  
Time to update our dataset from October 05, 2023, to October 14, 2023, (nine days): 1h06m.  
Time to update our dataset from April 14, 2023, to October 14, 2023, (six months): 6h23.  
Time to update our dataset from October 14, 2022, to October 14, 2023, (one year): 11h27m.