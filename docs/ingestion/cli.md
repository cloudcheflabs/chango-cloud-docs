# Chango CLI

To ingest external data to chango, chango provides Chango CLI to load json on local and S3 to chango as iceberg table. 
The chango client also provides to load local excel data to chango as iceberg table.

## Install Chango CLI

Chango CLI is written in Java. It is assumed that Java 11 needs to be installed before.  
Run the following commands to install Chango CLI.

```
curl -L -O https://github.com/cloudcheflabs/chango-client/releases/download/1.1.0/chango-client-1.1.0-executable.jar;
cp chango-client-1.1.0-executable.jar ~/bin/chango;
chmod +x ~/bin/chango;
```

Run chango, then it looks like this.

```
chango

Usage: chango [COMMAND]
Chango Client Console.
Commands:
  login   Login to Chango.
  upload  Upload CSV, JSON, Excel data to Chango.
```

On windows, run the following.

```
java -jar chango-client-1.0.0-executable.jar [Command]
```

## Login

You need to be logged in to proceed other jobs provided by Chango CLI.

```
chango login --admin-server https://chango-admin-oci.cloudchef-labs.com;
```

## Ingest Json Data to Chango

There are two options supported by Chango CLI to ingest json data to Chango CLI. Using Chango CLI, local json data can be ingested to chango. 
Json data located in S3 also can be ingested to chango.

### Create Iceberg Table before Json Data Ingestion

Before sending json data to chango, Iceberg table needs to be created which can be done, for example, using Redash provided by chango.

For example, first create iceberg schema and then create iceberg table with some properties using trino clients.

```
-- create iceberg schema.
CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;

-- create iceberg table.
CREATE TABLE iceberg.iceberg_db.test_iceberg (
    baseproperties ROW(eventtype varchar, 
                       ts bigint, 
                       uid varchar, 
                       version varchar), 
    itemid varchar, 
    price bigint, 
    quantity bigint 
)
WITH (
    partitioning = ARRAY['itemid'],
    format = 'PARQUET'
);
```

> **_NOTE:_** Take a note that the sequence  of table column names in lower case must be alphanumeric in ascending order.

### Local Json Data to Chango

To send json data in local json file, you need to know the Chango Data API Server URL.

Take a look at the Chango Data API Server URL has the following convention.

```
https://chango-data-api-jetty-oci-<user>.cloudchef-labs.com
```
, where `<user>` is the chango user name.

For instance, we will use https://chango-data-api-jetty-oci-user1.cloudchef-labs.com with the chango user `user1`.

```
chango upload json local \
--data-api-server https://chango-data-api-jetty-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table test_iceberg \
--file /home/opc/multi-line-json.json \
--batch-size 150000 \
;
```

- `schema` : iceberg schema created before.
- `table`: iceberg table where json data will be ingested in chango.
- `file` : local json file path.
- `batch-size` : list of json in gzip will be sent in batch mode. the size of json list.

You can send all json files in the directory.

```
chango upload json local \
--data-api-server https://chango-data-api-jetty-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table test_iceberg \
--directory /home/opc/json-files \
--batch-size 150000 \
;
```

- `directory`: the directory where json files are located which will be sent to chango.

### Json Data on S3 to Chango

Json data located on S3 also can be sent to chango.

For instance, you type the specific object name in S3 like this.

```
chango upload json s3 \
--data-api-server https://chango-data-api-jetty-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table test_iceberg \
--bucket <bucket> \
--access-key <access-key> \
--secret-key <secret-key> \
--endpoint <endpoint> \
--object-name chango/json-files/multi-line-json.json \
--batch-size 150000 \
;
```

- `bucket` : S3 bucket name.
- `access-key` : S3 access key.
- `secret-key` : S3 secret key.
- `endpoint` : S3 endpoint url.
- `object-name` :  object name which is json file to be sent to chango.

You can also send all the objects inside the specified object name.

```
chango upload json s3 \
--data-api-server https://chango-data-api-jetty-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table test_iceberg \
--bucket <bucket> \
--access-key <access-key> \
--secret-key <secret-key> \
--endpoint <endpoint> \
--object-name chango/json-files/ \
--batch-size 150000 \
;
```

Take a look at object-name is the object `chango/json-files/` . It means, all the json files located inside of this object will be sent to chango.


### Query Iceberg Table using Trino Clients

Json data ingested to iceberg table in chango can be queries using trino clients like Trino CLI, Redash and Metabase.

You can run the following query if json data is ingested to chango.

```
select * from iceberg.iceberg_db.test_iceberg limit 1000;
```

## Ingest Excel Data to Chango


There is no need to create iceberg table before sending excel data to chango. Chango will create iceberg table automatically.

> **_NOTE:_** Take a note that the header of excel must exist.

You can also define a iceberg table(for example, iceberg table with the definition of partition columns) before uploading excel to chango.

### Local Excel to Chango

Take a look at Chango Data API Server for Excel is different from the one for Json. It has the following convention.

```
https://chango-data-api-oci-<user>.cloudchef-labs.com
```

Let’s send excel to chango. For example, we will use the chango user `user1` .

```
chango upload excel local \
--data-api-server https://chango-data-api-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table excel_to_json \
--file /home/opc/chango/data-api/src/test/resources/data/excel-to-json2.xlsx \
;
```

### Query Iceberg Table using Trino Clients

You can run the following query if excel data is ingested to chango.

```
select * from iceberg.iceberg_db.excel_to_json limit 1000;
```

## Ingest CSV Data to Chango

As like Excel, you don’t have to create iceberg table beforehand. Chango will create iceberg table automatically.

> **_NOTE:_** Take a note that the header of csv must exist.

You can also define a iceberg table(for example, iceberg table with the definition of partition columns) before uploading csv to chango.


### Local CSV to Chango

Take a look at Chango Data API Server for CSV is different from the one for Json. It has the following convention like Excel file upload.

```
https://chango-data-api-oci-<user>.cloudchef-labs.com
```

Let’s upload CSV to chango. For example, we will use the chango user `user1` .

The following example is to upload CSV with the separator of comma.

```
chango upload csv local \
--data-api-server https://chango-data-api-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table csv_to_json_comma \
--separator "," \
--is-single-quote false \
--file /home/opc/chango/data-api/src/test/resources/data/csv-to-json-comma.csv \
;
```

- `separator` is the separator of csv data. If csv data is tab separated, the value is `TAB` . But for another separators, you need to type a separator value, for instance,  `,` , `|` , or something else.
- `is-single-quote` is if the escaped value of csv data is single quoted or not. Deffault is `false`.

For tab separated CSV, use the following.

```
chango upload csv local \
--data-api-server https://chango-data-api-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table csv_to_json_tab \
--separator TAB \
--is-single-quote false \
--file /home/opc/chango/data-api/src/test/resources/data/csv-to-json-tab.csv \
;
```
- parameter `separator` value must be `TAB`.

You can upload multiple csv files located in the directory.

```
chango upload csv local \
--data-api-server https://chango-data-api-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table csv_to_json_comma \
--separator "," \
--is-single-quote false \
--directory /home/opc/local-csvs \
;
```

### CSV on S3 to Chango

You can upload CSV data located on S3 to Chango.

```
chango upload csv s3 \
--data-api-server https://chango-data-api-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table csv_to_json_comma \
--separator "," \
--is-single-quote false \
--bucket <bucket> \
--access-key <access-key> \
--secret-key <secret-key> \
--endpoint <endpoint> \
--object-name csv-files/csv-to-json-comma.csv \
;
```

You can upload multiple CSV files located in the parent S3 object.

````
chango upload csv s3 \
--data-api-server https://chango-data-api-oci-user1.cloudchef-labs.com \
--schema iceberg_db \
--table csv_to_json_comma \
--separator "," \
--is-single-quote false \
--bucket <bucket> \
--access-key <access-key> \
--secret-key <secret-key> \
--endpoint <endpoint> \
--object-name csv-files/ \
;
````

Take a note that the value of `object-name` is `csv-files/` in which all the CSV files will be uploaded to chango.

### Query Iceberg Table using Trino Clients

You can run the following query if csv data is ingested to chango.

```
select * from iceberg.iceberg_db.csv_to_json_comma limit 1000;
```