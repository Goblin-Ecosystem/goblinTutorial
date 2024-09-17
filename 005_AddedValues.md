# Added values
As indicated on the page [003_WeaverAPI.md](003_WeaverAPI.md) of the tutorial, the Weaver allows adding values to the graph.
This tutorial helps to further understand these new nodes of the graph.

## Deletion of added values
### Automatic deletion
The added values can be automatically deleted by the Weaver.
#### CVE
CVE entries depend on the [OSV](https://osv.dev/) database. Upon the update of this database, all AddedValue nodes of type CVE are deleted from the database.

⚠️ The OSV database is updated automatically when the Weaver starts.
To prevent it from updating, add the `noUpdate` argument:
```sh
java -Dneo4jUri="bolt://localhost:7687/" -Dneo4jUser="neo4j" -Dneo4jPassword="Password1" -jar goblinWeaver-2.1.0.jar noUpdate
```

#### FRESHNESS, POPULARITY_1_YEAR, SPEED
These three metrics are calculated based on the information nodes present in the graph; they do not depend on external data.
Thus, when the Weaver starts, it checks if the graph has been updated. If it has been updated, it deletes these added values.

### Manual deletion
#### Weaver
The Weaver comes with routes that allow deleting the added values from the database.

To delete all added values, use the following route:   
**Method**: `DELETE`  
**ROUTE**: `/addedValues` 

To delete one or more specific added value(s), use the following route:  
**Method**: `DELETE`  
**ROUTE**: `/addedValue`   
**Body example**:

```json
{
  "addedValues": 
  ["CVE", "FRESHNESS"]
}
```

#### Cypher
You can also delete the AddedValue nodes by directly using Cypher on Neo4j.  
Example of a Cypher query:  
```cypher
:auto MATCH (n:AddedValue) WHERE n.type IN ['CVE', 'FRESHNESS'] CALL { WITH n DETACH DELETE n } IN TRANSACTIONS OF 10000 ROWS;
```

Here, we launch the transaction in batches of 10,000 to avoid overloading the memory, as the number of nodes of this type can reach millions. The `:auto` at the beginning of the query allows the use of transactions. As an example, this query without transactions would look like this:  
```cypher
MATCH (n:AddedValue) WHERE n.type IN ['CVE', 'FRESHNESS'] WITH n DETACH DELETE n
```

## Addition of added value
As mentioned on the [003_WeaverAPI.md](003_WeaverAPI.md) page, when you call the Weaver specifying a value to add, it will calculate and automatically add it to your Neo4j database.

The [Zenodo](https://doi.org/10.5281/zenodo.13734581) archive contains a database with the CVE, FRESHNESS, POPULARITY, and SPEED metrics already pre-calculated. However, if you wish to update the database or the CVE data, they will be automatically deleted (as they are no longer up-to-date). Also, if you want to add a custom metric or use aggregated metrics, you may want to populate the entire database rather than calculate them on the fly.

Here is a simple example of Java code to populate the database with a specific added value using the Weaver:  
```java
import org.json.simple.JSONArray;
import org.json.simple.JSONObject;

import java.io.IOException;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.util.List;

public class App
{
    private static final String API_URL = "http://localhost:8080";
    private static final int nodesMaxId = XXX;
    private static final int batchSize = 50000;

    public static void main( String[] args )
    {
        int startId = 0;
        for (int currentStart = startId; currentStart <= nodesMaxId; currentStart += batchSize) {
            int currentEnd = currentStart + batchSize - 1;
            String cypherQuery = String.format(
                    "MATCH (n:Release) " +
                            "WHERE id(n) >= %d AND id(n) <= %d " +
                            "RETURN n;",
                    currentStart, currentEnd
            );
            System.out.println(currentEnd + "/" + nodesMaxId);
            cypherQuery(cypherQuery, List.of("CVE"));
        }
    }

    public static void cypherQuery(String query, List<String> addedValues){
        String apiRoute = "/cypher";
        JSONObject bodyJsonObject = new JSONObject();
        bodyJsonObject.put("query", query);
        JSONArray jsonArray = new JSONArray();
        jsonArray.addAll(addedValues);
        bodyJsonObject.put("addedValues", jsonArray);
        executeQuery(bodyJsonObject, apiRoute);
    }

    private static void executeQuery(JSONObject bodyJsonObject, String apiRoute){
        try {
            URL url = new URL(API_URL+apiRoute);
            HttpURLConnection http = (HttpURLConnection) url.openConnection();
            http.setRequestMethod("POST");
            http.setRequestProperty("Content-Type", "application/json; utf-8");
            http.setRequestProperty("Accept", "application/json");
            http.setDoOutput(true);

            byte[] out = bodyJsonObject.toString().getBytes(StandardCharsets.UTF_8);

            OutputStream stream = http.getOutputStream();
            stream.write(out);

            if(http.getResponseCode() != 200){
                System.out.println("Error with query: \n "+bodyJsonObject);
            }
            http.disconnect();
        } catch (IOException e) {
            System.out.println("Unable to connect to API:\n" + e);
        }
    }
}
```
This code populates the database with one or more metrics by iterating over each node in batches of 50,000 to avoid overloading the memory.

To use this code, you need to make the following modifications:
- The Cypher query  `MATCH (n:Release)` hould be specified with the type of node to which the added values apply, such as `Release` or `Artifact`.
- Specify the added value(s) `List.of("CVE")` to indicate which added values to calculate.
- And finally, set the `nodesMaxId` variable to specify the maximum Neo4j ID present in the database. To find this value, execute the following Cypher query in Neo4j:  
```Cypher
MATCH (n)
RETURN max(id(n)) AS maxId
```