# Installation

We will present later on how to generate or update a dependency graph database. By now we suppose we work with a snapshot provided online.

## Requirements

The Goblin framework relies on a Neo4J graph database. You can either have this using a containers (Docker), using Neo4J desktop (on you machine) or using Neo4J as an application server (on a remote machine).

The requirements are:

- if you install things and run using Docker: Docker
- if you install manually and run Neo4J using Neo4J Desktop: Java 17 to run the Goblin tools.
- if you install manually and run Neo4J as a server application: Java 17 **and** Java 11 (it seems that Neo4J as a server application is not working with Java 17).

## Installation using Docker

We provide Docker files to install the Graph Database and Goblin Weaver [here](https://github.com/Goblin-Ecosystem/Neo4jWeaverDocker).
Please note that using this you are not able to choose the graph database version by default (you may modify `Dockerfile.neo4j` if you want). Furthermore the user name is `neo4j` and the password is set to `Password1` (again it can be changed in `Dockerfile.neo4j`).

For the first run:

```sh
docker-compose up --build
```

For other runs:

```sh
docker-compose up
```

Then, proceed to the "Test of the Installation" section, below, to check your installation.

## Manual Installation of the Graph Database

1. install [Neo4J](https://neo4j.com/product/neo4j-graph-database/), see [here](https://neo4j.com/docs/operations-manual/4.4/installation/)
   
    You can either install the Desktop application (if you are experimenting on your computer) or the server application (if you are running on a server machine).

    **Important:** if you are using the Desktop application you will be able to select the DBMS version when importing the database dump. If you are using the server application then you **must** ensure that you are installing version 4.x.

2. download the database dump from [here](https://doi.org/10.5281/zenodo.13734581)

   We release newer versions from time to time. See the versions list there.

3. importing the database dump into Neo4J

    a. Desktop application

    - open Neo4J Desktop
    - an example project must have been created and opened (if not, please create one)
    - click on the "Add" dropdown menu and select "File"
    - find and select your dump file
    - put the cursor over the dump file name in the project file list, select "..." and click on "Create new DBMS from dump"
    - give a name to your new database, setup a password (and keep it somewhere), and **select the last 4.x version in the "Version" dropdown**
    - click on the "Create" button
    - the database appears on top of the project and you can click "Start" to start it
    
    b. Server application

    - Stop the database if it's running:
     ```sh
     sudo systemctl stop neo4j
     ```
    - Load the dump:
     ```sh
     sudo -u neo4j neo4j-admin load --from=/path/to/dump.dump --database=neo4j --force
     ```
    - Start neo4j:
     ```sh
     sudo systemctl stop neo4j
     ```
    - To access the database, issue the command:
    ```sh
     cypher-shell -u neo4j -p your_password
     ```

## Manual Installation of Goblin Weaver

1. Make sure that the Neo4j database containing the graph is running.
2. Download the Goblin Weaver jar [here](https://github.com/Goblin-Ecosystem/goblinWeaver/releases)
3. Open a terminal and run the following command (If needed, update the Neo4j user, password and uri):
```sh
java -Dneo4jUri="bolt://localhost:7687/" -Dneo4jUser="neo4j" -Dneo4jPassword="Password1" -jar goblinWeaver-2.1.0.jar
```

The program will first download the osv.dev dataset and create a folder called "osvData", it's takes approximately 3m.
For other runs, **if you don't want to update the CVE data**, you can add the "noUpdate" argument on the java -jar command like this:
```sh
java -Dneo4jUri="bolt://localhost:7687/" -Dneo4jUser="neo4j" -Dneo4jPassword="Password1" -jar goblinWeaver-2.1.0.jar noUpdate
```

## Test of the Installation

### Neo4j dataset
To verify the Neo4j dataset works, open a terminal and run the following command:

**Important:** If needed, update the Neo4j user, password and uri.
```sh
curl -H "Content-Type: application/json" -X POST \
     -u neo4j:Password1 \
     -d '{"statements": [{"statement": "MATCH (n) RETURN count(n)"}]}' \
     http://localhost:7474/db/neo4j/tx/commit
```

If it works, you should get a response like this
```sh
{"results":[{"columns":["count(n)"],"data":[{"row":[14077982],"meta":[null]}]}],"errors":[]}%
```

### Weaver
If the Weaver has launched without displaying an error, it is ready for use.
The Swagger documentation should therefore be available under this link:
[http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)

## Accessibility
- By default, Neo4j web interface will be accessible via http://localhost:7474
- By default, Weaver REST API will be accessible via http://localhost:8080