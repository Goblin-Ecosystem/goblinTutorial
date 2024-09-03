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

2. download the database dump from [here](https://doi.org/10.5281/zenodo.11104819)

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

     - to appear

## Manual Installation of Goblin Weaver

## Test of the Installation

