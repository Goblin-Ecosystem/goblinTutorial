# Weaver API
The Goblin Weaver REST API is available as an alternative for direct access to the database using the Cypher language and for on-demand enrichment of the dependency graph with new information. A memoization principle is available to avoid re-computing enrichments, as soon as the base graph is not re-computed or incremented. For this, new kinds of nodes (type AddedValue) and edges (type addedValues from an Artifact or Release node to an AddedValue node) are used in the graph database. One should be careful, as the graph is large, calculating metrics (especially aggregate ones) for the whole graph can be time-consuming.

## Added values
Currently, the weaver can compute the following added values:

- CVE: We use the [osv.dev](https://osv.dev/) dataset to get CVEs information.
- CVE_AGGREGATED: Aggregate release and dependencies (with transitivity) CVE.
- FRESHNESS: Corresponds, for a specific release, to the number of more recent releases available and to the time elapsed in milliseconds between the specific release and the most recent release.
- FRESHNESS_AGGREGATED: Aggregate release and dependencies (with transitivity) freshness.
- POPULARITY_1_YEAR: Corresponds, for a specific release, the number of version released within a maximum of one year after the current graph date using the specified release.
- POPULARITY_1_YEAR_AGGREGATED: Aggregate release and dependencies (with transitivity) POPULARITY_1_YEAR.
- SPEED: Corresponds to the average number of releases per day of an artifact. More information [here](https://benevol2022.github.io/papers/DamienJaime.pdf).

## Use the Weaver
Pre-designed requests are available, but you can also send your own Cypher requests directly to the API.
You can add to the body query for the API a list of Added values, and it will enrich the result for you.

The Weaver API comes with its Swagger documentation: http://localhost:8080/swagger-ui/index.html

### Example: new versions of a release with metrics
Here, we want to know which are the latest versions available after jgrapht-core 1.5.0 and add to that their CVE, freshness and popularity information.

To do that, we use a pre-defined route

**Method**: `POST`  
**ROUTE**: `/release/newVersions`   
**Body**:

```json
{
  "groupId": "org.jgrapht",
  "artifactId": "jgrapht-core",
  "version": "1.5.0",
  "addedValues": ["CVE", "FRESHNESS", "POPULARITY_1_YEAR"]
}
```

**Response**
```json
{
  "nodes": [
    {
      "cve": [],
      "id": "org.jgrapht:jgrapht-core:1.5.1",
      "nodeType": "RELEASE",
      "freshness": {
        "numberMissedRelease": "1",
        "outdatedTimeInMs": "66882028000"
      },
      "version": "1.5.1",
      "popularity_1_year": 105,
      "timestamp": 1616171280000
    },
    {
      "cve": [],
      "id": "org.jgrapht:jgrapht-core:1.5.2",
      "nodeType": "RELEASE",
      "freshness": {
        "numberMissedRelease": "0",
        "outdatedTimeInMs": "0"
      },
      "version": "1.5.2",
      "popularity_1_year": 953,
      "timestamp": 1683053308000
    }
  ]
}
```

### Exemple: Cypher query
Routes can be used to simplify client use by not using Cypher, or to create more complex queries (e.g. a sequence of queries with processing between them).
But it's also possible to send Cypher directly to the Weaver and ask it to enrich the result.

For example, the query below using Cypher retrieves all log4j-core versions and adds the CVEs associated with each of them:

**Method**: `POST`  
**ROUTE**: `/cypher`    
**Body**:
```json
{
  "query": "MATCH (a:Artifact) WHERE a.id='org.apache.logging.log4j:log4j-core' WITH a MATCH (a)-[e:relationship_AR]->(r) RETURN r",
  "addedValues": ["CVE"]
}
```

## Extend the Weaver
The Weaver is designed to be extensible, allowing a user to easily add information their research need.

The Weaver source code is available on [GitHub](https://github.com/Goblin-Ecosystem/goblinWeaver)

### Build

**Requirements:** Java 17, Maven

To build the project, run:
```sh
mvn clean package 
```

### Add new added values
1. Go to weaver/addedValue/AddedValueEnum and add the name of your new value.
2. Fill the three methods of this enumeration with your new added value
3. Create a new class that extends weaver/addedValue/AbstractAddedValue.
4. (optional) If you also want to create an aggregated value of your new added value, create a new class that extends your previous new class and implements the "AggregateValue" interface.
5. Write your internal logic in this new class.

### Add new routes
1. The routes files are available in: `src/main/java/com/cifre/sap/su/goblinWeaver/api/controllers`
2. Here, we have one class per route type (release, artifact, graph, cypher). Open the file corresponding to your route or create a new one.
3. We use the Spring framework to build our API, you only need to create a new method with the following structure:
```java
@Operation(
    description = "My description",
    summary = "My summary"
)
@PostMapping("/my/route")
public JSONObject myNewRoute(@RequestBody MyQuery myQuery) {
} 
```

The body content (define by `@RequestBody`) is defined by the classes present in `src/main/java/com/cifre/sap/su/goblinWeaver/api/entities`. You can reuse an existing class or create a new one.