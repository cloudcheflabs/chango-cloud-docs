# Streaming Ingestion using Chango Client

External applications can insert incoming streaming json events to chango directly.

## Install Chango Client Library

Add chango client as dependency in maven.

```
<dependency>
  <groupId>co.cloudcheflabs.chango</groupId>
  <artifactId>chango-client</artifactId>
  <version>1.1.0</version>
</dependency>
```

Or, download chango client library which needs to be added to your application classpath.

```
curl -L -O https://github.com/cloudcheflabs/chango-client/releases/download/1.1.0/chango-client-1.1.0-executable.jar;
```

## Create Iceberg Table before sending json to chango

You need to create iceberg table beforehand. See [here](cli.md#create-iceberg-table-before-json-data-ingestion).


## Example of sending json to chango using Chango Client Library

You may construct chango client instance like this.

```
import co.cloudcheflabs.chango.client.component.ChangoClient;

...


String adminServer = "https://chango-admin-oci.cloudchef-labs.com";
String user = "user1";
String password = "anypassword";
String dataApiServer = "https://chango-data-api-jetty-oci-user1.cloudchef-labs.com";
int batchSize = 10000;
long interval = 1000;
String schema = "iceberg_db";
String table = "test_iceberg";

ChangoClient changoClient = new ChangoClient(adminServer,
        user,
        password,
        dataApiServer,
        schema,
        table,
        batchSize,
        interval);      
```

- `adminServer` : used to get access token after login.
- `user`: chango user name.
- `password` : chango password.
- `dataApiServer` : chango data api server url for json.
- `schema`: iceberg schema which needs to be created before sending json data to chango.
- `table`: iceberg table which also needs to be created beforehand.
- `batchSize` : the size of json list which will be sent to chango in batch mode and in gzip format.
- `interval` : json data will be queued internally in chango client. The queued json list will be sent in this period whose unit is milliseconds.

And then just call `add()` .

```
        String json = ... 
       // add json coming from your application to the internal queue in chango client.    
       changoClient.add(json);
```

## Query Iceberg Table using Trino Clients

Json data ingested to iceberg table in chango can be queries using trino clients like Trino CLI, Redash and Metabase.

You can run the following query if json data is ingested to chango.

```
select * from iceberg.iceberg_db.test_iceberg limit 1000;
```