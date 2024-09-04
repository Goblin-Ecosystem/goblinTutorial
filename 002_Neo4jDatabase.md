# Neo4j database
Database dumps are available on [Zenodo](https://doi.org/10.5281/zenodo.13683940)

The dependency graph database is composed of two node types (for libraries and for their releases) and two edge types (from releases to their dependencies and from libraries to their releases). The nodes for libraries (type Artifact) contain the Maven id (g.a) information. The nodes for releases (type Release) contain the Maven id (g.a.v), the release timestamp, and the version information. The edges for dependencies (type dependency) are from Release nodes to Artifact nodes and contain target version (which can be a range) and scope (compile, test, etc). The edges for versioning (type relationship_AR) edges are from Artifact nodes to Release nodes.

![](./img/Goblin_Neo4J_Dependency_Graph.png "Graph structure")

The latest version of our dataset, dated August 30th, 2024, contains 15,117,217 nodes (658,078 libraries and 14,459,139 releases) and 134,119,545 edges (119,660,406 dependencies and 14,459,139 versioning edges).

## Neo4j Cypher querying

Here, we will presents basic examples of Cypher queries on the Neo4j Maven Central dependency graph.
Cypher is Neo4j’s declarative query language.

### Get a Release Node
```cypher
MATCH (r:Release) WHERE r.id='org.jgrapht:jgrapht-core:1.5.2' return r
```

### Get all Library versions
To retrieve Release nodes only, use this request:
```cypher
MATCH (a:Artifact) WHERE a.id='org.jgrapht:jgrapht-core'
WITH a
MATCH (a)-[e:relationship_AR]->(r)
RETURN r
```

To retrieve the Artifact node, the edges and the Releases, use this request:
```cypher
MATCH (a:Artifact) WHERE a.id='org.jgrapht:jgrapht-core'
WITH a
MATCH (a)-[e:relationship_AR]->(r)
RETURN a, e, r
```

### Get Release direct dependencies
To get all direct dependencies of a Release, use this request:
```cypher
MATCH (r:Release) WHERE r.id='org.jgrapht:jgrapht-core:1.5.2'
WITH r
MATCH (r)-[e:dependency]->(a)
RETURN r, e, a
```

If you want to filter dependencies to remove test dependencies for example, do:
```cypher
MATCH (r:Release) WHERE r.id='org.jgrapht:jgrapht-core:1.5.2'
WITH r
MATCH (r)-[e:dependency]->(a) WHERE e.scope<>'test'
RETURN r, e, a
```

### Get Release dependents
Note that release versions contained in dependency edges can be ranges (e.g. [1.0,2.0)).
The following query therefore does not take into account the resolution of this type of dependency.

```cypher
MATCH (r:Release)-[d:dependency]->(a:Artifact)
WHERE a.id = 'org.jgrapht:jgrapht-core' AND d.targetVersion = '1.5.2'
RETURN r, d, a
```

## Neo4j Programmatic querying
For more complicated queries (such as iterations or recursivity), it may be simpler to proceed programmatically, using the neo4j drivers available in the various programming languages.

The [Weaver code](https://github.com/Goblin-Ecosystem/goblinWeaver) shows examples of how to use the Neo4j database in Java.
