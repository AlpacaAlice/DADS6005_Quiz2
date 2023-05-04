# DADS6005_Quiz2
KsqlDB Quiz

# Member
1. Nithida Phookriangkrai &nbsp; ID: 6420422004
2. Napasakon Monbut &nbsp;&emsp;&ensp; ID: 6420422009
3. Thunpitcha Sattabun &nbsp;&ensp;&nbsp; ID: 6420422010
4. Wisarut Sarai &nbsp;&emsp;&emsp;&emsp;&emsp;&nbsp; ID: 6420422012
5. Natchapat Youngchoay&nbsp; ID: 6420422013


# Objective

- Design & implement “data-streaming and real-time analytics” system
- Real-time data cleansing and analytics


# Dataset

- Food choices College students' food and cooking preferences ([Reference](https://www.kaggle.com/datasets/borapajo/food-choices?select=food_coded.csv))




# Procedure
- Please see the [Reference](https://docs.ksqldb.io/en/latest/tutorials/etl/) follow the link
1. Run docker
    ```batch
    docker-compose up -d
    ```
    
2. After docker build complete, run file <create_table.py> to create table inside Postgres
    ```python
    python create_table.py
    ```
    - You can check Postgres with command
        ```batch
        docker exec -it postgres /bin/bash
        ```
        ```batch
        psql -U postgres-user customers
        ```
    - Inside Ksql, run command to check
        ```sql
        SHOW TABLES;
        ```

3. Run file <import_foodcoded.py> to import data from <food_coded.csv>
    ```python
    python import_foodcoded.py
    ```

4. Start Ksql to create connector
    ```batch
    docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
    ```
    - Enter following queries to create a source connector
      ```sql
      SET 'auto.offset.reset' = 'earliest';
      ```
      ```sql
      CREATE SOURCE CONNECTOR customers_reader WITH (
          'connector.class' = 'io.debezium.connector.postgresql.PostgresConnector',
          'database.hostname' = 'postgres',
          'database.port' = '5432',
          'database.user' = 'postgres-user',
          'database.password' = 'postgres-pw',
          'database.dbname' = 'customers',
          'database.server.name' = 'localhost',
          'table.whitelist' = 'public.foodcoded',
          'transforms' = 'unwrap',
          'transforms.unwrap.type' = 'io.debezium.transforms.ExtractNewRecordState',
          'transforms.unwrap.drop.tombstones' = 'false',
          'transforms.unwrap.delete.handling.mode' = 'rewrite'
      );
      ```
    - Create a Stream topic1 for recieve data
      ```sql
      CREATE STREAM foodcoded WITH (
          kafka_topic = 'localhost.public.foodcoded',
          value_format = 'avro'
      );
      ```
    - Create a Stream topic2 for clean data follow <foodcoded_clean.sql>
      
      
    - Create a Stream topic3 for analyze data follow <foodcoded_analyze.sql>
    
    
    - Enter following queries to create a sink connector
      ```sql
      CREATE SINK CONNECTOR data_writer WITH (
          'connector.class' = 'io.confluent.connect.elasticsearch.ElasticsearchSinkConnector',
          'connection.url' = 'http://elastic:9200',
          'type.name' = 'kafka-connect',
          'topics' = 'foodcoded_analyze'
      );
      ```
      
5. Analyzed data from Elasticsearch can be check via command
    ```batch
    curl 'http://localhost:9200/foodcoded_analyze/_search?pretty'
    ```
